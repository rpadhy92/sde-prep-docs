# 06 — Relational Databases & SQL

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **5 h** · 2-month plan **8 h** (see [README](README.md)).

**Learn in this order (use a local Postgres/MySQL with seeded data)**
1. **Joins, `GROUP BY`/`HAVING`, subqueries, `NOT IN` + NULL trap** (§1) — 1 h
2. **Window functions + the practice set** (§2): 2nd-highest salary, duplicates, top-N per group — 1.5 h
3. **Indexing:** B-Tree, composite/leftmost-prefix, covering index, "why isn't my index used", `EXPLAIN` (§3) — 1.5 h
4. **Transactions:** ACID, isolation levels & anomalies, locking, lost-update prevention (§4) — 1 h

**Done when:** you solve all 🟢/🟡 queries in §2 alone, justify an index with leftmost-prefix reasoning, and explain how to prevent double booking.
**Skip if short on time:** recursive CTEs, pivoting, sharding/partitioning detail, MVCC internals, multi-tenancy.

> NoSQL, Redis, Cassandra → [07_NOSQL_CASSANDRA.md](07_NOSQL_CASSANDRA.md) · JPA/Hibernate → [04_SPRING_BOOT.md](04_SPRING_BOOT.md)
> Legend: 🟢 basic · 🟡 intermediate · 🔴 senior-depth

## How to Prepare (Method)
1. **Set up a local Postgres/MySQL** (Docker). Create sample `employees`, `departments`, `orders`, `customers` tables with ~100k generated rows — you need volume to see indexes matter.
2. **Daily:** 2 SQL problems (§2) written by hand, then run & `EXPLAIN`.
3. **Index lab:** run a slow query, read `EXPLAIN ANALYZE`, add an index, compare (§3).
4. **Transaction lab:** two `psql` sessions to reproduce dirty/non-repeatable/phantom reads and a deadlock (§4).
5. **Speak answers** from the question bank (§6).
Time: 5 h (1-month) / 8 h (2-month) — the extra 3 h go to more SQL practice and the transaction/deadlock lab.

---

## 1. SQL Fundamentals
- [ ] DDL/DML/DCL/TCL; constraints (PK, FK, UNIQUE, CHECK, NOT NULL, DEFAULT)
- [ ] Joins: INNER, LEFT, RIGHT, FULL, CROSS, SELF; `UNION` vs `UNION ALL`; `INTERSECT/EXCEPT`
- [ ] `GROUP BY`, `HAVING` vs `WHERE`; aggregates; `DISTINCT`; logical processing order (FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT)
- [ ] Subqueries (correlated vs uncorrelated); `EXISTS` vs `IN` vs `JOIN`; **`NOT IN` + NULL trap**
- [ ] CTEs (`WITH`), recursive CTEs
- [ ] **Window functions:** `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `LAG/LEAD`, `SUM() OVER (PARTITION BY … ORDER BY …)`
- [ ] `CASE`, `COALESCE`, `NULLIF`; NULL semantics (three-valued logic)
- [ ] `DELETE` vs `TRUNCATE` vs `DROP`; `CHAR` vs `VARCHAR`; `TIMESTAMP` vs `TIMESTAMPTZ`; `DECIMAL` for money (never float)
- [ ] Views, materialized views, stored procedures/functions, triggers (pros/cons)

## 2. SQL Practice (write each by hand)
**Sample schema:** `employees(id, name, salary, dept_id, manager_id)`, `departments(id, name)`, `orders(id, customer_id, amount, created_at)`, `customers(id, name)`.

- 🟢 Second-highest salary — 3 ways (subquery `MAX < MAX`, `LIMIT 1 OFFSET 1`, `DENSE_RANK`)
- 🟢 Employees earning more than their manager (self-join)
- 🟢 Find duplicate emails; **delete duplicates keeping the lowest id**
- 🟢 Departments with more than 5 employees (`GROUP BY … HAVING`)
- 🟡 Nth highest salary per department (`DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC)`)
- 🟡 Customers who never ordered (`LEFT JOIN … IS NULL` / `NOT EXISTS`)
- 🟡 Top 3 products per category (window + filter)
- 🟡 Running total & month-over-month growth (`SUM() OVER`, `LAG`)
- 🟡 Consecutive-day logins (gaps-and-islands: `date - ROW_NUMBER()` trick)
- 🟡 Customers with orders in each of last 3 months
- 🔴 Employee hierarchy / org-chart depth (recursive CTE)
- 🔴 Pivot rows → columns (`CASE` + aggregate)
- 🔴 Median salary (window / percentile)
- 🔴 Rewrite a correlated subquery as a join + show plan difference

```sql
-- Nth highest per department
SELECT * FROM (
  SELECT e.*, DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
  FROM employees e
) t WHERE rnk = 2;

-- Delete duplicates, keep lowest id
DELETE FROM person p
USING person q
WHERE p.email = q.email AND p.id > q.id;
```

## 3. Design, Normalization & Indexing
- [ ] **Normalization:** 1NF (atomic), 2NF (no partial deps), 3NF (no transitive deps), BCNF; when & why **denormalize** (read-heavy, reporting)
- [ ] ER modelling; 1–N, N–N via junction table; surrogate vs natural keys; UUID vs bigint keys (index locality, ordered UUIDv7)
- [ ] **Indexes:**
  - [ ] B-Tree (default; range + equality) vs Hash vs GIN/GiST (Postgres full-text/JSON) vs bitmap
  - [ ] Clustered (InnoDB PK defines row order) vs non-clustered/secondary (stores PK → extra lookup)
  - [ ] **Composite index & leftmost-prefix rule:** `(a,b,c)` serves `a`, `a,b`, `a,b,c` — **not** `b` alone; order columns by equality first, then range, selectivity
  - [ ] **Covering index** (index-only scan); partial/filtered index; unique index
  - [ ] Index cost: write amplification, storage, bloat; don't index low-cardinality columns blindly
  - [ ] **Index not used:** function/cast on column, leading wildcard `LIKE '%x'`, implicit type conversion, `OR`, low selectivity, stale stats
- [ ] **Reading plans:** `EXPLAIN` / `EXPLAIN ANALYZE` — Seq Scan vs Index Scan vs Index Only Scan, join algorithms (nested loop / hash / merge), row estimates vs actual
- [ ] **Query tuning checklist:** select only needed columns; filter early; avoid N+1 from ORM; batch; avoid `OFFSET` deep paging → **keyset pagination**; check cardinality estimates; `ANALYZE`
- [ ] **Scaling:** read replicas (+ replication lag), partitioning (range/list/hash), sharding (shard key, hot shards, cross-shard joins), connection pooling (PgBouncer/Hikari), caching
- [ ] **Schema migration without downtime:** expand → migrate → contract; online index creation (`CREATE INDEX CONCURRENTLY`); backfill in batches

## 4. Transactions & Concurrency
- [ ] **ACID** in detail; BASE
- [ ] **Isolation levels & anomalies**

| Level | Dirty read | Non-repeatable read | Phantom | Notes |
|---|---|---|---|---|
| Read Uncommitted | ✔ | ✔ | ✔ | rarely used |
| Read Committed | ✘ | ✔ | ✔ | PostgreSQL / Oracle default |
| Repeatable Read | ✘ | ✘ | ✔ (✘ in InnoDB/PG via MVCC/gap locks) | MySQL InnoDB default |
| Serializable | ✘ | ✘ | ✘ | highest cost; retries needed |

- [ ] **MVCC:** readers don't block writers; snapshots; vacuum/purge
- [ ] **Locking:** shared/exclusive, row vs table locks, gap locks (InnoDB), `SELECT … FOR UPDATE`, `SKIP LOCKED` (job queues), advisory locks
- [ ] **Deadlocks:** cause (opposite lock order), detection, mitigation (consistent ordering, short transactions, retries)
- [ ] **Lost update** prevention: optimistic (version column / `UPDATE … WHERE version=?`), pessimistic (`FOR UPDATE`), atomic update (`SET qty = qty - 1 WHERE qty > 0`), unique constraints
- [ ] **Idempotency:** unique key on request-id; upsert (`INSERT … ON CONFLICT`)
- [ ] Long transactions harm: lock holding, vacuum bloat, pool exhaustion

## 5. DB + Application Integration
- [ ] Connection pool sizing (HikariCP: `pool ≈ (core_count × 2) + spindles` as a starting point; measure)
- [ ] ORM pitfalls: N+1, lazy loading outside tx, huge `IN` lists, `SELECT *`, missing batch inserts
- [ ] Transactions at service layer; keep them short; no remote calls inside
- [ ] Soft delete trade-offs; audit columns; optimistic version column
- [ ] Money: `NUMERIC(19,4)`/integer minor units; time: store UTC
- [ ] Multi-tenancy models (schema/db/row-level)
- [ ] Backup/restore, PITR, RPO/RTO basics

## 6. Question Bank
1. 🟢 PK vs unique key vs FK; `WHERE` vs `HAVING`; `DELETE` vs `TRUNCATE`; `UNION` vs `UNION ALL`?
2. 🟢 INNER vs LEFT JOIN; find rows in A not in B?
3. 🟡 Clustered vs non-clustered index; what is a covering index?
4. 🟡 Composite index `(a,b,c)` — will `WHERE b=?` use it? `WHERE a=? AND c=?`?
5. 🟡 Why is a query slow even with an index? (function on column, implicit cast, low selectivity, stale stats, leading wildcard)
6. 🟡 How do you find & fix a slow query in production? (slow log/`pg_stat_statements` → `EXPLAIN ANALYZE` → index/rewrite → verify)
7. 🟡 Explain isolation levels with dirty/non-repeatable/phantom examples; DB default?
8. 🟡 Pagination for a 100M-row table (keyset vs offset)?
9. 🟡 `NOT IN` vs `NOT EXISTS` with NULLs?
10. 🟡 Normalization vs denormalization — when each?
11. 🔴 Two users book the last seat simultaneously — how to prevent double booking? (unique constraint, `SELECT FOR UPDATE`, optimistic version, atomic update, queue)
12. 🔴 Zero-downtime schema migration (expand/contract, online DDL, backfill)?
13. 🔴 Explain DB deadlock; how to detect/prevent; how an app should handle it (retry)?
14. 🔴 Sharding strategies, shard-key selection, resharding, cross-shard queries/transactions?
15. 🔴 Read replicas & replication lag — how do you handle read-your-writes?
16. 🔴 When would you choose NoSQL over RDBMS? How do you handle joins/transactions there? (→ file 07)
17. 🔴 How does MVCC work? Why does Postgres need VACUUM?
18. 🔴 Design a schema for an e-commerce order system (tables, keys, indexes, constraints, money handling, audit).
19. 🔴 Your DB CPU is 95% after a release — what do you check first? (new queries, missing index, N+1, plan regression, lock waits, connection storm)

## 7. Self-Check
- [ ] Solve all 🟢/🟡 SQL problems in §2 without help; 3 of the 🔴
- [ ] Read an `EXPLAIN ANALYZE` and justify an index choice (with leftmost-prefix reasoning)
- [ ] Reproduce at least 2 anomalies/deadlock with two sessions
- [ ] Explain double-booking prevention at DB and application layers
