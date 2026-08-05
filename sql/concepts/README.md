# SQL Interview Notes

Personal SQL revision notes, organized by concept. Written mostly from a MySQL
perspective (some behaviors, like `SUM(condition)` returning 0/1, are
MySQL-specific rather than standard ANSI SQL).

## Contents

1. [Aggregate Functions Basics](01-aggregate-functions.md) — `COUNT(*)` vs
   `COUNT(col)` vs `COUNT(DISTINCT col)`, boolean-to-int tricks
2. [Query Execution Order](02-query-execution-order.md) — the logical order
   SQL actually evaluates clauses in, with and without window functions
3. [SELECT, DISTINCT & GROUP BY](03-select-distinct-groupby.md) — what
   `DISTINCT` really operates on, sorting behavior, ordering by aggregates
4. [WHERE vs HAVING](04-where-vs-having.md)
5. [Functional Dependency (MySQL GROUP BY exception)](05-functional-dependency-mysql.md)
6. [Window Functions — Full Reference](06-window-functions.md) — aggregate,
   ranking, and value window functions; when `PARTITION BY`/`ORDER BY`/frame
   are required vs optional
7. [Common Pitfalls & Gotchas](07-common-pitfalls.md) — chained comparisons
   and other traps

## How to use this

Each file is self-contained — read whichever concept you need to refresh
before an interview. The window functions file (06) is the densest one;
it's worth a full re-read since ranking/value functions come up constantly
in interview SQL questions (top-N per group, running totals, etc.).
