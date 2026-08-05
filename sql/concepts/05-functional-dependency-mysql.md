# Functional Dependency & the MySQL GROUP BY Exception

Standard SQL (with `ONLY_FULL_GROUP_BY` mode, the strict ANSI behavior)
requires every column in `SELECT` to either be in the `GROUP BY` clause or
wrapped in an aggregate function. MySQL relaxes this **when the selected
column is functionally dependent on the grouped column** — most commonly
when you group by a primary key.

```sql
SELECT u.name
FROM MovieRating mr
JOIN Users u
  ON mr.user_id = u.user_id
GROUP BY mr.user_id;
```

This works even though `u.name` isn't in the `GROUP BY` list and isn't
aggregated.

## Interview-ready explanation

> "`Users.user_id` is the primary key of the `Users` table, so each
> `user_id` maps to exactly one `name`. Since the query groups by
> `user_id`, `name` is functionally dependent on the grouped column —
> there's no ambiguity about which `name` to return for each group — so
> MySQL allows it in `SELECT` without requiring it in `GROUP BY`."

**Caveat:** this only works because `user_id` is unique per row it maps to
(primary/unique key). If you grouped by a non-unique column and tried to
select an unrelated non-aggregated column, MySQL would either error out (in
strict mode) or return a non-deterministic value per group (in relaxed
mode) — both bad. Don't rely on this pattern unless the functional
dependency is genuinely guaranteed by a key constraint.
