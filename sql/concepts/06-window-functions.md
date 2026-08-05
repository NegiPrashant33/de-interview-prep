# Window Functions — Full Reference

## The core mental model

Window functions do **not** reduce the number of rows the way `GROUP BY`
does. They compute a value *for each row*, based on a "window" of related
rows, and attach that value alongside the row's other columns.

```
GROUP BY:          many rows  →  fewer rows (one per group)
Window function:   many rows  →  same number of rows, each with an extra
                                  computed column
```

**If a query has both `GROUP BY` and a window function**, the window
function operates on the *already-grouped* result set, not the raw base
table. `PARTITION BY` inside the window function then partitions those
already-aggregated rows, not the original rows.

```
Original table
     ↓
  WHERE
     ↓
  GROUP BY
     ↓
New (grouped) table
     ↓
Window function (e.g. ROW_NUMBER) operates on this new table
```

This lines up exactly with the [execution order](02-query-execution-order.md):
`FROM → WHERE → GROUP BY → aggregates → HAVING → window functions → SELECT`.
So a window function's `ORDER BY` can safely use an aggregate like
`COUNT(*)`, because that aggregate was already computed by the time the
window function runs:

```sql
SELECT
  u.name AS result,
  ROW_NUMBER() OVER (ORDER BY COUNT(*) DESC, u.name ASC) AS rnk
FROM MovieRating mr
INNER JOIN Users u ON u.user_id = mr.user_id
GROUP BY mr.user_id;
```

---

## Anatomy of a window function call

```sql
<function>(...) OVER (
  PARTITION BY <col(s)>   -- optional: splits rows into groups
  ORDER BY <col(s)>       -- optional or required, depends on function
  <frame clause>          -- ROWS/RANGE BETWEEN ... — optional, has defaults
)
```

- **`PARTITION BY`** — always optional for every window function. If
  omitted, the entire result set is treated as a single partition.
- **`ORDER BY`** and the **frame clause** — whether they're optional,
  required, or meaningless depends on the *category* of window function.
  See below.

---

## 1. Aggregate window functions

`SUM`, `COUNT`, `AVG`, `MAX`, `MIN`

| Clause | Required? |
|---|---|
| `PARTITION BY` | Optional |
| `ORDER BY` | Optional — but changes behavior significantly (see below) |
| Frame (`ROWS`/`RANGE`) | Optional — has a default that depends on `ORDER BY` |

**Without `ORDER BY`:**
Default frame = entire partition (`RANGE BETWEEN UNBOUNDED PRECEDING AND
UNBOUNDED FOLLOWING`). Every row in the partition gets the **same**
value — e.g. every row shows the partition's total `SUM`.

**With `ORDER BY`:**
Default frame becomes `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.
The aggregate becomes a **running/cumulative** calculation — each row gets
the aggregate of all rows up to and including itself in the specified
order (a running total, running max, running average, etc.).

```sql
-- Same value repeated for every row in the partition
SUM(amount) OVER (PARTITION BY country) AS country_total

-- Running total, increasing row by row
SUM(amount) OVER (PARTITION BY country ORDER BY trans_date) AS running_total
```

This is the single most common gotcha with aggregate window functions:
just adding an `ORDER BY` silently switches you from "total" to "running
total" because of the default frame change.

---

## 2. Ranking window functions

`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `CUME_DIST`, `PERCENT_RANK`

| Clause | Required? |
|---|---|
| `PARTITION BY` | Optional |
| `ORDER BY` | Technically optional syntactically, but **practically required** — without it, ranking is arbitrary/non-deterministic (all rows may tie at rank 1, or row numbers are assigned in unpredictable order) |
| Frame (`ROWS`/`RANGE`) | **Not applicable** — ranking functions ignore/disallow a frame clause; they always operate on the whole partition in the specified order |

### ROW_NUMBER vs RANK vs DENSE_RANK

Given rows with values `[90, 90, 80]` ordered descending:

| Value | ROW_NUMBER | RANK | DENSE_RANK |
|---|---|---|---|
| 90 | 1 | 1 | 1 |
| 90 | 2 | 1 | 1 |
| 80 | 3 | 3 | 2 |

- **`ROW_NUMBER`** — always unique, sequential, no ties. Even identical
  values get different numbers (order among ties is arbitrary unless you
  add a tie-breaker column to `ORDER BY`).
- **`RANK`** — ties get the same rank, but the *next* rank skips numbers
  (gap after ties equal to the number of tied rows).
- **`DENSE_RANK`** — ties get the same rank, and the next rank does **not**
  skip — it's always previous rank + 1.

### NTILE(n)

Divides the partition into `n` roughly equal buckets, numbered `1..n`.
Requires `ORDER BY` to define which rows go into which bucket
meaningfully.

### PERCENT_RANK vs CUME_DIST

Both describe a row's relative standing within its partition, but they're
computed differently:

| Function | Formula | Uses |
|---|---|---|
| `PERCENT_RANK()` | `(RANK - 1) / (Total Rows - 1)` | The row's **rank** |
| `CUME_DIST()` | `Last position of current value / Total Rows` | The **last position** among rows sharing the current value |

A useful way to think about `CUME_DIST()`:

```
CUME_DIST() = (position of the last row with this value, in ORDER BY order) / total rows
```

Both effectively require `ORDER BY` — without it, all rows tie as one
group, so `CUME_DIST()` = 1 for every row and `PERCENT_RANK()` = 0 for
every row.

---

## 3. Value window functions

`FIRST_VALUE`, `LAST_VALUE`, `LAG`, `LEAD`

| Function | `PARTITION BY` | `ORDER BY` | Frame |
|---|---|---|---|
| `LAG` / `LEAD` | Optional | **Effectively required** — "previous/next row" only means something relative to an order | Not applicable — frame clause doesn't affect `LAG`/`LEAD` |
| `FIRST_VALUE` / `LAST_VALUE` | Optional | Optional, but strongly affects the default frame | Applicable, and **matters a lot** |

### The LAST_VALUE gotcha

`FIRST_VALUE`/`LAST_VALUE` are affected by the same default-frame behavior
as aggregate window functions: with `ORDER BY` present and no explicit
frame, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT
ROW`. Since the frame ends at the *current row*, `LAST_VALUE` ends up just
returning the current row's own value — not the true last value of the
partition. To get the actual last value of the partition, you must
explicitly widen the frame:

```sql
LAST_VALUE(amount) OVER (
  PARTITION BY country
  ORDER BY trans_date
  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) AS true_last_value
```

### LAG / LEAD syntax

```sql
LAG(column, offset, default_value)  OVER (PARTITION BY ... ORDER BY ...)
LEAD(column, offset, default_value) OVER (PARTITION BY ... ORDER BY ...)
```

- `offset` — how many rows back/forward (default 1)
- `default_value` — what to return when there's no such row (default
  `NULL`), e.g. the first row in a partition has no `LAG` value

---

## Quick reference summary

| Category | Functions | `PARTITION BY` | `ORDER BY` | Frame |
|---|---|---|---|---|
| Aggregate | `SUM`, `COUNT`, `AVG`, `MAX`, `MIN` | Optional | Optional (changes default frame) | Optional, has default |
| Ranking | `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE` | Optional | Practically required | Not applicable |
| Distribution | `CUME_DIST`, `PERCENT_RANK` | Optional | Practically required | Not applicable |
| Offset | `LAG`, `LEAD` | Optional | Practically required | Not applicable |
| First/Last | `FIRST_VALUE`, `LAST_VALUE` | Optional | Optional (changes default frame — watch `LAST_VALUE`!) | Applicable, often needs to be set explicitly |
