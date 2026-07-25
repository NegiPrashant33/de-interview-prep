# Top N Per Group — SQL Interview Pattern

## Overview

**Top N per Group** is one of the most heavily tested SQL patterns in FAANG interviews. Classic examples: "top 3 highest-paid employees per department," "most recent order per customer," "top-selling product per category."

The naive instinct is to write a separate query per group or use nested subqueries with `MAX`/`LIMIT`. The clean, scalable solution is **ranking window functions** partitioned by group.

---

## Core Concept

- We want to rank rows **within each group** (not across the whole table), then keep only the top N ranks.
- This requires two things:
  1. **`PARTITION BY`** the group column — resets ranking for each group independently.
  2. **`ORDER BY`** the column that defines "top" (e.g., salary descending, order date descending).
- Once each row has a rank within its group, filter with `WHERE rank <= N`.

**Mental model:** Same "independent windows per partition" idea from running totals/moving averages — except instead of aggregating, we're **assigning a rank** and keeping only the best ones.

---

## The Three Ranking Functions

| Function | Behavior on ties | Skips ranks after a tie? |
|---|---|---|
| `ROW_NUMBER()` | Assigns a unique, arbitrary rank to each row — no ties possible | N/A |
| `RANK()` | Ties get the same rank | Yes (e.g., 1, 2, 2, 4) |
| `DENSE_RANK()` | Ties get the same rank | No (e.g., 1, 2, 2, 3) |

**Choosing the right one is the crux of this pattern:**

- "Exactly top 3 rows, no more, no less" (even with ties) → `ROW_NUMBER()`.
- "Top 3 distinct salary levels, include everyone tied at rank 3" → `RANK()` or `DENSE_RANK()`.
- If the interviewer doesn't specify, **ask** whether ties should produce extra rows or be broken arbitrarily.

---

## Basic Syntax

```sql
SELECT *
FROM (
  SELECT
    employee_id,
    department,
    salary,
    ROW_NUMBER() OVER (
      PARTITION BY department
      ORDER BY salary DESC
    ) AS rnk
  FROM employees
) ranked
WHERE rnk <= 3
```

**Clause breakdown:**

- `PARTITION BY department` — resets the ranking counter for each department independently.
- `ORDER BY salary DESC` — defines what "top" means; highest salary gets rank 1.
- Must wrap in a subquery/CTE — **window functions cannot be filtered directly in the same `SELECT`'s `WHERE` clause** (they're computed after `WHERE` runs).
- `WHERE rnk <= 3` in the outer query keeps only the top 3 per department.

### Using RANK() Instead (ties included)

```sql
RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
...
WHERE rnk <= 3
```

- If two people tie for 2nd place, both get rank 2, and the next person gets rank 4 — so you could get more than 3 rows per department.

---

## Common Pitfalls

1. **Forgetting `PARTITION BY`**
   - Without it, ranking runs across the *entire table*, not per group — you'll get the global top N, not top N per group. Query runs fine, just silently wrong.

2. **Using `WHERE` directly on the window function in the same query**
   - Window functions are evaluated after `WHERE`/`GROUP BY`, so `WHERE rnk <= 3` in the same `SELECT` as the window function fails or errors depending on the engine.
   - Always wrap in a subquery or CTE and filter in the outer layer.

3. **Picking the wrong ranking function for ties**
   - Using `ROW_NUMBER()` when the question implies ties should all be included (or vice versa) gives a technically-running-but-wrong answer.
   - Always clarify tie-handling expectations with the interviewer.

4. **Wrong `ORDER BY` direction**
   - "Top" almost always means `DESC` (highest first) but always double check — "top" earliest date, for example, may mean `ASC`.

5. **Using `LIMIT`/`TOP` instead of a window function**
   - `LIMIT N` only works for a single global top N — it cannot express "top N *per group*" on its own. A common broken attempt: `GROUP BY` + `LIMIT`, which doesn't apply per-group limits correctly.

6. **Reaching for correlated subqueries or self-joins**
   - Works but is slow (re-scans the table per group/row). Window functions solve this in a single pass.

---

## Interview Question 1: Top 3 Highest-Paid Employees Per Department

**Prompt:** Table `employees(employee_id, name, department, salary)`. Return the top 3 highest-paid employees in each department.

```sql
WITH ranked AS (
  SELECT
    employee_id,
    name,
    department,
    salary,
    DENSE_RANK() OVER (
      PARTITION BY department
      ORDER BY salary DESC
    ) AS salary_rank
  FROM employees
)
SELECT employee_id, name, department, salary
FROM ranked
WHERE salary_rank <= 3
ORDER BY department, salary DESC
```

- Used `DENSE_RANK()` here since "top 3 highest-paid" typically implies ties should be included together (all employees tied for 3rd place still count as top 3).
- Mention to the interviewer: if they instead want *exactly* 3 rows per department regardless of ties, swap to `ROW_NUMBER()`.

---

## Interview Question 2: Most Recent Order Per Customer

**Prompt:** Table `orders(order_id, customer_id, order_date, amount)`. Return each customer's most recent order.

```sql
WITH ranked AS (
  SELECT
    order_id,
    customer_id,
    order_date,
    amount,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY order_date DESC
    ) AS rn
  FROM orders
)
SELECT order_id, customer_id, order_date, amount
FROM ranked
WHERE rn = 1
```

- `ROW_NUMBER()` is the right choice here — "the most recent order" implies exactly **one row per customer**, even if two orders share the same timestamp (pick one arbitrarily, or add a tiebreaker like `order_id DESC`).
- **Edge case to mention:** if exact ties on `order_date` should both be returned as "most recent," switch to `RANK()` instead.

---

## Key Takeaways

- **Top N per Group = `PARTITION BY` (the group) + `ORDER BY` (the "top" criterion) + a ranking function, filtered in an outer query.**
- Pick the ranking function based on how ties should behave:
  - `ROW_NUMBER()` → exactly N rows per group, ties broken arbitrarily.
  - `RANK()` / `DENSE_RANK()` → ties are preserved, possibly returning more than N rows.
- Never filter a window function in the same `SELECT`'s `WHERE` clause — always wrap in a subquery/CTE.
- Whenever a question asks for "top N," "most recent," "highest," or "best per X," this is your signal to reach for a ranking window function partitioned by the group.
