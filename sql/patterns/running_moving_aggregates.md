# Running & Moving Aggregates — SQL Interview Pattern

## Overview

This pattern covers **two closely related window-function techniques** used to analyze how metrics behave over time:

1. **Running / Cumulative Aggregates** — "the snowball" (grows forever, e.g., running total of revenue).
2. **Moving Averages / Sliding Windows** — "the magnifying glass" (fixed size, slides forward, e.g., 7-day rolling average).

Both show up constantly in FAANG interviews (Amazon, Netflix, Stripe, Google, Meta) because almost every business metric is either **cumulative** (totals over time) or **noisy** (needs smoothing). Both rely on the same core tool: **window functions with a frame specification** — the difference is just how that frame is bounded.

---

## Core Concept 1: Running Totals (The Snowball)

- A **regular `GROUP BY`** computes one result per group and discards individual rows.
- A **window function** keeps every row and computes a value across a "window" of related rows that slides as you move through the data.
- For a **running total**: the window starts at the first row and **expands by one row at a time** — like a snowball rolling downhill, only ever growing, never shrinking.
- This is expressed using **`UNBOUNDED PRECEDING`** — no limit on how far back the window reaches; it always goes back to the very first row.
- Works with any aggregate: running sum, running count, running average, running min/max — but `SUM` is most common in interviews.

### Basic Syntax

```sql
SELECT
  order_date,
  amount,
  SUM(amount) OVER (
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total
FROM sales
```

**Clause breakdown:**

- `SUM(amount)` — aggregate function; doesn't collapse rows because it's paired with `OVER()`.
- `OVER(...)` — defines the window: which rows participate, what order, how wide.
- `ORDER BY order_date` — **critical**. Defines what "preceding" means. Without it, `PRECEDING`/`FOLLOWING` are meaningless and results become unpredictable.
- `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` — window starts at the very first row and ends at the current row.

### With PARTITION BY

```sql
SUM(amount) OVER (
  PARTITION BY region
  ORDER BY order_date
  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

- Splits data into separate groups (e.g., per region). Each partition gets its own independent running total ("separate snowballs").
- Without `PARTITION BY`, the running total accumulates across the **entire table**.

### ⚠️ Default Frame Warning

- If you write `SUM(amount) OVER (ORDER BY order_date)` **without** an explicit frame, most databases default to `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` — note: **RANGE**, not **ROWS** (see pitfalls below).
- **Best practice:** Always write the frame specification out explicitly. Never rely on defaults.

---

## Core Concept 2: Moving Averages (The Magnifying Glass / Sliding Window)

- Unlike a running total, a moving average uses a window of **fixed size** that **doesn't grow** — it **slides**.
- Example: a 7-day window on Monday covers Monday + the 6 days before it. Move to Tuesday, and the whole window shifts forward by one — Monday from a week ago falls off the back.
- **Why:** Daily data is noisy (e.g., spikes on weekends, dips on weekdays). Averaging across a consistent window cancels out spikes/dips and reveals the real underlying trend.

### Key Mental Shift from Running Totals

- Running total: `UNBOUNDED PRECEDING` (reaches back to the start).
- Moving average: bounded — `N PRECEDING` (a fixed number of rows behind the current row).

### Trailing vs. Centered Moving Average

- **Trailing moving average** — looks only backward (most common in business dashboards, since you only know the past).
- **Centered moving average** — looks both backward and forward (e.g., 3 preceding + 3 following); used in forecasting/signal processing where future data is available.

### Basic Syntax (7-day trailing average)

```sql
SELECT
  order_date,
  amount,
  AVG(amount) OVER (
    ORDER BY order_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS seven_day_moving_average
FROM sales
```

**Clause breakdown:**

- `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` → 6 rows before + the current row = **7 rows total**.
- **Rule:** For an **N-day** moving average, use **`N - 1 PRECEDING`** (the current row always counts as 1).

### With PARTITION BY

```sql
AVG(amount) OVER (
  PARTITION BY region
  ORDER BY order_date
  ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
)
```

- Each region/show/merchant gets its own independent sliding window — windows never mix across partitions.

### Centered Moving Average Syntax

```sql
ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING
```

- 3 rows behind + current + 3 rows ahead = 7-wide centered window.

---

## ROWS vs. RANGE — The Critical Distinction

This distinction matters for **both** running totals and moving averages.

| | `ROWS` | `RANGE` |
|---|---|---|
| **Basis** | Physical row position | Value/date awareness |
| **Behavior** | Steps back by row count, ignoring actual date gaps or duplicate values | Steps back by value; groups rows with the same value as one point |
| **Duplicates** | Each row treated individually | Rows with identical order-by values are treated as the same point — they see each other's values |
| **Gaps in data** | Ignores gaps — "6 preceding" always means 6 rows back, regardless of missing dates | Respects gaps — "last 6 days" includes only rows whose date actually falls in that range |

### Example (duplicate dates, running total)

Rows: Jan 1 ($100), Jan 1 ($200), Jan 2 ($300)

- **ROWS**: 100 → 300 (100+200) → 600 (100+200+300) — each row processed individually.
- **RANGE**: Both Jan 1 rows get 300 (lumped together as same date) → Jan 2 gets 600.

### Example (gaps in data, moving average)

- `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` → steps back 6 **rows**, which might span far more than 6 calendar days if the merchant doesn't transact daily.
- `RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW` → steps back 6 **calendar days**, correctly handling missing days (a missing day just contributes nothing).

**Rule of thumb:**

- Interviewer says "last N **rows**" → use `ROWS`.
- Interviewer says "last N **days**" (calendar days) → use `RANGE`.
- **Safe default for running totals:** use `ROWS` unless you specifically want duplicate values grouped together.
- **Database support note:** `RANGE ... INTERVAL` syntax is supported in Postgres/BigQuery/Snowflake; other engines (Oracle, SQL Server) may need a self-join on date ranges, a lateral join, or a generated calendar table as a fallback.

---

## Common Pitfalls (Both Patterns)

1. **Missing `ORDER BY` inside `OVER()`**
   - Most dangerous — query still runs without erroring, but results become nondeterministic or the frame is ignored entirely.
   - Always include `ORDER BY` for any cumulative or moving calculation.

2. **Confusing `ROWS` with `RANGE`**
   - See table above. Duplicate values or data gaps make these produce different (and sometimes very wrong) results.
   - Know which one the question intends — ask if unclear.

3. **Off-by-one errors in frame size (moving averages specifically)**
   - Writing `7 PRECEDING` for a "7-day" average actually creates an **8-row** window (current row counts as 1).
   - Correct: `N - 1 PRECEDING` for an N-day window.

4. **Not handling NULLs**
   - Aggregate functions silently skip NULLs, which can cause confusing jumps in a running total.
   - Fix: wrap with `COALESCE(amount, 0)` before summing to make behavior explicit.

5. **Partial windows at the edges (moving averages)**
   - The first few rows (before the window fills up) will use fewer than N rows — no error is thrown, they just average whatever exists.
   - Row 1's average = row 1 itself; row 2's average = avg(row 1, row 2); etc.
   - Fix: filter using a row-number check (`row_number >= N`) or a `COUNT(*)` over the same frame to require full windows, if the use case demands it.

6. **Reaching for a self-join or correlated subquery instead of a window function**
   - Works, but is extremely slow (effectively a full table scan per row).
   - Window functions solve this in a single pass — always prefer them.

**Common theme:** Always write frame specifications **explicitly** (never rely on defaults) — it signals clear intent and avoids subtle bugs.

---

## Interview Question 1 (Amazon): Running Total of Revenue Per Region

**Prompt:** Table `daily_revenue(revenue_date, region, amount)`. For each region, calculate daily revenue and the running total of revenue over time, ordered by date.

**Approach:**

- Need `PARTITION BY region` (running total resets per region).
- Need `ORDER BY revenue_date ASC` (chronological order).
- Frame: `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.

```sql
SELECT
  revenue_date,
  region,
  amount AS daily_revenue,
  SUM(amount) OVER (
    PARTITION BY region
    ORDER BY revenue_date ASC
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total
FROM daily_revenue
ORDER BY region, revenue_date
```

**Edge case:** If a day is missing from the table for a region, the running total simply skips it (no zero-row is created). Showing every date including gaps would require generating a calendar table and left-joining — a separate pattern.

---

## Interview Question 2 (Stripe): Cumulative Signups + Daily Percentage of Total

**Prompt:** Table `user_signups(user_id, signup_date, country)`. For each country: (1) daily signup count, (2) cumulative total of signups, (3) each day's signups as a percentage of the cumulative total so far.

**Approach — needs both aggregation AND a window function, so use a CTE:**

1. **CTE `daily_counts`** — aggregate raw signups into daily counts per country (`GROUP BY country, signup_date`).
2. **Outer query** — apply `SUM() OVER()` (partitioned by country) on top of the daily counts for the running total, then compute `daily / cumulative * 100` for the percentage.

```sql
WITH daily_counts AS (
  SELECT
    country,
    signup_date,
    COUNT(user_id) AS daily_signups
  FROM user_signups
  GROUP BY country, signup_date
)
SELECT
  country,
  signup_date,
  daily_signups,
  SUM(daily_signups) OVER (
    PARTITION BY country
    ORDER BY signup_date ASC
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS cumulative_signups,
  ROUND(
    daily_signups::decimal /
    SUM(daily_signups) OVER (
      PARTITION BY country
      ORDER BY signup_date ASC
      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) * 100,
    2
  ) AS percentage_of_total
FROM daily_counts
ORDER BY country, signup_date
```

**Why a CTE?** `COUNT` needs a `GROUP BY` which collapses rows; the running `SUM` needs individual (already-collapsed) rows to work with. Separating these two steps keeps the query clean and readable — readability matters in interviews.

**Note:** Percentage generally **decreases over time** as the cumulative total grows — this is expected behavior for cumulative metrics.

---

## Interview Question 3 (Netflix): 7-Day Trailing Moving Average Per Title

**Prompt:** Table `daily_watch_hours(watch_date, title_id, hours_watched)` — one row per title per day, no gaps. For each title, compute the 7-day trailing moving average of hours watched.

**Approach:**

- `PARTITION BY title_id` — each title needs its own sliding window.
- `ORDER BY watch_date ASC`.
- Frame: `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` (6 preceding + current = 7 rows).

```sql
SELECT
  watch_date,
  title_id,
  hours_watched,
  AVG(hours_watched) OVER (
    PARTITION BY title_id
    ORDER BY watch_date ASC
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS seven_day_moving_average
FROM daily_watch_hours
ORDER BY title_id, watch_date
```

**Why `ROWS` here (not `RANGE`)?** Because the data is assumed to have one row per title per day with no gaps, `ROWS` and `RANGE` would give identical results — `ROWS` is simpler. If gaps were possible, switch to `RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW`.

**Edge case — partial windows:** The first 6 rows per title have fewer than 7 days of data, so their averages are computed from a smaller window. If only fully-populated windows should count, filter using `ROW_NUMBER() >= 7` per partition, or use a `COUNT()` over the same frame and require it to equal 7.

---

## Interview Question 4 (Stripe): 30 Calendar-Day Rolling Average With Gaps

**Prompt:** Table `merchant_transactions(transaction_date, merchant_id, amount)` — NOT one row per day; some merchants don't transact daily (at most one row per merchant per day when active). For each merchant, compute a 30-**calendar**-day rolling average (missing days should still count as part of the window, not shift it).

**This is the classic `ROWS` vs. `RANGE` trap:**

- Using `ROWS BETWEEN 29 PRECEDING AND CURRENT ROW` steps back 29 physical **rows** — for a merchant who transacts twice a week, that could span 100+ calendar days. Wrong.
- Correct: `RANGE BETWEEN INTERVAL 29 DAY PRECEDING AND CURRENT ROW` — steps back by actual **calendar days**, correctly using whatever rows fall within that true 30-day window.

```sql
SELECT
  transaction_date,
  merchant_id,
  amount,
  AVG(amount) OVER (
    PARTITION BY merchant_id
    ORDER BY transaction_date
    RANGE BETWEEN INTERVAL '29' DAY PRECEDING AND CURRENT ROW
  ) AS rolling_30_day_average
FROM merchant_transactions
ORDER BY merchant_id, transaction_date
```

**Trace example:** Merchant transacts Jan 1, Jan 3, Jan 5, then a gap, then Jan 20.

- Jan 20's window (last 29 days) correctly includes Jan 1, 3, 5, and 20 (4 rows) — because all fall within the true calendar range.
- Using `ROWS` instead would incorrectly span over a month of calendar time while only capturing 4 physical rows.

**Fallback for unsupported engines:** `RANGE ... INTERVAL` works in Postgres/BigQuery/Snowflake. For Oracle/SQL Server, fall back to a self-join on date ranges, a lateral join, or a generated calendar table.

**Optional refinement:** To only report the rolling average when a merchant has at least N transactions in the window, add a `COUNT(*)` over the same range frame and filter (e.g., `>= 10`).

---

## Key Takeaways

- **Running/Cumulative aggregate** → use `SUM/COUNT/AVG(...) OVER (ORDER BY ... ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` — the "snowball" that only grows.
- **Moving average/Sliding window** → use `AVG(...) OVER (ORDER BY ... ROWS BETWEEN N-1 PRECEDING AND CURRENT ROW)` — the "magnifying glass" that stays a fixed size and slides.
- Both patterns depend on the same three ingredients: an aggregate function, `OVER()` with a mandatory `ORDER BY`, and an explicit frame specification.
- **The single most important decision:** does the question mean **row positions** or **actual calendar values**? That determines `ROWS` vs. `RANGE` — and getting this right is often the difference between a passing answer and an impressive one.
- Always write frame specifications explicitly, always partition when the question says "for each X," and always prefer window functions over self-joins/correlated subqueries for performance.
