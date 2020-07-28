# Django ticket #31830

Django is a popular web application framework in Python, similar to Ruby on Rails (same vintage).

I've been working in a Django shop as a backend developer for a few years now, so I've spent a lot of time with it.

Django's ORM has a feature where it detects expressions that are a priori false and optimizes accordingly.

For example, there is no need to send `SELECT ... FROM ... WHERE x IN (<empty list>)` to the database to know that no rows of results will be returned. It's like `WHERE 1=0`.

Django knows this and will actually give you the result without talking to the database.

And when you do something like `SELECT ... FROM ... WHERE (x IN (<empty list>)) OR (<something else>)`,
Django sends `SELECT ... FROM ... WHERE <something else>` to the database, for the same reason.

When you write a query like `SELECT COUNT(*) FROM ... WHERE EXISTS(<something that is apriori false>) OR <something else>`,
Django should send to the database something like `SELECT COUNT(*) FROM ... WHERE <something else>`.

There was a bug so that in the case of the previous query, it "forgot" about the `OR <something else>` and just acted as if you did 
`SELECT COUNT(*) FROM ... WHERE 1=0` and retuned 0, without querying the database at all.

I had code that, before performing an operation on a group of entities, constructed a single query with EXISTS using the acting user's permissions for all the entities,
and then filtered based on the EXISTS and counted how many entities there are.

If the count returned from the database is the same as the number of entities, the opertion proceeds, otherwise there is an "insufficient permissions" error.

The bug in Django caused a security bug in the application. It was caught in testing before the sprint was deployed,
and after some head scratching the issue in the application was fixed by special casing `<empty list>`
and generating different query in that case, almost manually performing the optimzation Django usually performs.

The project at work uses Django 2.2 LTS, specifically _because_ it is LTS, so that we don't have to switch versions often and are never in the situation where we have to either:
(a) let the customer run an unsupported version of Django or 
(b) perform the version upgrade for free or 
(c) tell the customer that they must pay us money for doing security work which brings no visible benefits or else they'll get hacked.

When I went to report the bug to Django upstream, I did my homework:

1. I've searched the bug tracker for mention of this issue (none).

2. I tested and found out that it was already fixed in django 3.0, which is not LTS.

3. I wrote a unit test, mergeable into the Django source code, for the bug, which you can see here:

A django testcase you can add to `tests/expressions/tests.py`:

```
class Ticket31830Tests(TestCase):
    def test_ticket_31830(self):
        employee1 = Employee.objects.create(firstname='a', lastname='b')
        company1 = Company.objects.create(name='1', num_employees=1, num_chairs=1, ceo=employee1)
        qs1 = Company.objects \
            .filter(ceo_id=OuterRef('pk'), id__in=[company1.id]) \
            .values('id')
        qs2 = Company.objects \
            .filter(ceo_id=OuterRef('pk'), id__in=[]) \
            .values('id')
        qs = Employee.objects \
            .annotate(qs1=Exists(qs1), qs2=Exists(qs2)) \
            .filter(Q(qs1=True) | Q(qs2=True))
        count_before = qs.count()
        count_items = len(qs)
        count_after = qs.count()
        self.assertEqual(count_before, count_items)
        self.assertEqual(count_before, count_after)
```

4. I explained the bug in [my bug report](https://code.djangoproject.com/ticket/31830).

The response from Django maintainer @felixxm was very fast and to the point.
They accurately recognized the issue and corectly informed me in which commit and ticket the issue was fixed.
And also that since it's not a security bug, the fix will not be backported to Django 2.2 LTS.

The bug was fixed in [a ticket](https://code.djangoproject.com/ticket/30158) called "Cleanup/optimization Subquery expressions unnecessarily added to group by",
in [a commit](https://code.djangoproject.com/changeset/fb3f034f1c63160c0ff13c609acd01c18be12f80/) called "Avoided unnecessary subquery group by on aggregation."
with a description of "Subquery annotations can be omitted from the GROUP BY clause on aggregation as long as they are not explicitly grouped against."

At first I thought maybe the maintainer is wrong, but I checked and indeed the bug is fixed by exactly that commit.

So now I need to add this issue to the list of things I watch out for in code review.

Maybe I'm super wrong, but I think had this issue caused a security incident in Instagram, Django project would have backported the fix to the LTS version.

But I guess "QuerySet.count() is no longer a lie" is a "Cleanup/optimization", and not worthy of backporting.
