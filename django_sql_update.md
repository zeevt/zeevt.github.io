Most bugs I've made and debugged were trivial and not interesting.

A few were more interesting concurrency bugs that are harder to model and test.

Especially nasty bug are where there is in fact something like a
state machine that is critical for the correctness of the system,
but it is not explicit in the code. It's not pointed out by variable names or comments.
Implicit, "hidden" state machines are bad.

The system is usually not very complicated.

I've never worked at a place where something like TLA+ was ever concidered.

It's possible that if I learn enough TLA+ to actually use it to solve a bug
or make some code a lot more trustworthy, other people might be convinced to give it a try.
But I haven't spent any real time learning it. When I encounter a problem like that I just use pencil and paper.

Anyway, one bug I had was in semantics of SQL subqueries in PostgreSQL 9.6.

I did not find out whether it's documented behavior of PostgreSQL or a bug in PostgreSQL,
though I am curious.

The issue is that it seems that a condition in a subquery is racy against other queries,
while a condition in the WHERE of the UPDATE itself is (as expected) not racy.

The important thing was to just make Django ORM put the condition in the WHERE of the UPDATE,
not in a subquery.

When we saw it in testing, I fixed it quickly 
(it was obvious to me what the only way for it to have happened was),
so I didn't dive into the docs.

The python code was something like this:

```
tenant_num_processes_qs = Item.objects \
  .filter(owner__profile__tenant_id=OuterRef('owner__profile__tenant_id'),
          state__in=(Item.WAITING, Item.IN_PROGRESS),
          stage__in=Item.SOME_STAGES) \
  .values('owner__profile__tenant_id') \
  .annotate(tenant_num_processes=Count('id')) \
  .values('tenant_num_processes')
qs = Item.objects.filter(state=Item.WAITING, stage=Item.PROCESS1)
qs = qs.annotate(tenant_used=Coalesce(Subquery(tenant_num_processes_qs, output_field=IntegerField()), Value(0)))
tenant_allowed = 10
update_qs = qs.filter(id=item_id, tenant_used__lt=tenant_allowed)
rows_updated = update_qs.update(state=Item.IN_PROGRESS)
```

and the SQL was something like this:

```
UPDATE item
SET state = 'in_progress'
WHERE item.id IN (
  SELECT a0.id
  FROM item a0
  JOIN user a1
  JOIN profile a2
  WHERE 
    a0.id = X
    AND a0.stage = 'process1'
    AND a0.state = 'waiting'
    AND COALESCE(
      (
         SELECT COUNT(b0.id)
         FROM item b0
         JOIN user b1
         JOIN profile b2
         WHERE
           b0.stage IN ('process1', 'process2', 'process3')
           AND b0.state IN ('waiting', 'in_progress')
           AND b2.tenant_id = a2.tenant_id
         GROUP BY b2.tenant_id
      )
    , 0) < 10
)
```

And the fix was to do this instead:

```
tenant_num_processes_qs = Item.objects \
  .filter(owner__profile__tenant__profile__user__item=OuterRef('id'),
```

so that the SQL would be like so:

```
UPDATE item
SET state = 'in_progress'
WHERE
  item.id = X
  AND item.stage = 'process1'
  AND item.state = 'waiting'
  AND COALESCE(
    (
       SELECT COUNT(b0.id)
       FROM item b0
       JOIN user b1
       JOIN profile b2
       JOIN tenant b3
       JOIN profile b4
       JOIN user b5
       FROM item b6
       WHERE
         b0.stage IN ('process1', 'process2', 'process3')
         AND b0.state IN ('waiting', 'in_progress')
         AND b6.id = item.id
       GROUP BY b2.tenant_id
    )
  , 0) < 10
```
