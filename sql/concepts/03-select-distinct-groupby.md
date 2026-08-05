# SELECT, DISTINCT & GROUP BY Notes

## DISTINCT applies to the whole row, not just one column

```sql
SELECT DISTINCT col1, col2 FROM table;
```

This does **not** mean "distinct `col1` values" — it means distinct
*combinations* of `(col1, col2)`. Two rows are only deduplicated if every
selected column matches.

## Sorting is lexicographic by default

```sql
ORDER BY name ASC
```

MySQL sorts strings lexicographically (dictionary order) by default, based
on the column's collation — not by string length or any custom logic.

## Aggregate functions can appear in SELECT, ORDER BY, and HAVING

Anywhere an aggregate is written, it's evaluated during the aggregation
step (see [Query Execution Order](02-query-execution-order.md)), so it's
valid in all three clauses:

```sql
SELECT department, COUNT(*) AS cnt
FROM employees
GROUP BY department
HAVING COUNT(*) > 5
ORDER BY COUNT(*) DESC, department ASC;
```

## Multi-key ORDER BY (tie-breaking)

```sql
ORDER BY COUNT(*) DESC, u.name ASC
ORDER BY AVG(mr.rating) DESC, m.title ASC
```

The second column only matters when the first column has ties — it's a
tie-breaker, useful (and often required by interview problems) to make
result ordering deterministic.
