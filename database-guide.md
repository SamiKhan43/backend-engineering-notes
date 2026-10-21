# Databases Explained: SQL, Design, Indexes, Transactions, NoSQL, Scaling

A beginner-to-engineer reference for everything about databases in backend development: what they are, relational vs NoSQL, SQL from basics to joins and window functions, schema design and normalization, indexes and query tuning, transactions and isolation, using databases from code (pooling, ORMs, migrations), replication and sharding, backups, security, and operations. Examples use **PostgreSQL** (the best default choice) and note differences where they matter.

---

## Quick summary

- A **database** stores data durably and lets you query and change it safely, even with many users at once.
- **Start with a relational database (PostgreSQL)**. Reach for NoSQL only when you have a clear reason.
- **Design tables carefully**: primary and foreign keys, constraints, and normalization. Bad schemas are expensive to fix later.
- **Indexes** make reads fast but cost writes and space. Use `EXPLAIN` to check.
- **Transactions (ACID)** make multi-step changes all-or-nothing. Know the **isolation levels** and **race conditions**.
- From code: use **parameterized queries**, **connection pooling**, **migrations**, and avoid the **N+1** problem.
- Scale in order: **fix queries and indexes → cache → read replicas → partitioning/sharding**.
- Always have **tested backups**, least-privilege access, and monitoring.

---

## Table of contents

1. [What is a database](#1-what-is-a-database)
2. [Types of databases](#2-types-of-databases)
3. [Relational model basics](#3-relational-model-basics)
4. [SQL fundamentals](#4-sql-fundamentals)
5. [Joins, subqueries, CTEs and window functions](#5-joins-subqueries-ctes-and-window-functions)
6. [Schema design and normalization](#6-schema-design-and-normalization)
7. [Indexes](#7-indexes)
8. [Query optimization](#8-query-optimization)
9. [Transactions and ACID](#9-transactions-and-acid)
10. [Concurrency and isolation levels](#10-concurrency-and-isolation-levels)
11. [How databases work inside](#11-how-databases-work-inside)
12. [Using databases from code](#12-using-databases-from-code)
13. [Migrations](#13-migrations)
14. [NoSQL databases](#14-nosql-databases)
15. [Replication, partitioning and sharding](#15-replication-partitioning-and-sharding)
16. [Backups and high availability](#16-backups-and-high-availability)
17. [Database security](#17-database-security)
18. [Monitoring and operations](#18-monitoring-and-operations)
19. [Common design patterns](#19-common-design-patterns)
20. [Choosing a database](#20-choosing-a-database)
21. [Learning roadmap](#21-learning-roadmap)
22. [Checklist: things to remember](#22-checklist-things-to-remember)
23. [Common pitfalls and interview questions](#23-common-pitfalls-and-interview-questions)

Related guides: [Introduction to Backend Development](backend-development-intro.md), [APIs](api-guide.md), [Authentication and Authorization](authentication-guide.md), [HTTP and HTTPS](http-https-guide.md).

---

## 1. What is a database

A **database** is an organized collection of data. A **DBMS** (Database Management System, such as PostgreSQL or MongoDB) is the software that stores it and lets you query it.

**Why not just use files?**

| Problem with plain files | What a database gives you |
|---|---|
| Finding data means reading everything | **Indexes** and a query language (fast lookup) |
| Two users writing at once corrupt data | **Concurrency control** and **transactions** |
| A crash mid-write leaves a half-written file | **Durability** and crash recovery |
| No rules about what data is valid | **Schemas, constraints, types** |
| Hard to relate data (users ↔ orders) | **Relationships** and **joins** |
| No access control | **Roles and permissions** |
| Backups and replication are manual | Built-in tooling |

**Where it fits**

```mermaid
flowchart LR
  A["Client"] --> B["API / backend app"]
  B --> C["Cache (Redis)"]
  B --> D[("Primary database")]
  D --> E[("Read replica")]
  B --> F["Object storage for files"]
```

**Key vocabulary**

- **Schema**: the structure (tables, columns, types, relationships).
- **Query**: a request to read or change data.
- **Transaction**: a group of operations that succeed or fail together.
- **Index**: a structure that speeds up lookups.
- **Persistence**: data survives restarts and crashes.
- **OLTP** (online transaction processing): many small reads and writes (your app's database). **OLAP** (analytics): few huge read queries over lots of data (data warehouses like BigQuery, Snowflake, ClickHouse).

---

## 2. Types of databases

| Type | Data model | Examples | Typical use |
|---|---|---|---|
| **Relational (SQL)** | Tables, rows, relationships | **PostgreSQL**, MySQL/MariaDB, SQLite, SQL Server, Oracle | Almost any business app, the default |
| **Document** | JSON-like documents | MongoDB, CouchDB, Firestore | Flexible, nested data; content, catalogs |
| **Key-value** | Key → value | **Redis**, DynamoDB, Memcached | Caching, sessions, counters, rate limiting |
| **Wide-column** | Rows with flexible columns by partition | Cassandra, ScyllaDB, HBase | Massive write-heavy, distributed workloads |
| **Graph** | Nodes and relationships | Neo4j, Amazon Neptune | Social networks, recommendations, fraud |
| **Time-series** | Timestamped measurements | TimescaleDB, InfluxDB, Prometheus | Metrics, IoT, monitoring |
| **Search engine** | Inverted index over documents | Elasticsearch, OpenSearch, Meilisearch | Full-text search, log analysis |
| **Vector** | Embeddings for similarity search | pgvector, Pinecone, Qdrant, Weaviate | AI search, recommendations |
| **Embedded** | Library inside your app | SQLite, DuckDB, RocksDB | Mobile apps, local tools, tests |
| **Data warehouse** | Columnar, analytical | BigQuery, Snowflake, ClickHouse, Redshift | Reporting and analytics |

**SQL vs NoSQL in one table**

| | SQL (relational) | NoSQL |
|---|---|---|
| Schema | Fixed, enforced | Flexible, often enforced by the app |
| Relationships | Joins, foreign keys | Usually embedded or handled in code |
| Transactions | Strong (ACID), multi-table | Varies, often limited to one record or partition |
| Scaling | Vertical first, replicas, sharding is harder | Often designed to scale out horizontally |
| Query power | Very high (SQL) | Depends on the system |

**Polyglot persistence**: real systems often combine several (PostgreSQL as the source of truth + Redis for caching + Elasticsearch for search + S3 for files). Add each one only for a measured need.

---

## 3. Relational model basics

Data lives in **tables** made of **rows** (records) and **columns** (attributes). Tables are linked through **keys**.

**Keys**

- **Primary key (PK)**: uniquely identifies a row. Never null, never changes.
- **Foreign key (FK)**: a column pointing at another table's primary key. The database enforces that the target exists.
- **Unique key**: no duplicates allowed (e.g. `email`).
- **Composite key**: a key made from several columns.
- **Surrogate key**: an artificial ID (`id`, UUID). **Natural key**: a real-world value (email, ISBN). Surrogate keys are the usual choice for primary keys, with natural keys protected by `UNIQUE`.

**Relationships**

| Type | Example | How to model |
|---|---|---|
| **One-to-one** | User ↔ profile | FK with a `UNIQUE` constraint |
| **One-to-many** | User → many orders | FK on the "many" side (`orders.user_id`) |
| **Many-to-many** | Students ↔ courses | A **junction table** (`enrollments(student_id, course_id)`) |

**Constraints** (the database defends your data even when the app has bugs)

- `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY ... ON DELETE CASCADE | RESTRICT | SET NULL`, `CHECK (price >= 0)`, `DEFAULT`.

**Data types (PostgreSQL)**

| Need | Use |
|---|---|
| Integer IDs | `INTEGER` / `BIGINT` with `GENERATED ALWAYS AS IDENTITY` (or `UUID`) |
| Text | `TEXT` (or `VARCHAR(n)` for a hard limit) |
| Money | `NUMERIC(12,2)` or integer cents. **Never `FLOAT`.** |
| True/false | `BOOLEAN` |
| Dates and times | `DATE`, **`TIMESTAMPTZ`** (store in UTC) |
| Flexible JSON | `JSONB` (indexed, queryable) |
| Unique IDs | `UUID` |
| Fixed set of values | `TEXT` + `CHECK`, or an `ENUM` type, or a lookup table |

```sql
CREATE TABLE users (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email      TEXT NOT NULL UNIQUE,
  name       TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE orders (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id    BIGINT NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  total      NUMERIC(12,2) NOT NULL CHECK (total >= 0),
  status     TEXT NOT NULL DEFAULT 'pending'
             CHECK (status IN ('pending', 'paid', 'shipped', 'cancelled')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## 4. SQL fundamentals

**SQL** (Structured Query Language) has several parts:

| Part | Purpose | Statements |
|---|---|---|
| **DDL** (Definition) | Define structure | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** (Manipulation) | Change data | `INSERT`, `UPDATE`, `DELETE` |
| **DQL** (Query) | Read data | `SELECT` |
| **DCL** (Control) | Permissions | `GRANT`, `REVOKE` |
| **TCL** (Transaction) | Transactions | `BEGIN`, `COMMIT`, `ROLLBACK` |

**Reading data**

```sql
SELECT id, email                      -- choose columns (avoid SELECT * in apps)
FROM users
WHERE created_at >= '2026-01-01'      -- filter rows
  AND email LIKE '%@example.com'
ORDER BY created_at DESC              -- sort
LIMIT 20 OFFSET 0;                    -- paginate
```

**Aggregation**

```sql
SELECT user_id,
       COUNT(*)      AS order_count,
       SUM(total)    AS revenue,
       AVG(total)    AS avg_order
FROM orders
WHERE status = 'paid'
GROUP BY user_id
HAVING SUM(total) > 1000              -- HAVING filters groups, WHERE filters rows
ORDER BY revenue DESC;
```

**Changing data**

```sql
INSERT INTO users (email, name) VALUES ('ana@example.com', 'Ana')
RETURNING id;                                          -- PostgreSQL: get the new row back

UPDATE orders SET status = 'shipped' WHERE id = 42;    -- ALWAYS use WHERE!
DELETE FROM orders WHERE status = 'cancelled' AND created_at < now() - interval '1 year';

-- Upsert (insert or update if it exists)
INSERT INTO users (email, name) VALUES ('ana@example.com', 'Ana B.')
ON CONFLICT (email) DO UPDATE SET name = EXCLUDED.name;
```

**How SQL is evaluated (logical order)**

`FROM / JOIN` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`.
This is why you cannot use a `SELECT` alias inside `WHERE`, but you can in `ORDER BY`.

**NULL (the classic trap)**

- `NULL` means "unknown", not zero or empty.
- `NULL = NULL` is **not true**. Use `IS NULL` / `IS NOT NULL`.
- Any comparison with `NULL` yields unknown, so rows disappear from `WHERE`.
- `NOT IN (subquery)` returns nothing if the subquery contains a `NULL`. Prefer `NOT EXISTS`.
- Aggregates like `COUNT(column)` ignore `NULL`s, while `COUNT(*)` counts rows.
- Use `COALESCE(x, 0)` to supply defaults.

**Handy expressions**

```sql
SELECT name,
       CASE WHEN total >= 100 THEN 'big' ELSE 'small' END AS size,
       COALESCE(nickname, name)                           AS display_name
FROM orders;
```

**Always back up and test before running destructive statements.** Run `SELECT` with the same `WHERE` first, and use a transaction so you can `ROLLBACK`.

---

## 5. Joins, subqueries, CTEs and window functions

### Joins

A **join** combines rows from two tables based on a condition.

| Join | Returns |
|---|---|
| **`INNER JOIN`** | Only rows that match on both sides |
| **`LEFT JOIN`** | All rows from the left table, matched rows from the right (`NULL` when none) |
| **`RIGHT JOIN`** | Mirror of left join (rarely needed) |
| **`FULL OUTER JOIN`** | All rows from both, matched where possible |
| **`CROSS JOIN`** | Every combination (Cartesian product) |
| **Self join** | A table joined to itself (e.g. employee → manager) |

```sql
-- Orders with their customer
SELECT o.id, o.total, u.email
FROM orders o
JOIN users u ON u.id = o.user_id;

-- Users who have NO orders (anti-join)
SELECT u.id, u.email
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE o.id IS NULL;

-- Many-to-many through a junction table
SELECT s.name, c.title
FROM students s
JOIN enrollments e ON e.student_id = s.id
JOIN courses c     ON c.id = e.course_id;
```

Always join on **indexed** columns (foreign keys should be indexed).

### Subqueries

```sql
-- Users whose total spending is above average
SELECT user_id
FROM orders
GROUP BY user_id
HAVING SUM(total) > (SELECT AVG(total_spent)
                     FROM (SELECT SUM(total) AS total_spent FROM orders GROUP BY user_id) t);

-- EXISTS is often faster and safer than IN / NOT IN
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id AND o.status = 'paid');
```

### CTEs (`WITH`)

Name a sub-result so complex queries read top to bottom.

```sql
WITH paid AS (
  SELECT user_id, SUM(total) AS spent
  FROM orders
  WHERE status = 'paid'
  GROUP BY user_id
)
SELECT u.email, p.spent
FROM paid p
JOIN users u ON u.id = p.user_id
WHERE p.spent > 500
ORDER BY p.spent DESC;
```

### Window functions

Calculate across related rows **without collapsing them** (unlike `GROUP BY`).

```sql
-- Latest order per user
SELECT *
FROM (
  SELECT o.*,
         ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS rn
  FROM orders o
) t
WHERE rn = 1;

-- Running total and ranking
SELECT id, total,
       SUM(total) OVER (ORDER BY created_at)       AS running_total,
       RANK()     OVER (ORDER BY total DESC)       AS rank_by_total
FROM orders;
```

Useful ones: `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`, `SUM/AVG OVER`.

### Set operations and views

- `UNION` (combine, remove duplicates), `UNION ALL` (combine, keep duplicates, faster), `INTERSECT`, `EXCEPT`.
- **View**: a saved query (`CREATE VIEW paid_orders AS SELECT ...`). **Materialized view**: a saved query whose **result is stored** and refreshed on demand (`REFRESH MATERIALIZED VIEW`), good for heavy reports.

---

## 6. Schema design and normalization

Good design = **each fact stored once, in the right place**.

### Normal forms (the short version)

| Form | Rule | Fixes |
|---|---|---|
| **1NF** | Each column holds a single value, no repeating groups (no `tags = "a,b,c"`) | Lists crammed in one column |
| **2NF** | Every non-key column depends on the **whole** key | Partial dependency in composite keys |
| **3NF** | Non-key columns depend on the key, **not on other non-key columns** | `orders.customer_city` stored next to `customer_id` |
| **BCNF** | Stricter 3NF: every determinant is a candidate key | Odd edge cases |

**Why it matters**: duplicated data causes **anomalies**: update anomalies (change it in one place, forget another), insert anomalies, delete anomalies.

**Denormalization**: deliberately duplicating data for read speed (e.g. storing `order_items.unit_price` at purchase time, or a precomputed `comment_count`). It is a trade-off: faster reads, harder writes, risk of inconsistency. Normalize first, denormalize with evidence.

### Example schema (e-commerce)

```mermaid
erDiagram
  USERS ||--o{ ORDERS : places
  ORDERS ||--|{ ORDER_ITEMS : contains
  PRODUCTS ||--o{ ORDER_ITEMS : "appears in"
  USERS {
    bigint id PK
    text email
    text name
  }
  ORDERS {
    bigint id PK
    bigint user_id FK
    numeric total
    text status
  }
  PRODUCTS {
    bigint id PK
    text name
    numeric price
  }
  ORDER_ITEMS {
    bigint order_id FK
    bigint product_id FK
    int quantity
    numeric unit_price
  }
```

Note `order_items.unit_price`: prices change over time, so the price at purchase is **historical data** and must be stored on the item.

### Design tips

- Give every table a **primary key**. Index every **foreign key**.
- Add `created_at` and `updated_at` to most tables (`TIMESTAMPTZ`, UTC).
- Use clear, consistent names (`snake_case`, plural table names, `user_id` for FKs).
- Let **constraints** enforce rules (`NOT NULL`, `UNIQUE`, `CHECK`, FK) instead of trusting the app.
- **Soft delete** (`deleted_at`) helps recovery but complicates every query and unique constraint (use partial unique indexes). Hard delete is simpler. Archive old data when needed.
- **JSONB** is great for truly flexible attributes, but do not hide your core relational data in it.
- **Enums**: a `CHECK` constraint or lookup table is easier to change than a native enum.
- **IDs**: sequential `BIGINT` is compact and index-friendly. `UUID` is good for distributed generation and hard-to-guess IDs (consider time-ordered UUIDs to keep indexes efficient).
- **Multi-tenant apps**: add a `tenant_id` to every table (shared schema), or use a schema or database per tenant. Enforce isolation (Row-Level Security helps).
- **Auditing**: an `audit_log` table or triggers recording who changed what.
- Store **files in object storage**, only the metadata in the database.
- Store **time in UTC**, **money as `NUMERIC` or integer cents**.

---

## 7. Indexes

An **index** is a separate sorted structure that lets the database find rows without scanning the whole table (like the index at the back of a book).

**Without an index**: *sequential scan* (read every row). **With an index**: *index scan* (jump to the matches).

```sql
CREATE INDEX idx_orders_user_id ON orders (user_id);                 -- speeds up WHERE/JOIN on user_id
CREATE UNIQUE INDEX idx_users_email ON users (email);                -- also enforces uniqueness
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at DESC);   -- composite
CREATE INDEX idx_orders_pending ON orders (created_at) WHERE status = 'pending';   -- partial index
CREATE INDEX idx_users_lower_email ON users (lower(email));          -- expression index
```

### Index types (PostgreSQL)

| Type | Good for |
|---|---|
| **B-tree** (default) | Equality, ranges, sorting (`=`, `<`, `>`, `BETWEEN`, `ORDER BY`, `LIKE 'abc%'`) |
| **Hash** | Equality only (rarely better than B-tree) |
| **GIN** | `JSONB`, arrays, full-text search, trigram search (`pg_trgm`) |
| **GiST / SP-GiST** | Geometric data, ranges, nearest-neighbor |
| **BRIN** | Huge tables with naturally ordered data (e.g. time-series), tiny index |

### Composite indexes and the leftmost prefix rule

An index on `(user_id, created_at)` can serve:

- `WHERE user_id = 5` ✔
- `WHERE user_id = 5 AND created_at > ...` ✔ (and a matching `ORDER BY`)
- `WHERE created_at > ...` alone ✘ (does not start with the first column)

Put **equality columns first**, then range or sort columns. A **covering index** (`INCLUDE (...)`) lets the database answer a query from the index alone (index-only scan).

### Costs

- Each index **slows down `INSERT/UPDATE/DELETE`** and uses disk and memory.
- Too many indexes hurt. Index what your real queries filter, join, and sort on.
- Tiny tables or columns with very few distinct values (e.g. a boolean) often gain nothing from a plain index.

### Why an index might not be used

- Applying a **function** to the column: `WHERE lower(email) = ...` (create an expression index instead).
- **Leading wildcard**: `LIKE '%abc'` (use `pg_trgm` + GIN for contains search).
- **Type mismatch** or implicit casts.
- Query returns a **large fraction** of the table (a sequential scan is faster).
- Outdated **statistics** (run `ANALYZE`).
- `OR` conditions across different columns, or the leftmost prefix rule above.

---

## 8. Query optimization

**Always measure with `EXPLAIN`.**

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 42 ORDER BY created_at DESC LIMIT 20;
```

Illustrative output (yours will differ):

```
Limit  (cost=0.43..8.45 rows=20) (actual time=0.03..0.05 rows=20 loops=1)
  ->  Index Scan using idx_orders_user_created on orders
        Index Cond: (user_id = 42)
Planning Time: 0.1 ms
Execution Time: 0.1 ms
```

**What to look for**

- **Seq Scan** on a big table in a frequent query → missing index.
- **Index Scan / Index Only Scan** → good.
- Estimated rows vs **actual rows** far apart → stale statistics (`ANALYZE`).
- **Sort** or **Hash** steps spilling to disk → needs memory or an index that provides the order.
- `EXPLAIN ANALYZE` actually **runs** the query. Wrap `UPDATE/DELETE` in a transaction and roll back.

**Practical checklist**

1. Select only the **columns you need**. Avoid `SELECT *`.
2. **Index** columns used in `WHERE`, `JOIN`, `ORDER BY`. Index foreign keys.
3. **Paginate** and use **keyset pagination** for deep pages (`WHERE id < :last ORDER BY id DESC LIMIT 20`), since big `OFFSET`s get slow.
4. Fix **N+1** queries (see section 12).
5. **Batch** writes: multi-row `INSERT`, `COPY` for bulk loads, one transaction instead of thousands.
6. Avoid functions on indexed columns in `WHERE`.
7. Prefer `EXISTS` over `IN` for large subqueries, and `UNION ALL` over `UNION` when duplicates do not matter.
8. Keep **transactions short**.
9. Avoid counting huge tables often (`COUNT(*)` scans). Cache or approximate counts.
10. Keep statistics fresh (`ANALYZE`, autovacuum) and find slow queries using the **slow query log** and `pg_stat_statements`.

**Cache** expensive read results (Redis) once queries are already reasonable. See the backend intro.

---

## 9. Transactions and ACID

A **transaction** groups several statements into **one unit of work**: all succeed, or none do.

```sql
-- Transfer 100 from account 1 to account 2
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1 AND balance >= 100;
-- (in app code: check that exactly 1 row was updated, otherwise ROLLBACK)
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;       -- make permanent
-- ROLLBACK;  -- undo everything since BEGIN
```

Without a transaction, a crash between the two updates would destroy money.

### ACID

| Letter | Property | Meaning |
|---|---|---|
| **A** | **Atomicity** | All or nothing |
| **C** | **Consistency** | Constraints and rules hold before and after (a valid state moves to a valid state) |
| **I** | **Isolation** | Concurrent transactions do not see each other's half-done work |
| **D** | **Durability** | Once committed, data survives crashes (thanks to the write-ahead log) |

**Practical notes**

- **Savepoints** (`SAVEPOINT x; ... ROLLBACK TO x;`) allow partial rollback.
- Keep transactions **short**: long ones hold locks, block others, and bloat the database.
- Never call slow external services (HTTP, email) **inside** a transaction.
- In code, use a transaction scope (`with session.begin():`) so errors roll back automatically.
- Relying on **atomic statements** often avoids explicit locking: `UPDATE stock SET qty = qty - 1 WHERE id = 7 AND qty > 0`.
- Single statements are already atomic.
- Many NoSQL systems offer limited transactions (per document or partition), so multi-record consistency becomes your job.

---

## 10. Concurrency and isolation levels

Many transactions run at once, and without care they interfere.

### Anomalies

| Anomaly | What happens |
|---|---|
| **Dirty read** | You read another transaction's **uncommitted** change |
| **Non-repeatable read** | You read a row twice and get different values (someone committed in between) |
| **Phantom read** | You run the same query twice and new rows appear |
| **Lost update** | Two transactions read, both modify, last write wins and erases the first |
| **Write skew** | Two transactions each check a rule using what they read, then write different rows, together breaking the rule (e.g. both doctors go off-call) |

### Isolation levels (SQL standard)

| Level | Prevents | Notes |
|---|---|---|
| **Read Uncommitted** | Nothing | In PostgreSQL it behaves like Read Committed |
| **Read Committed** | Dirty reads | **PostgreSQL default.** Each statement sees committed data |
| **Repeatable Read** | + Non-repeatable reads | MySQL InnoDB default. In PostgreSQL it is snapshot isolation (also prevents phantoms) |
| **Serializable** | + Phantoms, write skew | As if transactions ran one at a time. May abort some with a serialization error, so be ready to **retry** |

Higher isolation = more safety, but more aborts or waiting.

### Tools for handling concurrency

- **MVCC** (Multi-Version Concurrency Control): databases like PostgreSQL keep multiple row versions so **readers do not block writers** and writers do not block readers.
- **Row locks**: `SELECT ... FOR UPDATE` locks the selected rows until the transaction ends (**pessimistic locking**).
- **Optimistic locking**: add a `version` column, update with `WHERE id = :id AND version = :seen`, and retry if 0 rows were updated. Great for web apps (see the ETag/If-Match idea in the API guide).
- **Unique constraints** prevent duplicates even under races (let the database be the referee).
- **Atomic updates**: `SET qty = qty - 1 WHERE qty > 0`.
- **Advisory locks** for application-level mutual exclusion.
- **`FOR UPDATE SKIP LOCKED`** lets multiple workers pull different jobs from a table (simple job queue).

```sql
-- Pessimistic: lock the row, then change it
BEGIN;
SELECT qty FROM stock WHERE id = 7 FOR UPDATE;
UPDATE stock SET qty = qty - 1 WHERE id = 7;
COMMIT;
```

### Deadlocks

Transaction A holds lock 1 and waits for lock 2, while B holds lock 2 and waits for lock 1. The database detects it and **aborts one** of them. Reduce deadlocks by touching rows in a **consistent order**, keeping transactions short, and **retrying** on deadlock errors.

---

## 11. How databases work inside

A little knowledge here explains many "why" questions.

- **Pages / blocks**: data is stored in fixed-size pages (e.g. 8 KB) on disk, and cached in memory (the **buffer cache**). Memory hits are much faster than disk reads, so "hot data fits in RAM" is a big win.
- **B-tree indexes**: balanced trees that keep keys sorted, giving fast lookups, range scans, and ordering in O(log n).
- **WAL (Write-Ahead Log)**: before changing data pages, the change is appended to a sequential log and flushed to disk. After a crash, the database **replays the log** to recover (this provides durability). The WAL also powers **replication** and **point-in-time recovery**.
- **MVCC**: updates create a new row version, and old versions are cleaned up later (PostgreSQL's **VACUUM**). Without cleanup, tables **bloat**.
- **Query planner / optimizer**: turns SQL into an execution plan, choosing between scans, index use, and join algorithms (**nested loop**, **hash join**, **merge join**) using **statistics**.
- **LSM trees** (used by Cassandra, RocksDB, many NoSQL stores): writes go to memory and are flushed as sorted files, then merged (**compaction**). They favor write-heavy workloads, whereas B-trees favor balanced reads and writes.
- **Row-oriented vs column-oriented storage**: rows store each record together (good for OLTP). Columnar stores keep each column together (great for analytics: scan only the needed columns, compress well).

---

## 12. Using databases from code

### Connections and pooling

- Opening a connection is expensive and databases limit how many can exist (PostgreSQL defaults to about 100).
- Use a **connection pool**: the app keeps a set of open connections and lends them out per request.
- Size pools sensibly: `pool_size × app_instances` must stay **below** the database's connection limit. Use **PgBouncer** (or a managed proxy) when many app instances need many connections.
- Always **release** connections (context managers / `with` blocks).
- Set **timeouts** (connect, statement, idle-in-transaction) so a stuck query cannot block everything.

### Parameterized queries (prevents SQL injection)

```python
# ❌ NEVER build SQL with string formatting
query = f"SELECT * FROM users WHERE email = '{email}'"        # injectable!

# ✅ Pass values separately, the driver sends them as data, not SQL
cursor.execute("SELECT * FROM users WHERE email = %s", (email,))
```

An input like `' OR '1'='1` becomes harmless text when parameterized.

### ORMs (SQLAlchemy 2.0 example)

An **ORM** maps tables to classes. It saves boilerplate, but you still need to understand the SQL it generates.

```python
# pip install sqlalchemy "psycopg[binary]"
import os

from sqlalchemy import ForeignKey, String, create_engine, select, text
from sqlalchemy.orm import (DeclarativeBase, Mapped, Session, mapped_column,
                            relationship, selectinload)


class Base(DeclarativeBase):
    pass


class User(Base):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True)
    tasks: Mapped[list["Task"]] = relationship(back_populates="user")


class Task(Base):
    __tablename__ = "tasks"
    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), index=True)
    title: Mapped[str] = mapped_column(String(100))
    done: Mapped[bool] = mapped_column(default=False)
    user: Mapped[User] = relationship(back_populates="tasks")


engine = create_engine(
    os.environ["DATABASE_URL"],      # e.g. postgresql+psycopg://user:pass@localhost/mydb
    pool_size=10, max_overflow=5,    # connection pool
    pool_pre_ping=True,              # drop dead connections
)

# A transaction: commits on success, rolls back on any exception
with Session(engine) as session, session.begin():
    user = User(email="ana@example.com")
    user.tasks.append(Task(title="Learn databases"))
    session.add(user)

# Reading, with eager loading to avoid N+1
with Session(engine) as session:
    users = session.scalars(select(User).options(selectinload(User.tasks))).all()
    for u in users:
        print(u.email, [t.title for t in u.tasks])      # no extra query per user

    # Raw SQL is fine too, always with bound parameters
    rows = session.execute(text("SELECT id FROM tasks WHERE user_id = :uid"), {"uid": 1}).all()
```

(The password in `DATABASE_URL` comes from the environment, never from source code.)

### The N+1 problem

```
1 query:   SELECT * FROM users;                         -- 100 users
100 more:  SELECT * FROM tasks WHERE user_id = ?;       -- one per user
```

101 round trips for what could be 1 or 2. **Fix**: join, or eager-load (`selectinload` / `joinedload`), or fetch all related rows with `WHERE user_id IN (...)`. Spot it by logging queries per request.

### Other practical points

- Keep SQL and data access in a **repository / data layer**, separate from HTTP handlers.
- Do not put business logic in triggers unless you have a strong reason, since it becomes invisible.
- Handle **unique-violation**, **serialization failure**, and **deadlock** errors explicitly (map to `409`, or retry).
- Use **bulk operations** for big inserts (`COPY`, `executemany`, multi-row `INSERT`).
- Use async drivers (asyncpg, SQLAlchemy async) only if your whole stack is async.
- In tests, use a real PostgreSQL (e.g. in Docker), not a different engine, so behavior matches production.

---

## 13. Migrations

A **migration** is a versioned script that changes the schema. The set of migrations in Git is the **history of your database**. Never edit production tables by hand.

Tools: **Alembic** (SQLAlchemy), **Django migrations**, **Flyway**, **Liquibase**, **Prisma Migrate**, **Atlas**.

```bash
alembic revision --autogenerate -m "add tasks table"   # create a migration (review it!)
alembic upgrade head                                    # apply
alembic downgrade -1                                    # roll back one step
```

**Rules**

- **Review autogenerated migrations**. They are a draft.
- One logical change per migration, applied automatically in CI/CD.
- Test migrations on a **copy of production-sized data**: some operations lock tables for a long time.
- Plan for **zero downtime** with the **expand and contract** pattern:
  1. **Expand**: add the new column/table (nullable or with a default), deploy code that writes to both.
  2. **Backfill** old rows in small batches.
  3. **Switch** reads to the new column.
  4. **Contract**: remove the old column in a later release.
- Avoid breaking changes in a single step (renaming or dropping a column while old code is still running).
- In PostgreSQL, create big indexes with `CREATE INDEX CONCURRENTLY` to avoid blocking writes.
- Keep seed/reference data in migrations or controlled scripts.
- Have a **rollback plan**, and **back up before risky migrations**.

---

## 14. NoSQL databases

### Document databases (MongoDB)

Store JSON-like (BSON) documents in collections. No rigid schema, nested data is natural.

```javascript
// Insert
db.users.insertOne({
  name: "Ana",
  email: "ana@example.com",
  addresses: [{ city: "Karachi", primary: true }],
  tags: ["admin", "beta"]
});

// Query
db.users.find({ "addresses.city": "Karachi", tags: "admin" });

// Update one field
db.users.updateOne({ email: "ana@example.com" }, { $set: { name: "Ana B." } });

// Aggregation pipeline
db.orders.aggregate([
  { $match: { status: "paid" } },
  { $group: { _id: "$user_id", revenue: { $sum: "$total" } } },
  { $sort: { revenue: -1 } }
]);
```

- **Model for how you query**: embed data that is read together, reference data that is large, shared, or changes independently.
- Beware unbounded arrays and huge documents (16 MB limit).
- Joins are limited (`$lookup`), and multi-document transactions exist but cost more.
- Great for flexible, hierarchical content. Not a free pass to skip data modeling.

### Key-value stores (Redis)

An in-memory data store with rich data structures, extremely fast (microseconds).

```text
SET session:abc "user:42" EX 3600       # string with a 1-hour TTL
GET session:abc
INCR page:views                          # atomic counter
HSET user:42 name "Ana" plan "pro"       # hash
LPUSH jobs "send-email:7"                # list (queue)
SADD online_users 42                     # set
ZADD leaderboard 1500 "ana"              # sorted set
ZREVRANGE leaderboard 0 9 WITHSCORES     # top 10
```

- **Uses**: caching, sessions, rate limiting, leaderboards, pub/sub, job queues, distributed locks.
- **Persistence** options: RDB snapshots, AOF log. Redis is mainly a cache or fast secondary store, not usually your only source of truth.
- Set **TTLs** and an **eviction policy** (`maxmemory-policy`) so memory does not fill up.
- Commands are executed one at a time, so single operations like `INCR` are atomic.

### Wide-column stores (Cassandra, ScyllaDB)

- Data is partitioned across many nodes by a **partition key**, and tables are designed **around specific queries** (no joins, limited ad-hoc queries).
- Excellent for massive, write-heavy, always-on workloads (messaging, IoT, activity feeds).
- Typically **eventually consistent** with tunable consistency.

### Graph databases (Neo4j)

Model **nodes** and **relationships**. Great when the connections are the point.

```cypher
// Friends of friends of Ana who are not already her friends
MATCH (a:Person {name: "Ana"})-[:FRIEND]->(:Person)-[:FRIEND]->(fof:Person)
WHERE NOT (a)-[:FRIEND]->(fof) AND fof <> a
RETURN DISTINCT fof.name;
```

Use for social networks, recommendations, fraud detection, knowledge graphs.

### Other specialized stores

- **Time-series** (TimescaleDB, InfluxDB): high-volume timestamped data, downsampling, retention policies.
- **Search** (Elasticsearch, OpenSearch, Meilisearch): full-text search, relevance ranking, faceting. Usually a **secondary copy** fed from your main database.
- **Vector** (pgvector, Pinecone, Qdrant): similarity search over embeddings for AI features. `pgvector` lets PostgreSQL do it.
- **Analytics / warehouse** (ClickHouse, BigQuery, Snowflake): columnar storage for reports over billions of rows.

**When NoSQL is a good idea**: you have a clear access pattern it handles better, extreme scale or write volume, flexible or evolving data, or a specialized need (search, vectors, graphs, caching). Otherwise PostgreSQL (with `JSONB`, full-text search, `pgvector`) covers a great deal.

---

## 15. Replication, partitioning and sharding

### Scaling order

1. **Optimize** queries and **indexes** (cheapest, biggest win).
2. Add a **cache** (Redis).
3. **Scale up** (more CPU/RAM/faster disks).
4. Add **read replicas**.
5. **Partition** big tables.
6. **Shard** across databases (last resort, high complexity).

### Replication (copies of the same data)

```mermaid
flowchart LR
  A["App writes"] --> B[("Primary")]
  B -->|"WAL stream"| C[("Replica 1")]
  B -->|"WAL stream"| D[("Replica 2")]
  E["App reads"] --> C
  E --> D
```

- **Primary (leader)** takes writes. **Replicas (followers)** copy changes and serve reads.
- **Asynchronous** replication: fast, but replicas lag (you might read old data, and an unreplicated write can be lost on failover). **Synchronous** replication waits for a replica (safer, slower).
- **Replication lag** causes "I saved it but cannot see it" bugs. Fix: read your own recent writes from the primary (**read-your-writes**), or route that user's reads to the primary for a short time.
- **Failover**: promote a replica when the primary dies (automatic with tools such as Patroni, or managed services). Test it.
- **Multi-leader** and **leaderless** setups exist (Cassandra, DynamoDB) but need conflict resolution.

### Partitioning (splitting a table inside one database)

Split a huge table into smaller pieces for faster queries and easier maintenance:

- **Range** (by date: one partition per month), **list** (by region), **hash** (by ID).
- Queries that filter by the partition key skip irrelevant partitions (**partition pruning**). Old partitions can be dropped instantly.

```sql
CREATE TABLE events (
  id         BIGINT GENERATED ALWAYS AS IDENTITY,
  created_at TIMESTAMPTZ NOT NULL,
  payload    JSONB
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2026_10 PARTITION OF events
  FOR VALUES FROM ('2026-10-01') TO ('2026-11-01');
```

### Sharding (splitting data across multiple databases or servers)

- Pick a **shard key** (e.g. `user_id` or `tenant_id`) and route each row to a shard (by hash, range, or lookup).
- Challenges: **cross-shard queries and joins**, cross-shard transactions, **rebalancing** when adding shards (consistent hashing helps), **hot shards** (uneven load), and operational complexity.
- Choose the shard key carefully, since it is very hard to change.
- Consider managed or natively distributed systems (Citus, CockroachDB, YugabyteDB, Vitess, DynamoDB, Cassandra) instead of building it yourself.

### Distributed data theory in a nutshell

- **CAP theorem**: when a **network partition** happens, a distributed system must choose **Consistency** (reject some requests to stay correct) or **Availability** (keep answering, possibly with stale data).
- **Eventual consistency**: replicas converge over time if no new writes occur.
- **Consensus algorithms** (Raft, Paxos) let nodes agree on a leader or a value.
- **Idempotent operations** and **conflict resolution** (last-write-wins, versions, CRDTs) matter in distributed writes.
- **Outbox pattern** and **event streaming** (Kafka) keep databases and other systems in sync reliably.

---

## 16. Backups and high availability

> A backup you have never restored is only a hope.

**Backup types (PostgreSQL)**

| Type | How | Notes |
|---|---|---|
| **Logical** | `pg_dump` exports SQL/data | Portable, good for smaller databases and migrations; slow to restore at scale |
| **Physical** | Copy data files / base backup | Fast for large databases |
| **Continuous archiving + PITR** | Base backup + archived WAL | **Point-in-time recovery**: restore to "10:42 yesterday, just before the bad deploy" |
| **Snapshots** | Disk/volume snapshots (cloud) | Fast, convenient, ensure consistency |

```bash
pg_dump -Fc mydb > mydb.dump          # custom-format logical backup
pg_restore -d mydb_restored mydb.dump # restore into another database to test
```

**Key terms**

- **RPO** (Recovery Point Objective): how much data you can afford to lose (e.g. 5 minutes).
- **RTO** (Recovery Time Objective): how long you can afford to be down (e.g. 30 minutes).
- Replicas are **not backups**: a bad `DELETE` or `DROP TABLE` replicates instantly.

**Good practice**

- Automate backups, store them **off-site / in another region**, and **encrypt** them.
- Follow **3-2-1**: 3 copies, 2 different media, 1 off-site.
- **Test restores regularly** and time them.
- Enable **point-in-time recovery** for production.
- Keep **retention policies** (daily, weekly, monthly) and think about legal requirements (e.g. deletion requests).
- For **high availability**: replicas + automatic failover + health checks + multiple availability zones. Practice failover drills.
- Use **managed databases** (RDS, Cloud SQL, Azure Database, Supabase, Neon, MongoDB Atlas) when you can: backups, patching, and failover are handled for you.

---

## 17. Database security

- **Least privilege**: the app connects with a dedicated role that has only the permissions it needs (not the superuser or owner). Use separate roles for migrations, the app, and read-only analytics.

```sql
CREATE ROLE app_user LOGIN PASSWORD 'use-a-secret-manager-value';
GRANT CONNECT ON DATABASE mydb TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
```

- **Never expose the database to the public internet.** Put it in a private network or subnet, restrict access with firewalls and security groups, and reach it through the app or a VPN/bastion.
- **Encrypt in transit** (TLS connections, `sslmode=verify-full`) and **at rest** (disk or database-level encryption).
- **Parameterized queries** always (SQL injection).
- **Secrets** (database passwords) in a secret manager or environment, rotated, never in Git.
- **Hash passwords** in the app (Argon2id), never store plain text (see the Authentication guide). Encrypt other sensitive fields (tokens, national IDs) when needed.
- **Row-Level Security (RLS)** in PostgreSQL enforces per-row access in the database itself, a strong safety net for multi-tenant data:

```sql
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;
CREATE POLICY tasks_owner ON tasks
  USING (user_id = current_setting('app.current_user_id')::bigint);
```

- **Minimize and protect personal data (PII)**: store only what you need, mask it in logs and non-production copies, and plan for deletion and retention rules (GDPR and similar laws).
- **Audit logging** of sensitive actions, and monitor for unusual access.
- Keep the database **patched** and remove unused accounts and default credentials.
- **Do not use production data in development** without anonymizing it.
- Protect **backups** with the same care as the live database.

---

## 18. Monitoring and operations

**What to watch**

| Metric | Why |
|---|---|
| **Slow queries** (slow query log, `pg_stat_statements`) | Find what to index or rewrite |
| **Connections** (active, idle, idle in transaction) | Pool exhaustion, leaks |
| **Cache hit ratio** | Is hot data in memory? |
| **CPU, memory, disk I/O, disk space** | Capacity and saturation |
| **Replication lag** | Stale reads, failover risk |
| **Locks and deadlocks** | Blocked or failing transactions |
| **Transaction ID / table bloat, autovacuum** (PostgreSQL) | Maintenance health |
| **Error rates and query latency (p95/p99)** | User-facing impact |

**Routine maintenance (PostgreSQL)**

- **`VACUUM` / autovacuum** cleans dead row versions. Keep autovacuum on and healthy.
- **`ANALYZE`** updates planner statistics.
- **`REINDEX`** for bloated indexes (use `CONCURRENTLY` where possible).
- Watch for **long-running or "idle in transaction"** sessions, which block cleanup and hold locks.

**Useful queries**

```sql
-- What is running right now?
SELECT pid, state, now() - query_start AS runtime, query
FROM pg_stat_activity
WHERE state <> 'idle'
ORDER BY runtime DESC;

-- Table sizes
SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) AS size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC
LIMIT 10;
```

Tools: pgAdmin, DBeaver, `psql`, Grafana + Prometheus (postgres_exporter), your cloud provider's dashboards, `pgBadger` (log analysis). Set **alerts** on disk space, connection count, replication lag, and error rates, and plan **capacity** before you run out.

---

## 19. Common design patterns

- **Soft delete**: `deleted_at TIMESTAMPTZ NULL`. Filter `WHERE deleted_at IS NULL`, and use partial unique indexes (`UNIQUE (email) WHERE deleted_at IS NULL`).
- **Audit trail**: an append-only `audit_log` (who, what, when, old and new values).
- **Optimistic locking**: a `version` column checked in every `UPDATE`.
- **Idempotency table**: store `(idempotency_key, response)` to make retried `POST`s safe.
- **Outbox pattern**: write the business change **and** an "event to publish" row in one transaction, and a worker publishes the events. This keeps the database and message broker consistent.
- **Job queue in PostgreSQL**: `SELECT ... FOR UPDATE SKIP LOCKED` lets many workers claim different jobs safely.
- **Counters**: use atomic `UPDATE ... SET n = n + 1`, or Redis for very hot counters. Avoid read-modify-write in app code.
- **Full-text search in PostgreSQL**: `tsvector` + GIN index, good enough for many apps before adding Elasticsearch.
- **Hierarchies** (categories, comments): adjacency list (`parent_id`) with recursive CTEs, or materialized path / closure table for heavy reads.
- **Multi-tenancy**: `tenant_id` on every table + RLS, or schema/database per tenant.
- **Time-series tables**: partition by time, drop old partitions for retention.
- **Event sourcing / CQRS**: store events as the source of truth and build read models. Powerful but complex, so use only when needed.
- **Reference/lookup tables** for statuses and types when the set changes often.

---

## 20. Choosing a database

Ask in order:

1. **What is the data like?** Structured and related → relational. Nested, variable documents → document. Simple key lookups → key-value. Connections are the point → graph. Timestamped metrics → time-series. Text search → search engine. Embeddings → vector.
2. **What are the access patterns?** Mostly reads or writes? Ad-hoc queries or fixed ones? Joins and aggregations?
3. **What consistency do you need?** Money and inventory → strong consistency and transactions. Feeds and analytics → eventual consistency is often fine.
4. **What scale?** Most apps never outgrow one well-tuned PostgreSQL server (plus replicas and a cache).
5. **What does your team know and operate well?** Operational skill matters more than benchmarks.
6. **Managed or self-hosted?** Managed services save a lot of operational pain.

**Default advice**: **PostgreSQL** for your primary data. **Redis** for caching and fast ephemeral data. Add search, vector, or analytics stores only when you can name the problem they solve. SQLite is excellent for local tools, tests, and small apps.

---

## 21. Learning roadmap

1. **SQL basics**: `SELECT`, `WHERE`, `ORDER BY`, `INSERT/UPDATE/DELETE` (use PostgreSQL locally or in Docker).
2. **Joins and aggregation**: `JOIN`, `GROUP BY`, `HAVING`, subqueries.
3. **Schema design**: keys, relationships, constraints, normalization. Draw ER diagrams.
4. **Indexes and `EXPLAIN`**: learn to read query plans.
5. **Transactions, ACID, isolation levels**, locks, race conditions.
6. **From code**: pooling, parameterized queries, an ORM, migrations, N+1.
7. **CTEs and window functions** for real-world reporting.
8. **Redis** and at least one document database (MongoDB).
9. **Replication, backups, restores, and monitoring**.
10. **Scaling and distributed concepts**: partitioning, sharding, CAP, consistency models.

**Practice projects**

1. **Library or to-do schema**: tables, FKs, constraints, and 20 practice queries.
2. **E-commerce schema**: users, products, orders, inventory with transactions to prevent overselling.
3. **Blog with comments and tags**: many-to-many, pagination, full-text search.
4. **URL shortener**: unique constraint, indexes, Redis cache.
5. **Analytics reports**: window functions over a sample dataset.
6. **Migration drill**: rename a column with zero downtime using expand and contract.
7. **Backup drill**: take a backup, destroy a table, restore it, measure the time.

Good practice resources: sample databases (Pagila, Northwind), SQL exercise sites, and reading the official PostgreSQL docs.

---

## 22. Checklist: things to remember

1. A database gives **durability, concurrency control, constraints, and fast queries** that files cannot.
2. **Start with PostgreSQL.** Add other stores only for a clear need.
3. Every table has a **primary key**. Index **foreign keys**. Use **constraints** to protect data.
4. **Normalize** to avoid duplicated facts, **denormalize** only with evidence.
5. Money → `NUMERIC` or integer cents. Time → `TIMESTAMPTZ` in UTC. Never floats for money.
6. `NULL` is "unknown": use `IS NULL`, beware `NOT IN`.
7. Indexes speed reads but slow writes. Check with **`EXPLAIN`**. Mind the **leftmost prefix rule**.
8. Select only needed columns, **paginate**, avoid **N+1**, batch writes.
9. **Transactions are ACID**. Keep them short, and never call external services inside them.
10. Know the **isolation levels**, MVCC, and fix races with **atomic updates, unique constraints, row locks, or optimistic locking**.
11. **Retry** on deadlocks and serialization failures.
12. Use **parameterized queries**, **connection pools**, and **migrations** (expand and contract).
13. Never expose the database publicly, use **least-privilege roles**, TLS, encryption, and secret managers.
14. **Replicas are not backups.** Automate backups, enable PITR, and **test restores**.
15. Scale in order: **queries/indexes → cache → replicas → partitioning → sharding**.
16. Understand **replication lag**, **CAP**, and **eventual consistency**.
17. **Monitor** slow queries, connections, replication lag, disk space, and autovacuum.

---

## 23. Common pitfalls and interview questions

**Pitfalls**

- `UPDATE` or `DELETE` **without a `WHERE`** clause.
- Building SQL by string concatenation (**SQL injection**).
- No index on foreign keys or on frequently filtered columns, and the opposite, **too many indexes**.
- `SELECT *` everywhere, unbounded queries, deep `OFFSET` pagination.
- **N+1 queries** hidden behind an ORM.
- Using `FLOAT` for money, or storing local time instead of UTC.
- Cramming lists into one column (`"a,b,c"`) instead of a related table.
- Treating `NULL` like a normal value (`= NULL`, `NOT IN` with nulls).
- Long transactions, or HTTP calls inside transactions.
- Read-modify-write in application code instead of atomic updates (race conditions, overselling).
- Pool sizes that exceed the database connection limit, no timeouts.
- Editing production schemas by hand, or breaking migrations without expand and contract.
- Treating a **replica as a backup**, never testing a restore.
- Giving the app a superuser account, or exposing the database to the internet.
- Choosing NoSQL "because it scales" without a real need, then re-implementing joins and transactions in code.
- Sharding too early.
- Storing large files in the database.

**Interview questions**

- **SQL vs NoSQL: when do you choose which?**
  SQL for structured, related data needing strong consistency and transactions (the default). NoSQL when you have a specific need: flexible documents, massive horizontal scale or write volume, key-value speed, graphs, search, or vectors.
- **What is normalization and why do it? When would you denormalize?**
  Organizing data so each fact is stored once (1NF–3NF) to avoid update/insert/delete anomalies. Denormalize deliberately for read performance, accepting extra write complexity.
- **Explain primary key, foreign key, and unique constraint.**
  PK uniquely identifies a row. FK references another table's key and enforces referential integrity. Unique forbids duplicate values in a column or set of columns.
- **INNER vs LEFT vs FULL JOIN?**
  Inner returns only matches. Left keeps all left rows (nulls where unmatched). Full keeps all rows from both sides.
- **`WHERE` vs `HAVING`?**
  `WHERE` filters rows before grouping. `HAVING` filters groups after aggregation.
- **What is an index and what is its cost? What is a composite index?**
  A sorted structure (usually a B-tree) for fast lookups. It costs write speed, memory, and disk. A composite index covers several columns and follows the leftmost-prefix rule.
- **Why might a query not use an index?**
  Function on the column, leading-wildcard `LIKE`, type mismatch, low selectivity, stale statistics, or the leftmost prefix not matched.
- **What does `EXPLAIN ANALYZE` show?**
  The execution plan with estimated and actual costs, rows and timings: scans, joins, sorts. Used to find missing indexes and bad estimates.
- **What is ACID?**
  Atomicity (all or nothing), Consistency (rules hold), Isolation (concurrent transactions do not interfere), Durability (committed data survives crashes).
- **Explain the isolation levels and the anomalies they prevent.**
  Read Committed stops dirty reads, Repeatable Read also stops non-repeatable reads (and phantoms in PostgreSQL), Serializable behaves as if transactions ran one at a time. Higher levels cost more waiting or aborts.
- **How do you prevent two users from buying the last item?**
  An atomic update (`SET qty = qty - 1 WHERE qty > 0`), a row lock (`SELECT ... FOR UPDATE`), optimistic locking with a version column, or Serializable isolation with retries.
- **What is a deadlock and how do you handle it?**
  Two transactions waiting on each other's locks. The database aborts one. Reduce with consistent lock ordering and short transactions, and retry on failure.
- **What is MVCC?**
  Multi-Version Concurrency Control: keeping multiple row versions so readers do not block writers and vice versa. Old versions are vacuumed later.
- **What is the N+1 problem and how do you fix it?**
  One query for a list plus one per item. Fix with joins, eager loading, or `IN` batches.
- **What is connection pooling and why is it needed?**
  Reusing a limited set of open connections instead of opening one per request, because connections are expensive and limited.
- **How do you do a zero-downtime schema change?**
  Expand and contract: add new structures, dual-write, backfill in batches, switch reads, then remove old structures later.
- **Replication vs sharding vs partitioning?**
  Replication copies the same data to more servers (read scaling, availability). Partitioning splits a table within one database. Sharding splits data across multiple servers by a shard key (write and storage scaling, high complexity).
- **What is replication lag and how do you cope with it?**
  Replicas apply changes slightly later than the primary, so reads can be stale. Route read-your-own-writes to the primary, or tolerate eventual consistency.
- **Explain the CAP theorem.**
  During a network partition, a distributed system must choose between consistency and availability.
- **How do you back up and restore a production database?**
  Automated base backups plus WAL archiving for point-in-time recovery, stored off-site and encrypted, with regular restore tests. Define RPO and RTO.
- **How do you prevent SQL injection?**
  Parameterized queries or ORM bindings, least-privilege database accounts, and input validation.
- **When would you use Redis alongside PostgreSQL?**
  For caching, sessions, rate limiting, counters, leaderboards, and queues, while PostgreSQL stays the source of truth.
