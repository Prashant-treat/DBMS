 # SQL Notes: Beginner to Advanced

Practical SQL notes for backend development, database design, placements, and interviews. Examples use **PostgreSQL** syntax. Other databases share the core ideas but differ in functions, pagination, upserts, and date syntax.

## Table of Contents

1. [How to Use](#how-to-use)
2. [Priority Map](#priority-map)
3. [Database Foundations](#database-foundations)
	- [Vocabulary and ACID](#p0-vocabulary)
	- [Logical Query Order](#p0-logical-query-order)
4. [Practice Schema](#practice-schema)
5. [Database and Schema Commands](#database-and-schema-commands)
	- [Database, Schema, Session, and Comments](#p0-database-schema-session-and-comments)
6. [DDL: Tables and Constraints](#ddl-tables-and-constraints)
	- [Create, Alter, Rename, and Drop](#p0-create-alter-rename-and-remove)
	- [Types and Constraints](#p0-types-and-constraints)
	- [Truncate and Temporary Tables](#p0-truncate-and-temporary-tables)
7. [DML: Insert, Update, Delete](#dml-insert-update-delete)
	- [Insert, Update, Delete, and Returning](#p0-basic-writes)
	- [Upsert and Merge](#p1-upsert-and-merge)
	- [Import and Export](#p1-import-and-export)
8. [SELECT and Filtering](#select-and-filtering)
	- [Select, Aliases, Ordering, and Pagination](#p0-retrieval-aliases-and-ordering)
	- [Operators, Expressions, and NULL](#p0-operators-and-null)
9. [Joins](#joins)
10. [Aggregation](#aggregation)
11. [Subqueries and CTEs](#subqueries-and-ctes)
12. [Window Functions](#window-functions)
13. [Set Operations and Interview Patterns](#set-operations-and-interview-patterns)
14. [Transactions and Concurrency](#transactions-and-concurrency)
	- [Transaction Control](#p0-atomic-business-operation)
	- [Isolation and Locking](#p0-isolation-and-anomalies)
15. [Indexes and Performance](#indexes-and-performance)
	- [Indexes and Query Plans](#indexes-and-performance)
	- [Statistics and Maintenance](#p1-statistics-and-maintenance)
16. [Database Objects](#database-objects)
	- [Views and Materialized Views](#p1-views-and-materialized-views)
	- [Functions, Procedures, and Triggers](#p2p3-functions-procedures-and-triggers)
17. [Backend Security and Integration](#backend-security-and-integration)
18. [Design and Normalization](#design-and-normalization)
19. [Production DBMS Topics](#production-dbms-topics)
20. [Complete Command Checklist](#complete-command-checklist)
21. [Topic-by-Topic Command Lab](#topic-by-topic-command-lab)
22. [Which Command Is Better?](#which-command-is-better)
23. [Placement and Interview Revision](#placement-and-interview-revision)
24. [Practice Roadmap](#practice-roadmap)

## How to Use

- **P0 - Essential:** expected in almost every software or backend interview.
- **P1 - High value:** common in backend work and strong interview differentiators.
- **P2 - Specialized:** important for DBMS, distributed systems, and senior roles.
- **P3 - Optional:** vendor-specific or useful after the core path.
- Run every example, change the data, and explain the result without notes.
- Use the same practice schema so syntax is connected to real backend behavior.

## Priority Map

| Priority | Topics | Typical use |
| --- | --- | --- |
| P0 | `SELECT`, filters, joins, grouping, keys, constraints, normalization, transactions, indexes, `NULL` | All placements and backend roles |
| P1 | CTEs, window functions, query plans, isolation anomalies, upserts, pagination, views, locking | Backend and mid-level interviews |
| P2 | MVCC, WAL, partitioning, replication, sharding, recovery, consistency | DBMS and system design interviews |
| P3 | Stored procedures, triggers, vendor tuning, extensions | DBA and specialized roles |

## Database Foundations

### [P0] Vocabulary

- **Database:** organized data. **DBMS:** software that stores, retrieves, protects, and recovers it.
- **RDBMS:** a DBMS based on related tables and constraints.
- **Table, row, column:** a relation, a record, and an attribute.
- **Schema:** logical namespace and structure. **Instance:** data stored at a particular time.
- **Primary key:** unique, non-null row identity. **Foreign key:** reference to a parent key.
- **Candidate key:** minimal unique attribute set. **Surrogate key:** generated identity such as an integer or UUID.
- **OLTP:** many small concurrent business operations. **OLAP:** large analytical scans and aggregations.
- SQL is declarative: the optimizer chooses how to produce the requested result.

### [P0] Integrity and ACID

- **Entity integrity:** primary keys are unique and not `NULL`.
- **Referential integrity:** foreign keys reference valid parent rows.
- **Atomicity:** all operations in a transaction succeed or none do.
- **Consistency:** constraints and business invariants remain valid.
- **Isolation:** concurrent operations do not expose invalid intermediate states.
- **Durability:** committed data survives a crash.

### [P0] Logical query order

`FROM` / `JOIN` -> `WHERE` -> `GROUP BY` -> `HAVING` -> `SELECT` -> `DISTINCT` -> `ORDER BY` -> `LIMIT`.

This explains why a `SELECT` alias usually cannot be used in `WHERE`, and why filtering before grouping can reduce work.

## Database and Schema Commands

### [P0] Database, schema, session, and comments

These commands prepare the environment before tables are created. Run them in order when creating a local practice database; permissions for production are normally handled by a DBA or deployment system.

```sql
-- 1. Create and connect to a database. CREATE DATABASE is run outside a transaction.
CREATE DATABASE shop;
\c shop

-- 2. Group objects in a named schema.
CREATE SCHEMA sales;
SET search_path TO sales, public;

-- 3. Inspect or change the current session.
SHOW search_path;
SET TIME ZONE 'UTC';
SET statement_timeout = '5s';

-- 4. Document objects for the next developer.
COMMENT ON SCHEMA sales IS 'Order and payment tables';
```

`\c` is a `psql` client command, not portable SQL. `SHOW` reads a setting and `SET` changes it for the current session. Use a schema to separate application tables, reporting objects, and migrations.

## Practice Schema

Use this e-commerce schema for every scenario. A customer places orders; an order contains products.

```sql
CREATE TABLE customers (
	customer_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
	email TEXT NOT NULL UNIQUE,
	full_name TEXT NOT NULL,
	city TEXT,
	created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE products (
	product_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
	name TEXT NOT NULL,
	category TEXT NOT NULL,
	price NUMERIC(12, 2) NOT NULL CHECK (price >= 0),
	stock_quantity INTEGER NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
	active BOOLEAN NOT NULL DEFAULT true
);

CREATE TABLE orders (
	order_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
	customer_id BIGINT NOT NULL REFERENCES customers(customer_id),
	status TEXT NOT NULL DEFAULT 'pending'
		CHECK (status IN ('pending', 'paid', 'shipped', 'cancelled')),
	ordered_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE order_items (
	order_id BIGINT NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
	product_id BIGINT NOT NULL REFERENCES products(product_id),
	quantity INTEGER NOT NULL CHECK (quantity > 0),
	unit_price NUMERIC(12, 2) NOT NULL CHECK (unit_price >= 0),
	PRIMARY KEY (order_id, product_id)
);
```

The copied `unit_price` preserves the price paid even when the current product price changes.

## DDL: Tables and Constraints

### [P0] Create, alter, rename, and remove

**Scenario:** add a product SKU during a migration.

```sql
ALTER TABLE products ADD COLUMN sku TEXT;
UPDATE products SET sku = 'SKU-' || product_id WHERE sku IS NULL;
ALTER TABLE products ADD CONSTRAINT products_sku_unique UNIQUE (sku);
ALTER TABLE products ALTER COLUMN sku SET NOT NULL;
ALTER TABLE products RENAME COLUMN sku TO product_sku;
ALTER TABLE products RENAME COLUMN product_sku TO sku;

-- Only after checking dependencies and production data.
DROP TABLE IF EXISTS temporary_import;
```

`DROP` removes an object. `TRUNCATE` removes all rows quickly. `DELETE` can filter rows and fire row-level triggers. Production schema changes belong in reviewed, versioned migrations.

### [P0] Truncate and temporary tables

**Scenario:** load a fresh staging file without affecting the permanent product table.

```sql
CREATE TEMP TABLE product_import (
	sku TEXT,
	name TEXT,
	price NUMERIC(12, 2)
);

TRUNCATE TABLE product_import;
DROP TABLE IF EXISTS product_import;
```

`TRUNCATE` is for removing all rows from a table, such as resetting a staging table. It is not a filtered delete. `CREATE TEMP TABLE` creates a session-scoped table that is automatically removed when the connection ends.

### [P0] Types and constraints

- Use `BIGINT` for large IDs, `NUMERIC` for money, `TIMESTAMPTZ` for instants, and `BOOLEAN` for flags.
- Prefer database constraints: `NOT NULL`, `CHECK`, `UNIQUE`, `PRIMARY KEY`, and `FOREIGN KEY`.
- Use `ON DELETE CASCADE` only when a child has no meaning without its parent.
- A PostgreSQL `UNIQUE` constraint permits multiple `NULL`s because `NULL` means unknown.

## DML: Insert, Update, Delete

### [P0] Basic writes

**Scenario:** onboard a customer, change a category price, and cancel only a pending order.

```sql
INSERT INTO customers (email, full_name, city)
VALUES ('aisha@example.com', 'Aisha Khan', 'Pune')
RETURNING customer_id;

UPDATE products
SET price = price * 1.10
WHERE category = 'books' AND active = true;

DELETE FROM orders
WHERE order_id = 42 AND status = 'pending';
```

Always preview the equivalent `SELECT` before a production `UPDATE` or `DELETE`.

### [P1] Upsert and merge

**Scenario:** an idempotent API receives the same email twice.

```sql
INSERT INTO customers (email, full_name, city)
VALUES ('aisha@example.com', 'Aisha Khan', 'Pune')
ON CONFLICT (email) DO UPDATE
SET full_name = EXCLUDED.full_name, city = EXCLUDED.city
RETURNING customer_id;
```

The unique constraint handles the race between concurrent requests; a check-then-insert sequence does not.

For synchronizing a complete source with a target, PostgreSQL 15+ also supports `MERGE`:

```sql
MERGE INTO products AS target
USING product_import AS source ON target.sku = source.sku
WHEN MATCHED THEN
	UPDATE SET name = source.name, price = source.price
WHEN NOT MATCHED THEN
	INSERT (sku, name, category, price)
	VALUES (source.sku, source.name, 'uncategorized', source.price);
```

Use `ON CONFLICT` for a single-row API upsert and `MERGE` for a source-to-target synchronization job.

### [P1] Import and export

**Scenario:** move a CSV extract into a staging table and export a report.

```sql
COPY product_import (sku, name, price)
FROM '/var/lib/postgresql/import/products.csv'
WITH (FORMAT csv, HEADER true);

COPY (
	SELECT category, COUNT(*) AS product_count
	FROM products
	GROUP BY category
	ORDER BY category
) TO '/var/lib/postgresql/export/product_counts.csv'
WITH (FORMAT csv, HEADER true);
```

`COPY` reads or writes files on the database server. In `psql`, use `\copy` to read or write files on the client machine. Validate the staging data before merging it into production tables.

## SELECT and Filtering

### [P0] Retrieval, aliases, and ordering

**Scenario:** show active products with stable pagination.

```sql
SELECT product_id, name, price
FROM products
WHERE active = true
ORDER BY price DESC, product_id DESC
LIMIT 20;
```

Prefer explicit columns over `SELECT *` in APIs. A unique secondary sort prevents tied rows from moving between pages.

```sql
SELECT DISTINCT category FROM products ORDER BY category;

SELECT name,
	   CASE WHEN stock_quantity = 0 THEN 'out_of_stock'
			WHEN stock_quantity < 10 THEN 'low_stock'
			ELSE 'in_stock' END AS stock_status
FROM products;
```

### [P0] Operators and `NULL`

```sql
SELECT * FROM products
WHERE category IN ('books', 'games')
  AND price BETWEEN 10 AND 50
  AND name ILIKE '%sql%';

SELECT * FROM customers WHERE city IS NULL;
SELECT COALESCE(city, 'Unknown') AS display_city FROM customers;
SELECT NULLIF(stock_quantity, 0) FROM products;
```

`NULL` is not zero, false, or an empty string. Comparisons with it produce `UNKNOWN`; use `IS NULL` and `IS NOT NULL`. Avoid `NOT IN` when its subquery can return `NULL`; prefer `NOT EXISTS`.

## Joins

### [P0] Join types

**Scenario:** show every customer, including customers without orders.

```sql
SELECT c.customer_id, c.full_name, o.order_id, o.status
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.customer_id
ORDER BY c.customer_id, o.order_id;
```

- `INNER JOIN`: matching rows only.
- `LEFT JOIN`: every left row plus matches.
- `RIGHT JOIN`: usually rewrite by swapping tables and using `LEFT JOIN`.
- `FULL OUTER JOIN`: unmatched rows from both sides.
- `CROSS JOIN`: every combination; use only intentionally.
- Self join: a table joined to itself, such as employee-manager data.

To preserve customers with no paid orders, put the filter in `ON`, not `WHERE`:

```sql
SELECT c.customer_id, o.order_id
FROM customers AS c
LEFT JOIN orders AS o
  ON o.customer_id = c.customer_id AND o.status = 'paid';
```

Joins across one-to-many relationships multiply rows. Use `COUNT(DISTINCT ...)` or `EXISTS` when that multiplication is not desired.

## Aggregation

### [P0] GROUP BY, HAVING, and counts

**Scenario:** find customers whose paid orders total at least 10,000.

```sql
SELECT c.customer_id, c.full_name,
	   SUM(oi.quantity * oi.unit_price) AS lifetime_value
FROM customers AS c
JOIN orders AS o ON o.customer_id = c.customer_id
JOIN order_items AS oi ON oi.order_id = o.order_id
WHERE o.status = 'paid'
GROUP BY c.customer_id, c.full_name
HAVING SUM(oi.quantity * oi.unit_price) >= 10000
ORDER BY lifetime_value DESC;
```

`WHERE` filters rows before grouping; `HAVING` filters groups after aggregation.

```sql
SELECT c.customer_id,
	   COUNT(o.order_id) AS order_count,
	   COUNT(DISTINCT o.order_id) AS distinct_orders,
	   COALESCE(SUM(oi.quantity), 0) AS units_bought
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.customer_id
LEFT JOIN order_items AS oi ON oi.order_id = o.order_id
GROUP BY c.customer_id;
```

`COUNT(*)` counts preserved `LEFT JOIN` rows. `COUNT(column)` ignores `NULL`. `COUNT(DISTINCT ...)` prevents one-to-many multiplication from inflating a count.

## Subqueries and CTEs

### [P0] `EXISTS`, `IN`, and scalar queries

**Scenario:** find products that have never been ordered.

```sql
SELECT p.product_id, p.name
FROM products AS p
WHERE NOT EXISTS (
	SELECT 1 FROM order_items AS oi
	WHERE oi.product_id = p.product_id
);
```

`EXISTS` expresses a yes/no relationship and avoids duplicate result rows. A correlated subquery refers to the outer row; an uncorrelated subquery does not.

### [P0] CTEs

**Scenario:** calculate order totals, then filter expensive orders.

```sql
WITH order_totals AS (
	SELECT order_id, SUM(quantity * unit_price) AS total
	FROM order_items
	GROUP BY order_id
)
SELECT o.order_id, o.customer_id, ot.total
FROM orders AS o
JOIN order_totals AS ot USING (order_id)
WHERE ot.total > 500;
```

CTEs improve readability and organize multi-step statements. They are not automatically faster than subqueries; inspect the plan when performance matters.

### [P1] Recursive CTEs

**Scenario:** traverse an employee hierarchy.

```sql
WITH RECURSIVE org AS (
	SELECT employee_id, manager_id, name, 0 AS depth
	FROM employees WHERE manager_id IS NULL
	UNION ALL
	SELECT e.employee_id, e.manager_id, e.name, org.depth + 1
	FROM employees AS e JOIN org ON org.employee_id = e.manager_id
)
SELECT * FROM org ORDER BY depth, employee_id;
```

## Window Functions

### [P0] Ranking

**Scenario:** return the top two active products by price in each category.

```sql
WITH ranked AS (
	SELECT p.*, ROW_NUMBER() OVER (
		PARTITION BY category ORDER BY price DESC, product_id
	) AS position
	FROM products AS p
	WHERE active = true
)
SELECT * FROM ranked WHERE position <= 2;
```

- `ROW_NUMBER()` gives unique positions.
- `RANK()` leaves gaps after ties.
- `DENSE_RANK()` does not leave gaps after ties.
- `PARTITION BY` creates independent groups without collapsing rows.

### [P1] Running totals and comparisons

```sql
SELECT ordered_at::date AS day,
	   COUNT(*) AS daily_orders,
	   SUM(COUNT(*)) OVER (ORDER BY ordered_at::date) AS running_orders
FROM orders
GROUP BY ordered_at::date
ORDER BY day;

SELECT order_id, customer_id, ordered_at,
	   LAG(ordered_at) OVER (
		   PARTITION BY customer_id ORDER BY ordered_at
	   ) AS previous_order_at
FROM orders;
```

Window functions preserve row detail. `GROUP BY` collapses rows.

## Set Operations and Interview Patterns

### [P0] Set operations

```sql
SELECT email FROM customers
UNION
SELECT email FROM newsletter_subscribers;

SELECT email FROM customers
UNION ALL
SELECT email FROM newsletter_subscribers;
```

`UNION` removes duplicates; `UNION ALL` preserves them and is usually faster. `INTERSECT` returns common rows. `EXCEPT` returns rows in the first query but not the second. Both queries need compatible column counts and types.

### [P1] Reusable problem patterns

- **Top N per group:** `ROW_NUMBER() OVER (PARTITION BY group ORDER BY metric DESC)`.
- **Latest row per entity:** rank by timestamp descending and keep rank one.
- **Second highest value:** use `DENSE_RANK` when ties matter.
- **Deduplication:** rank by the business key and keep the first row.
- **Gaps and islands:** compare a value with `LAG`, or use row-number differences.
- **Retention/cohorts:** group by signup period, join activity by relative period, count distinct users.

**Scenario: keyset pagination for a large orders table.**

```sql
SELECT order_id, customer_id, ordered_at
FROM orders
WHERE (ordered_at, order_id) < ('2026-01-15 10:00:00+00', 9000)
ORDER BY ordered_at DESC, order_id DESC
LIMIT 50;
```

Keyset pagination is more stable and scalable than a large `OFFSET`.

## Transactions and Concurrency

### [P0] Atomic business operation

**Scenario:** place an order and decrement stock together.

```sql
BEGIN;

INSERT INTO orders (customer_id, status)
VALUES (1, 'paid')
RETURNING order_id;

UPDATE products
SET stock_quantity = stock_quantity - 1
WHERE product_id = 10 AND stock_quantity >= 1;

-- The application checks that exactly one row was updated.
COMMIT;
-- Use ROLLBACK if any step fails.
```

`SAVEPOINT` permits partial rollback:

```sql
BEGIN;
SAVEPOINT before_optional_item;
-- attempt optional operation
ROLLBACK TO SAVEPOINT before_optional_item;
COMMIT;
```

Keep transactions short. Do not wait for user input or call a remote API inside a write transaction.

### [P0] Isolation and anomalies

| Level | Main idea | Concern |
| --- | --- | --- |
| Read uncommitted | May see uncommitted writes; PostgreSQL treats it as read committed | Rarely appropriate |
| Read committed | Each statement gets a committed snapshot | Later statements may see newer data |
| Repeatable read | Stable transaction snapshot | Conflicting work can fail |
| Serializable | Enforces serializable behavior | Application retries may be needed |

Know: dirty read, non-repeatable read, phantom read, lost update, write skew, deadlock, serialization failure, shared lock, exclusive lock, optimistic locking, and pessimistic locking.

### [P1] Row locking

```sql
BEGIN;
SELECT stock_quantity FROM products
WHERE product_id = 10 FOR UPDATE;

UPDATE products
SET stock_quantity = stock_quantity - 1
WHERE product_id = 10 AND stock_quantity > 0;
COMMIT;
```

Lock rows in a consistent order to reduce deadlocks. Retry transient deadlocks and serialization failures with bounded backoff.

## Indexes and Performance

### [P0] Index basics

An index speeds reads but consumes storage and slows writes. Index columns used in selective filters, joins, ordering, and uniqueness checks.

```sql
CREATE INDEX orders_customer_date_idx
ON orders (customer_id, ordered_at DESC);

CREATE UNIQUE INDEX customers_email_idx ON customers (email);
```

Composite index order matters: `(customer_id, ordered_at)` helps a customer-and-date query, but may not help a query filtering only by date.

### [P1] Partial, expression, and covering indexes

```sql
CREATE INDEX active_products_category_idx
ON products (category, price) WHERE active = true;

CREATE INDEX customers_lower_email_idx ON customers (lower(email));

CREATE INDEX orders_customer_covering_idx
ON orders (customer_id, ordered_at DESC) INCLUDE (status);
```

Do not index every column. Consider selectivity, write rate, table size, and actual workload.

### [P0] Query plans

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT order_id, ordered_at
FROM orders
WHERE customer_id = 1
ORDER BY ordered_at DESC
LIMIT 20;
```

Compare estimated and actual row counts. Look for unnecessary sequential scans, expensive sorts, bad join order, and repeated loops. `EXPLAIN ANALYZE` executes the query, so use care with mutating statements in production.

### [P1] Statistics and maintenance

**Scenario:** refresh planner statistics after a large import and inspect table health.

```sql
ANALYZE products;
VACUUM (ANALYZE) products;

-- PostgreSQL administration examples; schedule carefully in production.
REINDEX TABLE products;
```

`ANALYZE` updates statistics used by the optimizer. `VACUUM` cleans obsolete row versions and helps prevent bloat. `REINDEX` rebuilds an index when corruption or severe bloat is suspected. These commands are maintenance tools, not substitutes for a missing or incorrect index.

## Database Objects

### [P1] Views and materialized views

**Scenario:** publish a stable reporting interface.

```sql
CREATE VIEW paid_order_totals AS
SELECT o.order_id, o.customer_id,
	   SUM(oi.quantity * oi.unit_price) AS total
FROM orders AS o
JOIN order_items AS oi USING (order_id)
WHERE o.status = 'paid'
GROUP BY o.order_id, o.customer_id;
```

A normal view stores a query. A materialized view stores results and needs refresh:

```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY daily_sales;
```

### [P2/P3] Functions, procedures, and triggers

Functions reuse database logic and return a value. Procedures implement database-side workflows and are called with `CALL`. Triggers run automatically when a table event occurs. Use them for local invariants or auditing; prefer explicit application code for cross-service business workflows.

**Scenario:** create a reusable order-total function and an audit trigger.

```sql
CREATE OR REPLACE FUNCTION order_total(input_order_id BIGINT)
RETURNS NUMERIC(12, 2)
LANGUAGE sql
STABLE
AS $$
	SELECT COALESCE(SUM(quantity * unit_price), 0)
	FROM order_items
	WHERE order_id = input_order_id;
$$;

SELECT order_total(42);
```

```sql
CREATE TABLE order_audit (
	audit_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
	order_id BIGINT NOT NULL,
	action TEXT NOT NULL,
	changed_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE OR REPLACE FUNCTION audit_order_insert()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
	INSERT INTO order_audit (order_id, action)
	VALUES (NEW.order_id, 'INSERT');
	RETURN NEW;
END;
$$;

CREATE TRIGGER orders_insert_audit
AFTER INSERT ON orders
FOR EACH ROW EXECUTE FUNCTION audit_order_insert();
```

To remove a trigger or routine during a migration, use `DROP TRIGGER` or `DROP FUNCTION` with the exact signature.

## Backend Security and Integration

### [P0] Parameterized queries

Never concatenate user input into SQL.

```text
Bad:  "SELECT * FROM users WHERE email = '" + email + "'"
Good: "SELECT * FROM users WHERE email = $1", [email]
```

Parameters protect values, not table names or sort directions. Allowlist dynamic identifiers and sort fields. Parameterization prevents SQL injection and may improve plan reuse.

### [P1] Roles and least privilege

```sql
CREATE ROLE app_readwrite LOGIN PASSWORD 'set-through-secret-management';
GRANT CONNECT ON DATABASE shop TO app_readwrite;
GRANT USAGE ON SCHEMA public TO app_readwrite;
GRANT SELECT, INSERT, UPDATE ON customers, orders, order_items TO app_readwrite;
```

Use secret management, separate migration and runtime roles, revoke unnecessary defaults, and never give the application superuser access.

### [P0] Backend rules

- Use a connection pool and release connections in a `finally` path.
- Use one transaction for one business invariant, not for an entire request chain.
- Set statement and lock timeouts.
- Map unique, foreign-key, check, deadlock, and serialization errors deliberately.
- Use versioned migrations, not manual production edits.
- Use `RETURNING` to obtain generated IDs without a second race-prone query.
- Make retries idempotent with an idempotency key and unique constraint.
- Log duration and safe query metadata, never passwords or sensitive values.

## Design and Normalization

### [P0] ER modeling

Model entities, attributes, relationships, cardinality, and optionality before writing tables. A many-to-many relationship becomes a junction table such as `order_items`.

### [P0] Normal forms

- **1NF:** atomic values and no repeating groups.
- **2NF:** 1NF plus no partial dependency on part of a composite key.
- **3NF:** 2NF plus no transitive dependency of non-key columns on a key.
- **BCNF:** every determinant is a candidate key.

Normalization prevents insert, update, and delete anomalies. Denormalize only for a measured read or reporting need, and define how duplicate data remains consistent.

### [P1] Design decisions

- Normalize transactional data; use summaries or materialized views for analytics.
- Choose `CASCADE`, `RESTRICT`, soft delete, or archival based on retention needs.
- Store timestamps with timezone awareness and convert only for display.
- Use numeric types for money, not floating point.
- Define one-to-one, one-to-many, and many-to-many relationships explicitly.
- Use UUIDs for distributed generation or opaque public IDs; use integers when compact sequential IDs are preferable.

## Production DBMS Topics

### [P1] Backup and recovery

- **RPO:** maximum acceptable data loss measured in time.
- **RTO:** maximum acceptable recovery time.
- Full backups, incremental backups, point-in-time recovery, WAL archiving, and restore drills are separate concerns.
- A backup is not proven until it has been restored and validated.

### [P2] MVCC, WAL, and vacuum

- MVCC gives readers consistent snapshots while writers proceed.
- WAL records changes before data pages are flushed, enabling crash recovery and replication.
- Old row versions require cleanup; PostgreSQL uses vacuum and autovacuum.
- Long-running transactions can prevent cleanup and cause bloat.

### [P2] Replication, partitioning, and sharding

- Read replicas improve read capacity and disaster recovery but introduce replication lag.
- Synchronous replication trades latency for stronger acknowledgement guarantees.
- Partitioning divides one logical table, often by time or tenant, and can improve pruning and maintenance.
- Sharding distributes data across nodes but adds routing, rebalancing, cross-shard transaction, and query complexity.
- Caches reduce database load but require invalidation, expiry, and consistency decisions.

### [P2] OLTP and analytics

Keep request-path queries bounded and indexed. Move heavy aggregations to replicas, reporting databases, materialized views, or an analytical warehouse. Do not solve a workload problem by adding random indexes to a transactional schema.

## Complete Command Checklist

Use this as a final revision sheet. For each command, know its purpose, its safest use case, and whether it changes data or only reads metadata.

### DDL: Data Definition Language

| Command | What it does | Practical use case |
| --- | --- | --- |
| `CREATE DATABASE` | Creates a database | Create a local development database |
| `CREATE SCHEMA` | Creates a namespace | Separate application and reporting objects |
| `CREATE TABLE` | Creates a table | Define customers, orders, or products |
| `CREATE TEMP TABLE` | Creates a session-scoped table | Stage an import without permanent storage |
| `ALTER TABLE` | Changes table structure | Add a column or constraint in a migration |
| `RENAME` | Renames a table, column, or object | Correct a schema name during a migration |
| `DROP` | Removes an object | Remove an obsolete table, view, or index |
| `TRUNCATE` | Removes every row efficiently | Reset a disposable staging table |
| `CREATE INDEX` | Adds a read access path | Speed customer order history queries |
| `CREATE VIEW` | Saves a reusable query | Publish a stable reporting interface |
| `CREATE MATERIALIZED VIEW` | Saves query results | Cache an expensive daily report |
| `CREATE FUNCTION` | Defines reusable database logic | Calculate an order total consistently |
| `CREATE PROCEDURE` | Defines callable database workflow | Run a database-side batch operation |
| `CREATE TRIGGER` | Runs logic on table events | Audit inserts or enforce a local invariant |
| `COMMENT ON` | Documents an object | Explain a non-obvious column or schema |

### DML: Data Manipulation Language

| Command | What it does | Practical use case |
| --- | --- | --- |
| `INSERT` | Adds rows | Create a customer or order |
| `UPDATE` | Changes existing rows | Change product price or order status |
| `DELETE` | Removes selected rows | Delete a test customer or pending order |
| `MERGE` | Synchronizes source and target | Apply a catalog import to products |
| `TRUNCATE` | Removes all rows | Clear a staging table before reload |
| `COPY` / `\copy` | Bulk imports or exports rows | Load or produce a CSV report |
| `RETURNING` | Returns affected rows | Get a generated ID after an insert |
| `ON CONFLICT` | Handles a uniqueness conflict | Make an API retry idempotent |

### DQL: Data Query Language

| Command or clause | What it does | Practical use case |
| --- | --- | --- |
| `SELECT` | Reads rows or expressions | Build an API response |
| `FROM` | Chooses the source relation | Read from a table, view, or CTE |
| `JOIN` | Combines related rows | Show customers with their orders |
| `WHERE` | Filters individual rows | Return only active products |
| `GROUP BY` | Creates aggregate groups | Count orders by customer |
| `HAVING` | Filters aggregate groups | Find customers above a spend threshold |
| `ORDER BY` | Sorts the result | Show newest orders first |
| `DISTINCT` | Removes duplicate result rows | List unique product categories |
| `LIMIT` / `OFFSET` | Limits or skips result rows | Small-page browsing; use keyset pagination at scale |
| `WITH` | Defines a CTE | Break a complex report into named steps |
| `UNION` / `UNION ALL` | Combines compatible result sets | Combine customer and subscriber emails |
| `INTERSECT` | Returns common rows | Find users in two campaigns |
| `EXCEPT` | Returns rows missing from another set | Find products never ordered |
| `OVER` | Applies a window calculation | Rank products inside each category |
| `EXPLAIN` | Shows the planned execution | Diagnose a slow query before changing indexes |

### TCL: Transaction Control Language

| Command | What it does | Practical use case |
| --- | --- | --- |
| `BEGIN` / `START TRANSACTION` | Starts a transaction | Group order creation and stock update |
| `COMMIT` | Makes changes permanent | Finish a successful business operation |
| `ROLLBACK` | Undoes uncommitted changes | Recover from a failed payment or validation |
| `SAVEPOINT` | Creates a partial rollback point | Undo an optional item but keep the order |
| `ROLLBACK TO SAVEPOINT` | Returns to a savepoint | Recover one failed sub-operation |
| `SET TRANSACTION` | Configures isolation or access mode | Use serializable behavior for a critical operation |
| `LOCK TABLE` | Takes a table lock explicitly | Coordinate a rare schema-sensitive batch |
| `SELECT ... FOR UPDATE` | Locks selected rows | Prevent two checkouts from spending the same stock |

### DCL: Access Control and administration

| Command | What it does | Practical use case |
| --- | --- | --- |
| `CREATE ROLE` / `CREATE USER` | Creates an identity or group | Create a restricted runtime account |
| `GRANT` | Gives privileges | Allow an API to read and write selected tables |
| `REVOKE` | Removes privileges | Remove accidental access after a role change |
| `SET ROLE` | Changes the active role | Test permissions or perform a controlled admin task |
| `ANALYZE` | Refreshes optimizer statistics | Run after a bulk load |
| `VACUUM` | Cleans obsolete row versions | Control PostgreSQL table bloat |
| `REINDEX` | Rebuilds an index | Repair or compact a problematic index |
| `REFRESH MATERIALIZED VIEW` | Recomputes stored report results | Update a daily sales dashboard |

### Command safety rules

1. Preview `UPDATE` and `DELETE` with the same `WHERE` clause in a `SELECT`.
2. Use parameters for values; never concatenate request data into SQL.
3. Wrap related writes in one transaction and verify affected-row counts.
4. Test destructive commands on a disposable database first.
5. Use migrations for DDL and record who, when, and why a production change happened.

## Topic-by-Topic Command Lab

This lab gives one separate command example for each small topic. Run the practice schema first, then execute the examples in order. The examples are PostgreSQL syntax and are intentionally small so you can change one line and observe the result.

### 1. Create a table

**Use case:** create a table for shipment tracking.

```sql
CREATE TABLE shipments (
	shipment_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
	order_id BIGINT NOT NULL,
	tracking_code TEXT NOT NULL UNIQUE,
	shipped_at TIMESTAMPTZ
);
```

### 2. Add or change a column

**Use case:** add a delivery status during a versioned migration.

```sql
ALTER TABLE shipments ADD COLUMN status TEXT DEFAULT 'created';
ALTER TABLE shipments
	ADD CONSTRAINT shipments_status_check
	CHECK (status IN ('created', 'in_transit', 'delivered'));
```

### 3. Add a primary key and foreign key

**Use case:** connect each shipment to exactly one order.

```sql
ALTER TABLE shipments ADD CONSTRAINT shipments_order_unique UNIQUE (order_id);
ALTER TABLE shipments
	ADD CONSTRAINT shipments_order_fk
	FOREIGN KEY (order_id) REFERENCES orders(order_id);
```

The table is intentionally created without the relationship so the next example can demonstrate adding the foreign key in a later migration.

### 4. Insert one row

**Use case:** create a customer and immediately return its generated ID to an API.

```sql
INSERT INTO customers (email, full_name, city)
VALUES ('meera@example.com', 'Meera Shah', 'Delhi')
RETURNING customer_id, created_at;
```

### 5. Insert many rows

**Use case:** seed products for a local development database.

```sql
INSERT INTO products (name, category, price, stock_quantity)
VALUES
	('SQL Handbook', 'books', 29.99, 40),
	('Keyboard', 'electronics', 75.00, 15),
	('Notebook', 'stationery', 4.50, 100);
```

### 6. Update selected rows

**Use case:** mark one paid order as shipped.

```sql
UPDATE orders
SET status = 'shipped'
WHERE order_id = 42 AND status = 'paid'
RETURNING order_id, status;
```

### 7. Delete selected rows

**Use case:** remove test customers created by a local seed script.

```sql
DELETE FROM customers
WHERE email LIKE '%@test.local'
RETURNING customer_id, email;
```

### 8. Read and alias columns

**Use case:** return API-friendly names without changing the database schema.

```sql
SELECT product_id AS id,
	   name AS product_name,
	   price AS current_price
FROM products;
```

### 9. Filter rows

**Use case:** show products that can currently be purchased.

```sql
SELECT product_id, name, price
FROM products
WHERE active = true
  AND stock_quantity > 0
  AND price <= 100;
```

### 10. Sort and paginate

**Use case:** show the first page of newest orders.

```sql
SELECT order_id, customer_id, ordered_at
FROM orders
ORDER BY ordered_at DESC, order_id DESC
LIMIT 20 OFFSET 0;
```

### 11. Handle NULL values

**Use case:** display a customer city even when it was not supplied.

```sql
SELECT full_name,
	   COALESCE(city, 'City not provided') AS display_city
FROM customers;
```

### 12. Use conditional expressions

**Use case:** label products for an inventory dashboard.

```sql
SELECT name,
	   CASE
		   WHEN stock_quantity = 0 THEN 'out_of_stock'
		   WHEN stock_quantity < 10 THEN 'reorder'
		   ELSE 'healthy'
	   END AS inventory_state
FROM products;
```

### 13. Inner join

**Use case:** show orders only when their customer record is present.

```sql
SELECT o.order_id, c.full_name, o.status
FROM orders AS o
INNER JOIN customers AS c ON c.customer_id = o.customer_id;
```

### 14. Left join

**Use case:** show every customer, even a customer with zero orders.

```sql
SELECT c.customer_id, c.full_name, o.order_id
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.customer_id;
```

### 15. Self join

**Use case:** compare products in the same category without comparing a product to itself.

```sql
SELECT product.name AS first_product,
	   similar.name AS second_product,
	   product.category
FROM products AS product
JOIN products AS similar
	ON similar.category = product.category
   AND similar.product_id > product.product_id;
```

### 16. Aggregate rows

**Use case:** count orders for every customer.

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id;
```

### 17. Filter aggregate results

**Use case:** find customers with at least five orders.

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) >= 5;
```

### 18. Use a subquery

**Use case:** find products priced above the average product price.

```sql
SELECT product_id, name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

### 19. Use EXISTS

**Use case:** find customers who have placed at least one paid order without duplicating customers.

```sql
SELECT c.customer_id, c.full_name
FROM customers AS c
WHERE EXISTS (
	SELECT 1
	FROM orders AS o
	WHERE o.customer_id = c.customer_id
	  AND o.status = 'paid'
);
```

### 20. Use a CTE

**Use case:** calculate order totals in one named step and filter them in the next.

```sql
WITH totals AS (
	SELECT order_id, SUM(quantity * unit_price) AS total
	FROM order_items
	GROUP BY order_id
)
SELECT order_id, total
FROM totals
WHERE total > 500;
```

### 21. Use a window function

**Use case:** rank products inside each category while keeping product rows.

```sql
SELECT product_id, name, category, price,
	   ROW_NUMBER() OVER (
		   PARTITION BY category ORDER BY price DESC, product_id
	   ) AS category_rank
FROM products;
```

### 22. Use set operations

**Use case:** combine two compatible product lists without removing duplicates.

```sql
SELECT product_id FROM products WHERE category = 'books'
UNION ALL
SELECT product_id FROM products WHERE price < 10;
```

### 23. Run an upsert

**Use case:** safely retry a customer registration request.

```sql
INSERT INTO customers (email, full_name, city)
VALUES ('meera@example.com', 'Meera Shah', 'Delhi')
ON CONFLICT (email) DO UPDATE
SET full_name = EXCLUDED.full_name,
	city = EXCLUDED.city
RETURNING customer_id;
```

### 24. Run a transaction

**Use case:** create an order and reduce stock as one atomic business operation.

```sql
BEGIN;
INSERT INTO orders (customer_id, status)
VALUES (1, 'paid')
RETURNING order_id;

UPDATE products
SET stock_quantity = stock_quantity - 1
WHERE product_id = 10 AND stock_quantity > 0;

COMMIT;
```

If either operation fails, run `ROLLBACK` instead of `COMMIT`.

### 25. Lock a row

**Use case:** prevent two checkouts from changing the same inventory row at the same time.

```sql
BEGIN;
SELECT stock_quantity
FROM products
WHERE product_id = 10
FOR UPDATE;
-- Check the value, then perform the update.
COMMIT;
```

### 26. Create and use an index

**Use case:** speed a customer's order-history endpoint.

```sql
CREATE INDEX orders_customer_date_idx
ON orders (customer_id, ordered_at DESC);

SELECT order_id, status, ordered_at
FROM orders
WHERE customer_id = 1
ORDER BY ordered_at DESC
LIMIT 20;
```

### 27. Inspect a query plan

**Use case:** verify whether the order-history query uses the index.

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT order_id, status, ordered_at
FROM orders
WHERE customer_id = 1
ORDER BY ordered_at DESC
LIMIT 20;
```

### 28. Create a view

**Use case:** give reporting code a reusable paid-order query.

```sql
CREATE OR REPLACE VIEW paid_orders AS
SELECT order_id, customer_id, ordered_at
FROM orders
WHERE status = 'paid';

SELECT * FROM paid_orders WHERE customer_id = 1;
```

### 29. Grant and revoke access

**Use case:** allow an application to read products but not alter the schema.

```sql
GRANT SELECT ON products TO app_readwrite;
REVOKE DELETE ON products FROM app_readwrite;
```

### 30. Maintain statistics

**Use case:** refresh the optimizer after a large product import.

```sql
ANALYZE products;
VACUUM (ANALYZE) products;
```

### 31. Import and export CSV data

**Use case:** load a validated staging file and export a report.

```sql
CREATE TEMP TABLE product_import (
	sku TEXT,
	name TEXT,
	price NUMERIC(12, 2)
);

COPY product_import (sku, name, price)
FROM '/var/lib/postgresql/import/products.csv'
WITH (FORMAT csv, HEADER true);

COPY (SELECT category, COUNT(*) FROM products GROUP BY category)
TO '/var/lib/postgresql/export/product_counts.csv'
WITH (FORMAT csv, HEADER true);
```

Use `\copy` in `psql` when the file is on the client machine rather than the database server.

## Which Command Is Better?

There is no universally best command. Choose based on whether you need to preserve rows, preserve the table, remove duplicates, protect concurrency, or optimize for readability and scale.

### [P0] `DELETE` vs `TRUNCATE` vs `DROP`

| Choose | Best use case | Keeps table structure? | Filtering? | Main caution |
| --- | --- | --- | --- | --- |
| `DELETE FROM table WHERE ...` | Remove selected customer or test rows | Yes | Yes | Can be slow for many rows; check the `WHERE` clause |
| `TRUNCATE TABLE table` | Empty a staging or temporary table completely | Yes | No | Removes every row; takes a strong lock |
| `DROP TABLE table` | Permanently remove an obsolete table | No | No | Data and dependent objects may be lost |

**Rule:** use `DELETE` for business data, `TRUNCATE` for disposable complete resets, and `DROP` only for schema removal. For an auditable business record, prefer a status update or soft delete over physical deletion.

### [P0] `WHERE` vs `HAVING`

| Choose | Filters | Example use case |
| --- | --- | --- |
| `WHERE` | Individual rows before grouping | Only paid orders enter a sales total |
| `HAVING` | Groups after aggregation | Keep customers whose total exceeds 10,000 |

```sql
-- Better: filter rows before the expensive grouping step.
SELECT customer_id, SUM(total_amount)
FROM orders
WHERE status = 'paid'
GROUP BY customer_id
HAVING SUM(total_amount) > 10000;
```

Use `WHERE` when the condition does not depend on an aggregate. Use `HAVING` only when it needs `SUM`, `COUNT`, `AVG`, `MIN`, or `MAX`.

### [P0] `INNER JOIN` vs `LEFT JOIN` vs `EXISTS`

| Choose | Result | Best use case |
| --- | --- | --- |
| `INNER JOIN` | Only matching rows | Return orders that have a valid customer |
| `LEFT JOIN` | Every left row, with optional match | Show every customer, including those with no orders |
| `EXISTS` | A true/false match without child duplication | Check whether a customer has at least one paid order |

```sql
-- Better than joining and using DISTINCT when only existence matters.
SELECT c.customer_id, c.full_name
FROM customers AS c
WHERE EXISTS (
	SELECT 1 FROM orders AS o
	WHERE o.customer_id = c.customer_id AND o.status = 'paid'
);
```

Use a join when columns from both tables are needed. Use `EXISTS` when the question is only “does a related row exist?”.

### [P0] `COUNT(*)` vs `COUNT(column)` vs `COUNT(DISTINCT column)`

| Choose | Counts | Best use case |
| --- | --- | --- |
| `COUNT(*)` | Result rows, including rows with `NULL` values | Count rows in a table or preserved `LEFT JOIN` rows |
| `COUNT(column)` | Non-`NULL` values in that column | Count customers who have an order |
| `COUNT(DISTINCT column)` | Unique non-`NULL` values | Count unique orders after joining order items |

**Rule:** after a one-to-many join, question whether the join multiplied the rows before choosing `COUNT`. Use `COUNT(DISTINCT order_id)` when each order must count once.

### [P0] `UNION` vs `UNION ALL`

| Choose | Behavior | Best use case |
| --- | --- | --- |
| `UNION` | Combines results and removes duplicates | A unique list of users from two sources |
| `UNION ALL` | Combines results and keeps duplicates | Append monthly partitions or preserve event counts |

Prefer `UNION ALL` when duplicates are valid or already impossible; it avoids the extra deduplication work. Use `UNION` only when duplicate removal is part of the requirement.

### [P0] `IN` vs `EXISTS` vs `NOT EXISTS`

| Choose | Best use case | Important behavior |
| --- | --- | --- |
| `IN (fixed values)` | Small, readable list such as categories | Clear for a known list |
| `IN (subquery)` | Membership in a clean, non-null result | Check a value against a set |
| `EXISTS` | Correlated relationship check | Stops when a match is found and avoids duplicates |
| `NOT EXISTS` | Find rows with no related record | Safe when the subquery contains `NULL`s |

For anti-joins, prefer `NOT EXISTS` over `NOT IN` unless the subquery column is guaranteed `NOT NULL`:

```sql
SELECT p.product_id, p.name
FROM products AS p
WHERE NOT EXISTS (
	SELECT 1 FROM order_items AS oi
	WHERE oi.product_id = p.product_id
);
```

### [P0] `CASE` vs `COALESCE` vs `NULLIF`

| Choose | Purpose | Example use case |
| --- | --- | --- |
| `CASE` | Multiple conditional branches | Label stock as low, available, or empty |
| `COALESCE` | First non-`NULL` value | Display `Unknown` when city is missing |
| `NULLIF` | Convert a matching value to `NULL` | Avoid division by zero when a count is zero |

Do not use `COALESCE` to hide invalid required data; enforce required values with `NOT NULL` when appropriate.

### [P0] `UPDATE` vs `INSERT ... ON CONFLICT` vs `MERGE`

| Choose | Best use case | Why |
| --- | --- | --- |
| `UPDATE` | The row must already exist | Clear intent for changing an existing order |
| `INSERT ... ON CONFLICT` | One API request may create or update one key | Atomic and ideal for idempotent requests |
| `MERGE` | Synchronize many source rows with a target | Express matched and unmatched bulk behavior |

Use a unique constraint as the final protection against duplicates. Do not implement an upsert with a separate `SELECT` followed by `INSERT` without handling concurrent requests.

### [P0] Offset vs keyset pagination

| Choose | Best use case | Tradeoff |
| --- | --- | --- |
| `LIMIT ... OFFSET ...` | Small admin pages or jumping to a known page | Gets slower and less stable at large offsets |
| Keyset condition plus `LIMIT` | Infinite scroll, feeds, and large tables | Fast and stable, but cannot jump directly to page 100 |

For a frequently changing order feed, prefer keyset pagination with a unique ordering pair such as `(ordered_at, order_id)`.

### [P0] `GROUP BY` vs window functions

| Choose | Result shape | Best use case |
| --- | --- | --- |
| `GROUP BY` | One row per group | Total sales per customer |
| Window function with `OVER` | Keeps each source row | Rank every product within its category |

Use `GROUP BY` when detail rows are no longer needed. Use a window function when the output needs both the original row and a calculation such as rank, running total, `LAG`, or `LEAD`.

### [P1] CTE vs subquery vs temporary table

| Choose | Best use case | Tradeoff |
| --- | --- | --- |
| Subquery | One small nested calculation | Compact but can become difficult to read |
| CTE (`WITH`) | Several named logical steps | Clear structure; not automatically faster |
| Temporary table | Reuse an intermediate result across statements | Can be indexed, but requires lifecycle and storage management |

Start with a subquery for a simple expression, use a CTE for a readable single statement, and use a temporary table when multiple statements reuse or index the intermediate data.

### [P1] View vs materialized view vs table

| Choose | Stores | Best use case |
| --- | --- | --- |
| View | Query definition | Always-current reusable read model |
| Materialized view | Query result | Expensive report refreshed on a schedule |
| Table | Application-owned data | Source of truth that receives writes |

Do not use a materialized view for data that must be real-time unless its refresh strategy meets the requirement.

### [P1] `FOR UPDATE` vs optimistic locking

| Choose | Best use case | Tradeoff |
| --- | --- | --- |
| `SELECT ... FOR UPDATE` | Short, high-conflict stock or balance update | Holds a database lock while the transaction runs |
| Optimistic version check | Low-conflict edits such as profile updates | Failed update must be detected and retried or reported |

Optimistic example:

```sql
UPDATE products
SET price = 49.99, version = version + 1
WHERE product_id = 10 AND version = 7;
-- If affected rows = 0, another request changed the row first.
```

### [P1] Composite vs partial vs expression indexes

| Choose | Best use case | Key decision |
| --- | --- | --- |
| Composite index | Queries filter or sort by multiple columns | Put the commonly filtered leading column first |
| Partial index | A small, frequently queried subset | Add a predicate such as `WHERE active = true` |
| Expression index | Queries filter on a calculated expression | Index the exact expression, such as `lower(email)` |

Always confirm the choice with `EXPLAIN (ANALYZE, BUFFERS)` and real workload data. An index is not automatically beneficial just because a column appears in a query.

### [P1] `COMMIT` vs `ROLLBACK` vs `SAVEPOINT`

| Choose | Best use case | Result |
| --- | --- | --- |
| `COMMIT` | Every business operation succeeds | Makes all transaction changes durable |
| `ROLLBACK` | A required operation fails | Undoes the entire transaction |
| `SAVEPOINT` | An optional step can fail independently | Undoes only work after the savepoint |

For an order, payment authorization and inventory changes should normally use `ROLLBACK` on failure; a savepoint is appropriate only when an optional operation can safely be omitted.

## Placement and Interview Revision

### Must-answer P0 questions

1. Differentiate primary key, candidate key, unique key, and foreign key.
2. Explain `WHERE` versus `HAVING`, `INNER JOIN` versus `LEFT JOIN`, and `COUNT(*)` versus `COUNT(column)`.
3. What is `NULL`, and why can `NOT IN` surprise you?
4. Explain normalization with an update anomaly.
5. Explain ACID and transaction boundaries.
6. Describe dirty reads, lost updates, phantom reads, and deadlocks.
7. What is an index, and when can it hurt performance?
8. Write second-highest, top-N-per-group, latest-row-per-user, and duplicate queries.
9. Why are parameterized queries required?
10. Design users, orders, payments, inventory, and order items.

### Strong P1 questions

- Explain composite indexes and why column order matters.
- Read `EXPLAIN ANALYZE` and identify the expensive operation.
- Compare offset and keyset pagination.
- Explain optimistic versus pessimistic locking and prevent overselling.
- Explain isolation levels, MVCC, and retryable errors.
- Design an idempotent payment endpoint.
- Explain replication lag, read-after-write consistency, and cache invalidation.

### SQL interview method

1. Clarify output columns, duplicates, ties, `NULL`s, and time boundaries.
2. State the result grain: one row per customer, order, day, or other entity.
3. Build the smallest correct query, then add joins, aggregation, or windows.
4. Test empty input, duplicates, ties, missing relationships, and `NULL`s.
5. Explain cost, useful indexes, and behavior under concurrency.

## Practice Roadmap

### Week 1: Foundations

Create the schema, insert realistic data, draw the ER diagram, and solve basic `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and constraint exercises.

### Week 2: Query fluency

Solve at least 30 problems covering filters, joins, grouping, `NULL`, subqueries, and set operations. Recreate them from memory.

### Week 3: Interview SQL

Solve ranking, running totals, deduplication, latest-row, gaps-and-islands, cohort, and retention problems. State the result grain first.

### Week 4: Backend reliability

Implement an order endpoint with a transaction, parameterized queries, an idempotency key, inventory locking, migrations, indexes, and error handling. Test concurrent requests.

### Final project checklist

- ER diagram and normalized schema
- Migration files and seed data
- CRUD and reporting queries
- Constraints and deliberate delete behavior
- Transaction and retry strategy
- Query plans for important endpoints
- Backup and restore procedure
- Least-privilege roles
- Tests for duplicates, `NULL`s, concurrency, and authorization
