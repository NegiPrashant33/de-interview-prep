# WHERE vs HAVING

| | Runs when | Can reference aggregates? |
|---|---|---|
| `WHERE` | Before grouping — filters raw rows | No |
| `HAVING` | After grouping/aggregation — filters groups | Yes |

```sql
SELECT department, COUNT(*) AS cnt
FROM employees
WHERE active = 1              -- filters individual employee rows first
GROUP BY department
HAVING COUNT(*) > 5;          -- filters departments after counting
```

**Rule of thumb:** if the condition is about an individual row's raw
column value, use `WHERE`. If the condition is about a computed/aggregated
value for a group, use `HAVING`.
