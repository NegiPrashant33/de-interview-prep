# Collection Aggregates — SQL Concept

## What It Is

Most aggregate functions (`SUM`, `COUNT`, `AVG`) collapse a group of rows into a single **scalar value**. **Collection aggregates** instead collapse a group of rows into a single **collection** — an array, string, struct, or other multi-value type — capturing every value from that column rather than reducing it to one number.

No information is lost; it's just reshaped from many rows into one compound value per group.

This is a general SQL **concept**, not one specific pattern — it covers a whole family of functions across engines: `ARRAY_AGG`, `STRING_AGG`, `GROUP_CONCAT`, `LISTAGG`, `COLLECT_LIST`, and similar. They all follow the same idea: gather grouped values into one collection of a given type (array, string, struct, etc.) — the difference is just output type and dialect.

---

## Core Behavior

- Works with standard `GROUP BY` — one collection produced per group, just like `SUM` produces one number per group.
- Can also run as a **window function** (`OVER (PARTITION BY ...)`) to attach the full group's collection to every row without collapsing them.
- Order of elements inside the collection is **not guaranteed** unless you specify an `ORDER BY` inside the aggregate call.
- Add `DISTINCT` inside the aggregate to drop duplicate values.
- The output type generalizes beyond strings/arrays of scalars — many engines let you aggregate into arrays of structs/rows, not just single-column values.
- NULL handling varies by function/engine — check or wrap with `COALESCE` if it matters.

---

## Common Pitfalls

1. **Unpredictable order inside the collection**
   - Without an internal `ORDER BY`, element order depends on however the engine scans rows. Always add `ORDER BY` inside the aggregate if order matters.

2. **Forgetting `DISTINCT` when duplicates shouldn't appear**
   - If underlying rows repeat a value, it's included every time by default.

3. **NULLs sneaking into the collection**
   - Some functions include NULLs as elements, others silently drop them — behavior differs by engine. Verify or wrap with `COALESCE`.

4. **Row-count explosion from a prior JOIN**
   - If you join before aggregating and the join fans out rows, values get duplicated in the resulting collection. Aggregate first (e.g., in a CTE), then join.

5. **Confusing this with the reverse operation**
   - Collection aggregates *collapse* rows into a collection. Expanding a collection back into rows is the opposite operation — `UNNEST` in Postgres/BigQuery. Interviewers sometimes chain both directions in one question.

---

## Example 1 — PostgreSQL: `ARRAY_AGG`

```sql
SELECT
  customer_id,
  ARRAY_AGG(DISTINCT product_name ORDER BY product_name) AS products_ordered
FROM orders
GROUP BY customer_id;
```

- Returns a native array type, e.g. `{Keyboard, Laptop, Mouse}`.
- `DISTINCT` removes duplicate products; `ORDER BY` guarantees alphabetical order within the array.
- Postgres also offers `STRING_AGG(product_name, ', ')` for a delimited string instead of an array.

## Example 2 — BigQuery: `ARRAY_AGG`

```sql
SELECT
  customer_id,
  ARRAY_AGG(DISTINCT product_name ORDER BY product_name) AS products_ordered
FROM orders
GROUP BY customer_id;
```

- Same function and syntax as Postgres — BigQuery returns a `REPEATED` array field.
- BigQuery also supports aggregating structs directly, e.g. `ARRAY_AGG(STRUCT(product_name, order_date))`, letting you collect multiple related columns per row into one array of records.
- For a delimited string instead, BigQuery offers `STRING_AGG(product_name, ', ')`.

---

## Interview Question 1: List of Products Per Customer

**Prompt:** Table `orders(order_id, customer_id, product_name, order_date)`. For each customer, return an alphabetically sorted array of the distinct products they've ordered.

```sql
-- Works identically in PostgreSQL and BigQuery
SELECT
  customer_id,
  ARRAY_AGG(DISTINCT product_name ORDER BY product_name) AS products_ordered
FROM orders
GROUP BY customer_id;
```

---

## Interview Question 2: Collection Aggregate as a Window Function

**Prompt:** Table `orders(order_id, order_date, amount)`. For each row, show the order's own detail *plus* an array of all order IDs that happened on the same date.

```sql
-- PostgreSQL
SELECT
  order_id,
  order_date,
  amount,
  ARRAY_AGG(order_id) OVER (
    PARTITION BY order_date
    ORDER BY order_id
  ) AS order_ids_same_day
FROM orders
ORDER BY order_date, order_id;
```

```sql
-- BigQuery equivalent
SELECT
  order_id,
  order_date,
  amount,
  ARRAY_AGG(order_id) OVER (
    PARTITION BY order_date
    ORDER BY order_id
  ) AS order_ids_same_day
FROM orders
ORDER BY order_date, order_id;
```

- Using the **window function** form instead of `GROUP BY` preserves one row per original order while still attaching the full day's list to each row.

---

## Key Takeaway

Whenever a question asks to "list all X per group," "collect values into an array/string," or "capture every value of a column into one structure," reach for a collection aggregate (`ARRAY_AGG`/`STRING_AGG` or the engine's equivalent) instead of hacking it together with string concatenation or extra joins — it's the native SQL way to turn grouped rows into a single compound value of any type.
