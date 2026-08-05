# Common Pitfalls & Gotchas

## Chained comparisons don't work like in Python

```sql
-- WRONG — does not mean "l1.num = l2.num AND l2.num = l3.num"
WHERE l1.num = l2.num = l3.num
```

SQL evaluates this left to right: `l1.num = l2.num` first produces a
boolean (`1`/`0`/`TRUE`/`FALSE`), and *that* result is then compared
against `l3.num` — almost never what you intended.

**Correct version:**

```sql
WHERE l1.num = l2.num AND l2.num = l3.num
```

## DISTINCT is row-wise, not column-wise

Covered in [03](03-select-distinct-groupby.md) — `SELECT DISTINCT a, b`
dedupes on the combination of `(a, b)`, not on `a` alone.

## Aggregates in ORDER BY that aren't in SELECT

This works because `ORDER BY` runs after aggregation, using values already
computed — see [02](02-query-execution-order.md). Don't assume you need to
`SELECT` a column to `ORDER BY` it.

## Window functions "see" the grouped table, not the raw table

If a query combines `GROUP BY` with a window function, remember the window
function's `PARTITION BY`/`ORDER BY` operate on the post-aggregation
result set. See [06](06-window-functions.md) for the full breakdown.

## LAST_VALUE returning the "wrong" value

The most common `LAST_VALUE` bug: adding `ORDER BY` without an explicit
frame silently limits the frame to "up to current row," so `LAST_VALUE`
just echoes the current row. Always pair `LAST_VALUE` with an explicit
`ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` if you want the
true last value of the partition.
