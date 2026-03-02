
## Table of contents (Phase 2)

1. Advanced joins (LATERAL, USING, NATURAL, CROSS APPLY pattern)
2. `EXISTS` vs `IN` vs `JOIN` (performance & correctness)
3. `LATERAL` / set-returning functions / `jsonb` expansion
4. Common anti-patterns (cartesian explosion, SELECT *)
5. Common Table Expressions (CTE) — materialization, recursive CTEs
6. Window functions — deep (ROW_NUMBER / RANK / LEAD / LAG / aggregates over window)
7. Top-N per group patterns (RANK vs ROW_NUMBER)
8. `EXPLAIN` / `EXPLAIN ANALYZE` — how to read query plans (practical)
9. Index basics — types, when to use, multicolumn order, partial & expression & INCLUDE indexes, index-only scans
10. Index maintenance, VACUUM/ANALYZE, statistics (brief)
11. Quick Exercises (with answers)

---

## 1 — Advanced Joins

### Concept

Joins are set operations that combine rows. Beyond inner/outer joins, PostgreSQL supports patterns (LATERAL) and syntactic sugars (USING, NATURAL) with important semantics.

### USING vs ON

* `USING(col)` merges the join column into a single column in the output (no duplicated `col`).
* `ON` allows arbitrary join conditions.

**Syntax**

```sql
SELECT * FROM a JOIN b USING (id);
SELECT * FROM a JOIN b ON a.id = b.id AND b.flag = true;
```

**Pitfall:** `USING` hides which table the column came from; avoid when ambiguous.

### NATURAL JOIN

* Joins on all identically named columns automatically — fragile; **avoid** in production and interviews. It breaks when schema changes.

### LATERAL (the important one)

* `LATERAL` allows the right-hand side of the FROM to reference columns from the left-hand side per row — effectively a per-row subquery that can return multiple rows.

**Syntax**

```sql
SELECT u.id, r.*
FROM users u
CROSS JOIN LATERAL (
  SELECT * FROM orders o
  WHERE o.user_id = u.id
  ORDER BY o.created_at DESC LIMIT 1
) r;
```

**Use cases**

* Top-N per row, expanding JSON arrays per row, calling set-returning functions that take left-side columns.

**MySQL comparison**

* MySQL lacks `LATERAL` until recent versions with `LATERAL`/`JSON_TABLE`; Postgres `LATERAL` is powerful and standard.

**Interview trap**

* Don’t confuse `LATERAL` with `CROSS APPLY` semantics in other DBs — `LATERAL` is equivalent to CROSS APPLY.

---

## 2 — EXISTS vs IN vs JOIN

### Correctness differences

* `IN (subquery)` compares set values; if subquery returns NULLs, semantics can be surprising unless values are NOT NULL.
* `EXISTS (correlated_subquery)` returns true/false; it's row-oriented and short-circuits when a match is found.

### Performance guidance

* For correlated checks, `EXISTS` is usually efficient.
* For de-duplicated small list checks, `IN (const list)` is fine.
* For retrieving columns from the related table, prefer `JOIN` (with necessary dedup/aggregation).

**Example**

```sql
-- existence check
SELECT * FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.id AND o.total > 100
);
```

**Pitfall**

* `IN (SELECT col FROM big_table)` can be slower when `col` is not indexed or when subquery produces many rows. Compare plans with `EXPLAIN`.

---

## 3 — LATERAL, set-returning functions, jsonb expansion

### Example: expand jsonb array into rows with LATERAL

```sql
SELECT p.id, elem ->> 'name' AS tag
FROM posts p
CROSS JOIN LATERAL jsonb_array_elements(p.tags_jsonb) AS elem;
```

**Use cases**

* Normalize a JSON array per row
* Call function that returns rows per input row

**Pitfall**

* LATERAL queries can be expensive per-row; ensure left set is limited or indexed.

---

## 4 — Common anti-patterns (short list)

* `SELECT *` in production queries (index-only scans and planner optimizations fail).
* Cartesian products due to missing join condition — always validate join predicates.
* Using correlated subqueries where a single JOIN + GROUP BY suffices (or window function), leading to N+1 behavior.

---

## 5 — Common Table Expressions (CTE)

### Non-recursive CTE

**Syntax**

```sql
WITH recent_orders AS (
  SELECT * FROM orders WHERE created_at > now() - interval '7 days'
)
SELECT r.customer_id, COUNT(*) FROM recent_orders r GROUP BY r.customer_id;
```

### Recursive CTE

* Useful for hierarchical traversals (trees, graphs).
  **Example — employee hierarchy**

```sql
WITH RECURSIVE org AS (
  SELECT id, name, manager_id, 1 AS level
  FROM employees WHERE manager_id IS NULL

  UNION ALL

  SELECT e.id, e.name, e.manager_id, o.level + 1
  FROM employees e
  JOIN org o ON e.manager_id = o.id
)
SELECT * FROM org;
```

### Materialization caveat (important interview point)

* Historically, Postgres materialized CTEs (optimization fence) — meaning the CTE was evaluated once, results stored, not inlined; this could hurt performance.
* Since Postgres 12+, planner can inline non-recursive CTEs when safe (improves performance). Still, explicit `MATERIALIZED` / `NOT MATERIALIZED` options exist to force behavior.

**Syntax**

```sql
WITH t AS MATERIALIZED ( ... )
WITH t AS NOT MATERIALIZED ( ... )
```

**Interview trap**

* Don’t assume CTEs act like subqueries; mention materialization behaviour and version-dependent changes in interviews.

---

## 6 — Window functions — deep

### Concept

Window functions compute a value across a set of rows related to the current row (without collapsing rows). They are crucial for "top-N per group", running totals, moving averages, etc.

### Core syntax

```sql
<window_func>() OVER (
  [PARTITION BY expr,...]
  [ORDER BY expr [ASC|DESC] [NULLS {FIRST|LAST}], ...]
  [ROWS|RANGE frame_spec]
)
```

### Common functions

* `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE(n)`
* `LAG(expr [, offset [, default]])`, `LEAD(expr [, offset [, default]])`
* Aggregate variants: `SUM() OVER (...)`, `AVG() OVER (...)`

### Examples

**ROW_NUMBER for top-N per group**

```sql
SELECT *
FROM (
  SELECT e.*,
         ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn
  FROM employees e
) t
WHERE rn <= 3;
```

**Running total**

```sql
SELECT id, created_at, amount,
       SUM(amount) OVER (ORDER BY created_at ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM payments
ORDER BY created_at;
```

**LEAD / LAG**

```sql
SELECT id, value,
       LAG(value) OVER (ORDER BY id) AS prev_value,
       LEAD(value) OVER (ORDER BY id) AS next_value
FROM series;
```

### Frame types: ROWS vs RANGE

* `ROWS` frames are defined by physical offset (rows).
* `RANGE` frames are logical, use ORDER BY values; be careful with duplicates/NULLs.

### Performance

* Window functions can cause sorts; ensure appropriate indexes for ORDER BY to help planner.

### Interview traps

* Difference between `ROW_NUMBER`, `RANK`, and `DENSE_RANK`:

  * `ROW_NUMBER`: unique consecutive integers — breaks ties arbitrarily.
  * `RANK`: ties receive same rank; gaps appear in ranking.
  * `DENSE_RANK`: ties receive same rank; no gaps.
* Using `ROW_NUMBER` to remove duplicates vs `RANK` will change tie behavior — choose deliberately.

---

## 7 — Top-N per group patterns (recipes and tradeoffs)

### Preferred: Window functions (clean, efficient)

See ROW_NUMBER example above.

### Alternative: DISTINCT ON (Postgres-specific)

* `DISTINCT ON` returns the first row per distinct key in the ORDER BY sequence.
  **Example:**

```sql
SELECT DISTINCT ON (dept_id) dept_id, id, salary
FROM employees
ORDER BY dept_id, salary DESC;
```

**Important:** `DISTINCT ON` is concise and often faster than window function for single-row-per-group use-cases. But the ordering must include the `DISTINCT ON` columns first.

**Interview trap**

* `DISTINCT ON` is PostgreSQL-specific — mention portability tradeoff.

---

## 8 — EXPLAIN / EXPLAIN ANALYZE — reading the plan

### Two commands

* `EXPLAIN <query>;` — shows estimated plan (no execution).
* `EXPLAIN ANALYZE <query>;` — runs query and shows actual timing and row counts (useful for diagnosing).

### What to look for (practical)

* **Seq Scan vs Index Scan** — full table scan vs index usage. Seq scan may be fine for large fraction reads.
* **Index Only Scan** — possible when index covers all needed columns and visibility map is up-to-date.
* **Nested Loop, Merge Join, Hash Join** — join strategy chosen:

  * `Nested Loop`: good for small inner set, else expensive.
  * `Hash Join`: builds hash of one side; memory and build cost.
  * `Merge Join`: requires sorted inputs (good for ordered ranges).
* **Estimated rows vs actual rows** — large divergence signals bad statistics; run `ANALYZE` or adjust queries/indexes.
* **Buffers and I/O** (use `EXPLAIN (ANALYZE, BUFFERS, VERBOSE)` for more details).

### Example: bad vs good

Bad (no index, many rows):

```
Seq Scan on orders  (cost=0.00..20000 rows=100000 width=32)
```

Good (indexed):

```
Index Scan using orders_user_id_idx on orders  (cost=0.27..11.00 rows=5 width=32)
```

### Practical debugging steps

1. Run `EXPLAIN ANALYZE` to see actual time and counts.
2. If actual >> estimated, check `ANALYZE` and statistics, data skew, and indexes.
3. Consider rewriting query (join order, lateral, CTE removal/inlining).
4. Use `SET enable_seqscan = off` temporarily for experiments only (not in production).

### Interview traps

* Don’t suggest forcing an index globally — explain costs and safer steps (analyze, create appropriate indexes, rewrite query).

---

## 9 — Index basics (types and usage)

### Why indexes

* Speed up lookups, join matches, ORDER BY, and grouping — at the cost of write overhead and storage.

### Major index types in Postgres

1. **B-Tree (default)** — equality and range queries (`=`, `<`, `>`, `BETWEEN`, `ORDER BY`). Use for primary keys and common filters.
2. **Hash** — equality only; historically limited, but improved in recent versions — prefer B-tree unless special case.
3. **GIN (Generalized Inverted Index)** — excellent for `jsonb`, full-text `tsvector`, arrays, and containment (`@>`). Good for multi-key indexing.

   * Example: `CREATE INDEX idx_posts_tags ON posts USING GIN (tags_jsonb);`
4. **GiST (Generalized Search Tree)** — spatial and nearest-neighbour indexes (PostGIS), range types, full-text search extension components.
5. **BRIN (Block Range Index)** — tiny index for very large, append-only tables where correlated physical order exists (time-series data).

### Multicolumn index order matters

* For index `(a, b)`, queries filtering by `a` or by both `a` and `b` use index; filtering by `b` only does not use the index (except index-only when equality and special cases).
* For ORDER BY, index can support order if ordering matches index key order and direction.

### Partial index

* Index only rows matching WHERE clause:

```sql
CREATE INDEX idx_active_users_email ON users (email) WHERE active;
```

* Useful when majority rows are irrelevant for queries.

### Expression/index with INCLUDE (covering index)

* Expression index:

```sql
CREATE INDEX idx_lower_email ON users (LOWER(email));
```

* INCLUDE clause stores non-key columns in the index for index-only scans:

```sql
CREATE INDEX idx_orders_userid_include_total ON orders (user_id) INCLUDE (total);
```

### Index-only scans

* If the index contains all columns needed and visibility map is clean, planner can use index-only scan (avoids heap fetch).

### Unique indexes vs constraints

* `UNIQUE` constraint builds a unique index.

### Creating indexes concurrently (production-safe)

```sql
CREATE INDEX CONCURRENTLY idx_name ON table (col);
```

* Non-blocking for writes, but cannot be run inside a transaction block.

### Interview traps / tradeoffs

* More indexes → slower writes and more disk usage. Ask: read-heavy vs write-heavy workload?
* BRIN useful for huge append-only logs; small memory footprint but only helps when physical order correlates to query predicate.
* Partial + expression indexes are powerful — prefer them to brute-force indexing.

---

## 10 — Index maintenance, VACUUM/ANALYZE, statistics

### VACUUM

* Reclaims space from dead tuples. `autovacuum` runs automatically; tune thresholds for heavy-write tables.

### VACUUM FULL

* Rewrites the table, releases disk; requires exclusive lock — avoid during business hours.

### ANALYZE

* Updates statistics used by planner. Run `ANALYZE` after bulk loads or major distribution changes.

### Visibility map & index-only scans

* Index-only scans require visible tuples; `vacuum` sets visibility map bits.

### Reindex

* Rebuilds an index if corrupted or bloated:

```sql
REINDEX INDEX idx_name;
```

### Monitoring

* `pg_stat_user_tables`, `pg_stat_user_indexes` provide statistics (hot updates, dead tuples).
* Check `pg_stat_activity` for long-running transactions that prevent vacuum cleanup.

### Interview traps

* Long-running idle-in-transaction sessions block VACUUM, cause table bloat — mention this explicitly and how to fix (kill/IBTX/notify team).

---

## 11 — Quick Exercises (Phase 2) — with answers

### E1 — Distinct top per group with `DISTINCT ON`

**Question:** Return the latest order per customer (orders table with `customer_id`, `created_at`).
**Answer:**

```sql
SELECT DISTINCT ON (customer_id) customer_id, id, created_at, total
FROM orders
ORDER BY customer_id, created_at DESC;
```

**Why?** Efficient and concise in Postgres.

---

### E2 — Convert correlated subquery to window function

**Question:** For `sales (id, seller_id, amount)`, find each sale with its seller's average sale amount.
**Window answer:**

```sql
SELECT id, seller_id, amount,
       AVG(amount) OVER (PARTITION BY seller_id) AS seller_avg
FROM sales;
```

---

### E3 — Recursive CTE — path from node X to root

**Question:** Given `tree(id, parent_id)`, produce path from node `42` to root.
**Answer:**

```sql
WITH RECURSIVE path AS (
  SELECT id, parent_id, ARRAY[id] AS nodes FROM tree WHERE id = 42
  UNION ALL
  SELECT t.id, t.parent_id, p.nodes || t.id
  FROM tree t
  JOIN path p ON t.id = p.parent_id
)
SELECT nodes FROM path WHERE parent_id IS NULL;
```

---

### E4 — JSONB indexing with containment

**Question:** Query posts where tags JSONB contains `"python"` and make it fast.
**Answer:**

```sql
CREATE INDEX idx_posts_tags_gin ON posts USING GIN (tags_jsonb);
SELECT * FROM posts WHERE tags_jsonb @> '["python"]'::jsonb;
```

---

### E5 — Identify why a query unexpectedly does seq scan

**Question:** You expected an index scan on `orders(user_id)` but plan shows seq scan. What to check?
**Checklist answer:**

* Were statistics updated? Run `ANALYZE orders;`
* Cardinality: maybe the predicate returns a large fraction → seq scan cheaper.
* Check `EXPLAIN (ANALYZE, BUFFERS)` for actual hits and estimated rows.
* Visibility map: recent massive updates prevent index-only scans.
* Parameter sniffing / prepared statements might cause different plans.

---

## Final — Phase 2 summary (action items and strict checklist)

1. Read plans with `EXPLAIN ANALYZE` — compare estimated vs actual rows. If off by >10x, update stats or re-evaluate indexes.
2. Use window functions for top-N per group and running totals; use `DISTINCT ON` when you need a single row per group in Postgres and want concise syntax.
3. Use `LATERAL` for per-row expansion (top-N per left row, jsonb expansion).
4. Index smartly: think about query patterns, cardinality, and write cost. Prefer expression/partial indexes over global indexes when appropriate.
5. Monitor autovacuum, long-running transactions, and bloat.

---
