# Aggregate Functions Basics

## COUNT variants

| Expression | What it counts |
|---|---|
| `COUNT(*)` | All rows in the result set, **including** rows where some columns are `NULL` |
| `COUNT(column_name)` | Only rows where `column_name` is **NOT NULL** |
| `COUNT(DISTINCT column_name)` | Unique **non-NULL** values in that column |

**Interview point:** `COUNT(*)` never ignores `NULL`s because it isn't
looking at any particular column — it's counting rows. `COUNT(column)` and
`COUNT(DISTINCT column)` both ignore `NULL`s because they evaluate the
column's value per row.

## Using boolean expressions as 0/1 (MySQL trick)

In MySQL, a boolean expression evaluates to `1` (true) or `0` (false), so it
can be summed directly to get a conditional count:

```sql
SELECT
  DATE_FORMAT(trans_date, '%Y-%m') AS month,
  country,
  COUNT(id)                         AS trans_count,
  SUM(state = 'approved')           AS approved_count,
  SUM(amount)                       AS trans_total_amount,
  SUM((state = 'approved') * amount) AS approved_total_amount
FROM Transactions
GROUP BY month, country;
```

`SUM(state = 'approved')` is exactly equivalent to:

```sql
SUM(CASE WHEN state = 'approved' THEN 1 ELSE 0 END) AS approved_count
```

The `CASE WHEN` form is more verbose but is **standard ANSI SQL** and works
in databases (like SQL Server or older Postgres versions) where boolean
expressions don't implicitly cast to integers the way MySQL allows. Good to
know both — use the short form in MySQL, but recognize/write the `CASE`
form if asked in a database-agnostic interview setting.

Similarly, `SUM((state = 'approved') * amount)` multiplies the 0/1 flag by
`amount`, effectively summing `amount` only for approved rows.
