
# Table of Contents (Phase 3)

1. MVCC — Deep Internal Model
2. Transaction Isolation Levels (PostgreSQL Reality)
3. Locking & Deadlocks
4. Write-Ahead Logging (WAL) & Checkpoints
5. Query Planner Internals
6. Statistics & Cardinality Estimation
7. Partitioning (Declarative)
8. Advanced Indexing Strategy (GIN/GiST/BRIN Deep Dive)
9. Full-Text Search (Brief but Interview-Relevant)
10. JSONB Performance Strategy
11. Concurrency Patterns & Production Pitfalls
12. Interview Kill-Questions

---

# 1 — MVCC (Deep Internal Model)

## Core Idea

PostgreSQL never overwrites a row.

When you `UPDATE`:

* Old row version remains.
* New row version is inserted.
* Each row version contains metadata:

  * `xmin` → transaction that created it
  * `xmax` → transaction that deleted it

Each transaction sees rows based on its **snapshot**.

This enables:

* Readers do not block writers.
* Writers do not block readers.

---

## Example Timeline

Transaction T1:

```sql
BEGIN;
UPDATE users SET balance = 100 WHERE id = 1;
-- not committed yet
```

Transaction T2:

```sql
SELECT balance FROM users WHERE id = 1;
```

T2 sees **old committed value**, not uncommitted update.

Why? Snapshot isolation.

---

## Consequence: Dead tuples

Every update leaves behind dead versions.

Requires:

```sql
VACUUM;
```

Autovacuum cleans automatically.

---

## Interview Traps

* Q: Why do large UPDATE operations cause table bloat?
  → Because new versions are inserted; old ones remain until vacuum.

* Q: Why are long-running transactions dangerous?
  → They prevent vacuum from cleaning dead tuples.

* Q: Does Postgres use read locks?
  → No. It uses MVCC for readers.

---

# 2 — Transaction Isolation Levels (Postgres Specific)

Supported levels:

1. READ COMMITTED (default)
2. REPEATABLE READ
3. SERIALIZABLE

Postgres does NOT implement READ UNCOMMITTED separately (it behaves like READ COMMITTED).

---

## READ COMMITTED

Each query sees snapshot of committed data at start of query.

Non-repeatable reads possible.

---

## REPEATABLE READ

All queries in transaction see same snapshot.

No non-repeatable reads.
Phantom reads prevented in Postgres due to MVCC implementation.

---

## SERIALIZABLE

Uses Serializable Snapshot Isolation (SSI).
Detects dangerous patterns and aborts conflicting transactions.

---

## Example

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
```

If conflict detected:

```
ERROR: could not serialize access due to concurrent update
```

You must retry transaction.

---

## Interview Trap

* MySQL InnoDB “Repeatable Read” behaves differently.
* PostgreSQL SERIALIZABLE is stricter (uses SSI, not simple locking).

---

# 3 — Locking & Deadlocks

Even with MVCC, locks exist:

* Row-level locks (UPDATE/DELETE)
* Table-level locks (ALTER TABLE, VACUUM FULL)

---

## Lock Types (Simplified)

* ACCESS SHARE → SELECT
* ROW EXCLUSIVE → INSERT/UPDATE/DELETE
* ACCESS EXCLUSIVE → DROP/ALTER

---

## Deadlock Example

T1 updates row A then B
T2 updates row B then A

→ circular wait → deadlock

Postgres detects and kills one transaction.

---

## View Locks

```sql
SELECT * FROM pg_locks;
SELECT * FROM pg_stat_activity;
```

---

## Interview Trap

* Deadlocks are not bugs — they’re detected and resolved by killing one transaction.
* Always lock resources in consistent order.

---

# 4 — WAL (Write-Ahead Logging)

## Principle

Changes written to WAL before data file.

Guarantees durability.

---

## Checkpoints

Periodically:

* Dirty pages flushed
* WAL truncated

Heavy write systems must tune:

* `checkpoint_timeout`
* `max_wal_size`

---

## Replication

WAL used for:

* Streaming replication
* Point-in-time recovery

---

## Interview Trap

* Why is WAL write-heavy system IO-bound?
* Why large transactions increase WAL size dramatically?

---

# 5 — Query Planner Internals

Planner chooses execution plan based on:

* Cost estimation
* Statistics
* Available indexes
* Join order
* Parallelization

---

## Cost Model

Costs measured in arbitrary units:

* seq_page_cost
* random_page_cost
* cpu_tuple_cost

Planner chooses lowest estimated cost plan.

---

## Join Strategies

1. Nested Loop
2. Hash Join
3. Merge Join

Understanding when each chosen is interview-critical.

---

## Example

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 10;
```

If high selectivity → index scan.
If low selectivity → seq scan.

Seq scan is not always bad.

---

## Interview Trap

* Forcing index is bad practice.
* Planner sometimes correct choosing seq scan.

---

# 6 — Statistics & Cardinality

Planner relies on:

* Histograms
* Most Common Values (MCV)
* Correlation statistics

If distribution skewed:

* Planner misestimates
* Wrong join order chosen

---

## Update statistics

```sql
ANALYZE table_name;
```

---

## Increase statistics target

```sql
ALTER TABLE table_name ALTER COLUMN col SET STATISTICS 1000;
```

Improves estimation for skewed data.

---

## Interview Trap

* Why does query slow down after large data insert?
  → Stats outdated.

---

# 7 — Partitioning (Declarative)

## Why partition?

* Large tables (100M+ rows)
* Time-series
* Archive separation
* Faster pruning

---

## Example — Range Partition

```sql
CREATE TABLE orders (
  id BIGINT,
  created_at DATE
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2024 PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

---

## Partition Pruning

Planner automatically scans only relevant partitions.

---

## Interview Trap

* Partitioning improves maintenance and pruning.
* It does NOT automatically speed up all queries.
* Requires proper index per partition.

---

# 8 — Advanced Indexing Deep Dive

## GIN Index

Best for:

* JSONB
* Full-text search
* Array containment

Example:

```sql
CREATE INDEX idx_tags ON posts USING GIN (tags_jsonb);
```

---

## GiST Index

Used for:

* Geospatial (PostGIS)
* Range types
* Nearest neighbor search

---

## BRIN Index

* Tiny
* Good for append-only large tables
* Time-series

Example:

```sql
CREATE INDEX idx_brin_created ON logs USING BRIN (created_at);
```

---

## Index Tradeoffs

| Type   | Best For       | Size   | Write Cost |
| ------ | -------------- | ------ | ---------- |
| B-tree | equality/range | medium | medium     |
| GIN    | jsonb/text     | large  | high       |
| GiST   | spatial        | medium | medium     |
| BRIN   | huge time data | tiny   | low        |

---

# 9 — Full-Text Search (Brief)

Postgres has built-in FTS.

```sql
SELECT to_tsvector('english', content) @@ plainto_tsquery('english', 'database');
```

Requires GIN index on tsvector.

---

# 10 — JSONB Performance Strategy

* Use `jsonb`, not `json`
* Create GIN index
* Avoid deeply nested repeated extraction
* Use containment operator `@>` when possible

Bad:

```sql
WHERE jsonb_extract_path_text(data,'a','b') = 'x'
```

Better:

```sql
WHERE data @> '{"a":{"b":"x"}}'
```

---

# 11 — Production Pitfalls

* Long idle transactions → bloat
* Too many indexes → slow writes
* Autovacuum disabled → disaster
* Large unbounded CTEs → memory pressure
* Unindexed foreign keys → slow deletes

---

# 12 — Interview Kill-Questions

If interviewer asks:

**Q1:** Why is PostgreSQL considered stronger than MySQL for analytics?
Answer:

* MVCC implementation
* Better window functions
* Rich indexing types (GIN/GiST/BRIN)
* Advanced CTE and planner
* JSONB indexing

**Q2:** Why does UPDATE increase table size?
→ MVCC row versioning.

**Q3:** Why is SERIALIZABLE expensive?
→ SSI conflict detection and retries.

**Q4:** Why might planner choose seq scan even if index exists?
→ Low selectivity or cost model suggests cheaper.

---

# You Now Have:

Phase 1 → Core SQL + Constraints
Phase 2 → Advanced Querying + Window + Index Basics
Phase 3 → Internals + Planner + Partitioning + Advanced Indexing

---
