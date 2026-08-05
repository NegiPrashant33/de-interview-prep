# Query Execution (Logical Processing) Order

This is one of the highest-value things to have memorized cold for
interviews — it explains almost every "why does/doesn't this work" SQL
question.

```
FROM
  ↓
JOIN ... ON
  ↓
WHERE
  ↓
GROUP BY            (if present)
  ↓
Aggregate Functions (COUNT, SUM, AVG, etc. — only if used anywhere in the query)
  ↓
HAVING
  ↓
Window Functions    (ROW_NUMBER, RANK, DENSE_RANK, LAG/LEAD, etc.)
  ↓
SELECT
  ↓
DISTINCT
  ↓
ORDER BY
  ↓
LIMIT
```

## Key takeaways

- **`WHERE` filters rows before grouping.** It runs on raw, ungrouped rows,
  so you can't reference an aggregate (`WHERE COUNT(*) > 5` is invalid).
- **`HAVING` filters groups after aggregation.** This is why `HAVING` can
  reference aggregates like `COUNT(*)` but `WHERE` can't.
- **Aggregate functions are computed once GROUP BY has formed groups**, and
  they get computed for *every* aggregate expression used anywhere in the
  query (`SELECT`, `HAVING`, or `ORDER BY`) — not just the ones written in
  `SELECT`.
- **Window functions run after HAVING, before SELECT/DISTINCT.** This means
  window functions see the already-grouped, already-filtered result set —
  never the raw base table (more on this in the window functions doc).
- **`SELECT` runs late.** This is why you can `GROUP BY` a column you never
  select, `ORDER BY` a column you never select, or use an aggregate in
  `ORDER BY`/`HAVING` that isn't in `SELECT` at all.

## Worked example

```sql
SELECT department
FROM employees
GROUP BY department
ORDER BY COUNT(*) DESC;
```

Sequence of execution:

1. `FROM employees` — get raw rows
2. `GROUP BY department` — form groups
3. Compute aggregate functions used anywhere in the query — here, `COUNT(*)`
   is computed per group (even though it doesn't appear in `SELECT`)
4. `SELECT department` — project just the department column
5. `ORDER BY COUNT(*) DESC` — sort using the aggregate value that was
   already computed in step 3

So `COUNT(*)` is available to `ORDER BY` because it was computed during
aggregation, long before `SELECT` (and its column list) was ever evaluated.
