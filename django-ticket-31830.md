# Django ticket #31830

Django's ORM has a feature where it detects expressions that are a priori false and optimizes accordingly.

For example, there is no need to send `SELECT ... FROM ... WHERE x IN (<empty list>)` to the database to know that no rows of results will be returned. It's like `WHERE 1=0`.

Django knows this and will actually give you the result without talking to the database.

And when you do something like `SELECT ... FROM ... WHERE (x IN (<empty list>)) OR (<something else>)`,
Django sends `SELECT ... FROM ... WHERE <something else>` to the database, for the same reason.

When you write a query like `SELECT COUNT(*) FROM ... WHERE EXISTS(<something that is apriori false>) OR <something else>`,
Django should send to the database something like `SELECT COUNT(*) FROM ... WHERE <something else>`.

There was a bug so that in the case of the previous query, it "forgot" about the `OR <something else>` and just acted as if you did
`SELECT COUNT(*) FROM ... WHERE 1=0` and returned 0, without querying the database at all.

I had code that, before performing an operation on a group of entities, did something like this:

```python
qs_a = EntityPermission.objects \
    .filter(entity_id=OuterRef('pk'),
            permission_id__in=request.user.permissions_list)
count_allowed = Entity.objects \
    .annotate(allowed_by_a=Exists(qs_a), allowed_by_b=...) \
    .filter(Q(allowed_by_a=True) | Q(allowed_by_a=True),
            id__in=request.input_list) \
    .count()
if count_allowed != len(request.input_list):
    ...
```

If the count returned from the database is the same as the number of entities, the operation proceeds, otherwise there is an "insufficient permissions" error.

The bug in Django caused a bug in the application in case `request.user.permissions_list` is empty.
The bug was hiding too much, not showing too much, but still bad.
The bug was caught in testing before the sprint was deployed,
and after some head scratching the issue in the application was fixed by special casing `<empty list>`
and generating different query in that case, almost manually performing the optimization Django usually performs.

The project at work uses Django 2.2 LTS, specifically _because_ it is LTS, so that we don't have to switch versions often and are never in the situation where we have to either:
(a) let the customer run an unsupported version of Django or
(b) perform the version upgrade for free or
(c) tell the customer that they must pay us money for doing security work which brings no visible benefits or else they'll get hacked.

When I went to report the bug to Django upstream, I did my homework:

1. I've searched the bug tracker for mention of this issue (none).

2. I tested and found out that it was already fixed in django 3.0, which is not LTS.

3. I explained the bug in [my bug report](https://code.djangoproject.com/ticket/31830).

4. I wrote a unit test, mergeable into the Django source code, for the bug, which you can see here:

A django testcase you can add to `tests/expressions/tests.py`:

```python
class Ticket31830Tests(TestCase):
    def test_ticket_31830(self):
        employee1 = Employee.objects.create(firstname='a', lastname='b')
        company1 = Company.objects.create(
            name='1', num_employees=1, num_chairs=1, ceo=employee1)
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

This is a repro you can run in a shell of a new Django project with all default settings:

```python
from django.contrib.auth.models import User, Group
from django.db.models import Exists, OuterRef, Q
user1, created = User.objects.get_or_create(username='user1')
group1, created = Group.objects.get_or_create(name='group1')
User_groups = User._meta.fields_map['User_groups+'].related_model
user_group, created = User_groups.objects.get_or_create(user=user1, group=group1)
user_groups1_qs = User_groups.objects \
    .filter(group__id__in=[group1.id], user_id=OuterRef('pk')) \
    .values('id')
user_groups2_qs = User_groups.objects \
    .filter(group__id__in=[], user_id=OuterRef('pk')) \
    .values('id')
qs = User.objects \
    .annotate(has_groups1=Exists(user_groups1_qs), has_groups2=Exists(user_groups2_qs)) \
    .filter(Q(has_groups1=True) | Q(has_groups2=True))
count_before = qs.count()
items = list(qs)
count_after = qs.count()
print(count_before)
print(len(items))
print(count_after)
```

The response from Django maintainer @felixxm was very fast and to the point.
They accurately recognized the issue and correctly informed me in which commit and ticket the issue was fixed.
And also that since it's not a security bug, the fix will not be backported to Django 2.2 LTS.

The bug was fixed in [a ticket](https://code.djangoproject.com/ticket/30158) called "Cleanup/optimization Subquery expressions unnecessarily added to group by",
in [a commit](https://code.djangoproject.com/changeset/fb3f034f1c63160c0ff13c609acd01c18be12f80/) called "Avoided unnecessary subquery group by on aggregation."
with a description of "Subquery annotations can be omitted from the GROUP BY clause on aggregation as long as they are not explicitly grouped against."

At first I thought maybe the maintainer was wrong, but I checked and indeed the bug is fixed by exactly that commit.

So now I need to add this issue to the list of things I watch out for in code review.

Maybe I'm super wrong, but I think had this issue caused a security incident in Instagram, Django project would have backported the fix to the LTS version.

But I guess "QuerySet.count() is no longer a lie" is a "Cleanup/optimization", and not worthy of backporting.
