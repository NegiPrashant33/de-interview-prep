# Gaps and Islands — SQL Interview Pattern

## Overview

- **Gaps and Islands** is a common SQL interview pattern used by companies like Meta, Google, and Microsoft.
- Classic question: *"For each user, find the longest streak of consecutive days they logged in."*
- A row-by-row looping approach (like in Python/Java) is a trap in SQL — it's slow, ugly, and signals you don't "think in sets."
- The data hides a structure: consecutive days form **islands** (continuous runs) separated by **gaps** (missing days/values).

---

## Core Intuition: The Row-Number-Difference Trick

**Goal:** Teach the database to detect groups (islands) automatically — no loops, no manual row comparisons.

### The Insight

1. Assign each row a sequential number using `ROW_NUMBER()` ordered by the sequence column (e.g., login date).
2. Subtract the row number (as days) from the date value.
3. **Within a consecutive run**, this subtraction produces the **same constant value** for every row.
4. **When the sequence breaks** (a gap), the subtraction produces a **new/different constant**.

### Example

Dates: Jan 1, 2, 3, (gap), 6, 7, (gap), 10

| Date | Row # | Date − Row# (days) |
|------|-------|---------------------|
| Jan 1 | 1 | Dec 31 |
| Jan 2 | 2 | Dec 31 |
| Jan 3 | 3 | Dec 31 |
| Jan 6 | 4 | Jan 2 |
| Jan 7 | 5 | Jan 2 |
| Jan 10 | 6 | Jan 4 |

- Rows sharing the same result = same **island**.
- This result column is called the **island key** (a "fingerprint" for each run).

### Mental Model
>
> The row number is a smooth ramp (+1 each step). The real sequence is a bumpy ramp (sometimes smooth, sometimes jumps). Subtracting the smooth ramp from the bumpy one **flattens consecutive runs into a constant** and **exposes breaks as new constants**.
>
> **Flat = island. Jump = gap.**

- This same logic works for **integers** too (e.g., order IDs): `order_id − ROW_NUMBER()`.

---

## Writing the SQL (Clause by Clause)

**Setup:** Table `user_login(user_id, login_date)`. Goal: find start, end, and length of every streak per user.

### Step 1 — CTE to compute row number + island key

```sql
WITH numbered AS (
  SELECT
    user_id,
    login_date,
    login_date - ROW_NUMBER() OVER (
      PARTITION BY user_id
      ORDER BY login_date ASC
    ) * INTERVAL '1 day' AS island_key
  FROM user_login
)
```

**Key clause notes:**

- `PARTITION BY user_id` — **essential**. Without it, row numbers run across the whole table, mixing users together and corrupting the island key.
- `ORDER BY login_date ASC` — **non-negotiable**. This must be the sequence column being analyzed; changing it breaks the island key.
- The date arithmetic shifts the login date backward by `row_number` days.

### Step 2 — Outer query groups by island key

```sql
SELECT
  user_id,
  MIN(login_date) AS streak_start,
  MAX(login_date) AS streak_end,
  COUNT(*) AS streak_length
FROM numbered
GROUP BY user_id, island_key
```

- Grouping by `(user_id, island_key)` collapses each run into one row.
- `MIN` = streak start, `MAX` = streak end, `COUNT(*)` = streak length.

### Database-Specific Syntax for Date Arithmetic

| Database | Approach |
|----------|----------|
| Postgres | `login_date - row_number` (direct subtraction works) |
| MySQL | `DATE_SUB()` function with interval |
| Snowflake / BigQuery | `DATE_ADD()` with a negative value |

- If unsure which engine an interviewer uses, just ask — most care about the idea, not exact dialect syntax.

### For Integer Sequences

- Drop the interval multiplication entirely.
- Island key = `id - ROW_NUMBER()`. Everything else stays the same.

---

## Four Pitfalls That Break This Pattern

1. **Forgetting `PARTITION BY` for per-entity problems**
   - If the question says "for each user/customer/account," you must partition by that column.
   - Without it, row numbers run across the whole table and mash different users into the same streak — silently, with no error.

2. **Wrong ordering inside `OVER()`**
   - Always order by the column that's supposed to be contiguous (e.g., login date for date streaks, order ID for ID sequences).
   - Wrong ordering produces meaningless row numbers and a noisy island key.

3. **Date arithmetic that doesn't translate across databases**
   - Postgres allows direct date-integer subtraction; MySQL needs `DATE_SUB`; Snowflake/BigQuery need `DATE_ADD` with negative values.
   - If uncertain, describe the logic verbally ("shift date back by row number days") — most interviewers accept this.

4. **Duplicate rows silently breaking row numbers**
   - Duplicate rows (e.g., two rows for the same login date) get different row numbers despite representing the same point in the sequence, corrupting the island key.
   - **Fix:** Deduplicate first — wrap raw data in a CTE using `SELECT DISTINCT` before running `ROW_NUMBER()`.

**Common theme:** All four pitfalls come from violating the requirement of **clean, ordered, one-row-per-sequence-point input**.

---

## Interview Question 1 (Meta): Longest Consecutive Login Streak

**Prompt:** For each user, find the longest streak of consecutive login days. Return user ID, streak start, streak end, and length. If there's a tie, return the earliest streak.

**Approach:** Three layered stages:

1. **`numbered`** CTE — assign row numbers per user + compute island key.
2. **`streaks`** CTE — group by `(user_id, island_key)` to collapse each run into one summary row (start, end, length).
3. **Outer query** — rank streaks per user by length (descending), using streak start (ascending) as a tiebreaker, then keep only rank 1.

```sql
WITH numbered AS (
  SELECT
    user_id,
    login_date,
    login_date - ROW_NUMBER() OVER (
      PARTITION BY user_id ORDER BY login_date ASC
    ) * INTERVAL '1 day' AS island_key
  FROM user_login
),
streaks AS (
  SELECT
    user_id,
    MIN(login_date) AS streak_start,
    MAX(login_date) AS streak_end,
    COUNT(*) AS streak_length
  FROM numbered
  GROUP BY user_id, island_key
),
ranked AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY user_id
      ORDER BY streak_length DESC, streak_start ASC
    ) AS streak_rank
  FROM streaks
)

SELECT user_id, streak_start, streak_end, streak_length
FROM ranked
WHERE streak_rank = 1
```

**Why three layers?**

- Layer 1: generate island key.
- Layer 2: collapse runs into streak summaries.
- Layer 3: pick the longest streak per user.
- Keeping them separate is the most readable approach — readability matters in interviews.

**Edge cases to mention:**

- Users with **no logins** won't appear in the result (they're not in the source table). To include them, left join against a full users table and coalesce nulls to zero.
- **Tiebreaker preference** varies (earliest vs. most recent tied streak) — clarify with the interviewer up front to avoid rework.

---

## Interview Question 2 (Google): Find Every Missing Order-ID Range (Using LEAD)

**Prompt:** Table `orders(order_id)` should have contiguous integers starting from 1, but some IDs are missing due to failed inserts/deletions. Find every missing range, returning `gap_start` and `gap_end` (one row per contiguous missing range).

**Approach:** Use `LEAD()` instead of row-number-difference — this is the "gaps" side of the pattern (opposite of "islands").

- `LEAD()` lets a row peek at the value in the **next** row (opposite of `LAG()`, which looks backward).
- Core logic: for each row, compare its ID to the next row's ID. If the difference is more than 1, everything in between is a missing range.

```sql
WITH with_next AS (
  SELECT
    order_id,
    LEAD(order_id) OVER (ORDER BY order_id ASC) AS next_id
  FROM orders
)
SELECT
  order_id + 1 AS gap_start,
  next_id - 1 AS gap_end
FROM with_next
WHERE next_id - order_id > 1
```

**How it works:**

- `next_id - order_id > 1` → filters for real gaps (adjacent existing IDs differing by more than 1 = gap; exactly 1 = no gap).
- `gap_start = order_id + 1` → gap begins right after current ID.
- `gap_end = next_id - 1` → gap ends right before the next existing ID.
- Contiguous missing IDs automatically collapse into **one row** — no extra grouping needed.
- The last row's `next_id` is `NULL`; any arithmetic with `NULL` is `NULL`, which fails the `> 1` filter, so it's automatically excluded.

**Example trace:**
IDs present: 1, 2, 5, 6, 7, 10, 15
→ Missing ranges found: **3–4**, **8–9**, **11–14** (four missing IDs 11–14 collapse into a single row).

**Edge case to mention:**

- If the sequence should start at 1 but the smallest ID present is greater than 1 (or should end at some max but the largest ID is smaller), this query **won't catch those boundary gaps** since there's no adjacent row to compare against.
- **Fix:** Union in sentinel rows at the start/end, or check against the expected min/max separately.

---

## Islands vs. Gaps — Choosing the Right Tool

| Problem type | Technique |
|---|---|
| Find **consecutive runs** ("streak," "consecutive") | `ROW_NUMBER()` difference (island key) |
| Find **missing ranges** ("gap," "missing range," "contiguous") | `LEAD()` (or `LAG()`) |

These are two complementary techniques for two sides of the same underlying pattern.

---

## Key Takeaway

When you hear **"consecutive," "streak," "missing range,"** or **"contiguous"** in an interview question:

- Stop thinking about individual rows — start thinking about **sequences**.
- Use **row-number difference** to find runs (islands).
- Use **LEAD/LAG** to find breaks (gaps).
