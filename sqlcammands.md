# SQL Commands: Complete Beginner-to-Advanced Notes

This is a practical SQL command handbook. Examples use **PostgreSQL** syntax. SQL is mostly portable, but database vendors differ in identity columns, date functions, pagination, upserts, stored routines, and administration commands.

Run the examples in order in a disposable database. Never run destructive examples against production without a reviewed migration, a backup, and a rollback plan.

## Contents

1. [SQL and DBMS foundations](#1-sql-and-dbms-foundations)
2. [Database setup commands](#2-database-setup-commands)
3. [Practice schema](#3-practice-schema)
4. [DDL: Data Definition Language](#4-ddl-data-definition-language)
5. [DML: Data Manipulation Language](#5-dml-data-manipulation-language)
6. [DQL: Querying data](#6-dql-querying-data)
7. [Operators, expressions, and functions](#7-operators-expressions-and-functions)
8. [Joins](#8-joins)
9. [Grouping and aggregation](#9-grouping-and-aggregation)
10. [Subqueries, CTEs, and set operations](#10-subqueries-ctes-and-set-operations)
11. [Window functions](#11-window-functions)
12. [TCL: Transactions and concurrency](#12-tcl-transactions-and-concurrency)
13. [Indexes and performance](#13-indexes-and-performance)
14. [Views, functions, procedures, and triggers](#14-views-functions-procedures-and-triggers)
15. [DCL: Users, roles, and privileges](#15-dcl-users-roles-and-privileges)
16. [Import, export, and maintenance](#16-import-export-and-maintenance)
17. [Design and normalization](#17-design-and-normalization)
18. [Security and application integration](#18-security-and-application-integration)
19. [Command revision checklist](#19-command-revision-checklist)

## 1. SQL and DBMS foundations

### What SQL means

- **SQL:** Structured Query Language, used to define, read, change, and secure relational data.
- **Database:** A collection of related data and database objects.
- **DBMS:** Software that stores data, manages concurrent users, enforces rules, and recovers from failures.
- **RDBMS:** A DBMS based on tables and relationships.
- **Table:** Data arranged as rows and columns.
- **Row:** One record; **column:** one attribute of a record.
- **Schema:** A namespace containing tables and other objects.
- **Primary key:** The unique, non-null identity of a row.
- **Foreign key:** A reference to a key in another table.
- **Constraint:** A database rule that protects valid data.

### SQL command families

| Family | Meaning | Main commands |
| --- | --- | --- |
| DDL | Define database structure | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| DML | Insert and change rows | `INSERT`, `UPDATE`, `DELETE`, `MERGE` |
| DQL | Read data | `SELECT` |
| TCL | Control transactions | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` |
| DCL | Control access | `GRANT`, `REVOKE`, `CREATE ROLE` |

### Important rules

1. SQL keywords are conventionally written in uppercase; names are usually lowercase.
2. End statements with `;`.
3. Use `--` for a one-line comment and `/* ... */` for a block comment.
4. `NULL` means unknown or missing. It is not zero, `false`, or an empty string.
5. SQL is declarative: you describe the result, and the optimizer chooses an execution plan.

### Logical order of a query

Although SQL is written as `SELECT ... FROM ...`, it is logically processed as:

```text
FROM / JOIN -> WHERE -> GROUP BY -> HAVING -> SELECT -> DISTINCT -> ORDER BY -> LIMIT
```

This explains why a `SELECT` alias normally cannot be used in `WHERE`, and why `WHERE` filters rows before aggregation.

## 2. Database setup commands

These examples are PostgreSQL commands. `\c`, `\dt`, and `\d` are `psql` client commands, not portable SQL.

```sql
-- Create a database. Run this outside a transaction.
CREATE DATABASE shop;

-- Connect when using the psql client.
\c shop

-- Create and use a namespace.
CREATE SCHEMA sales;
SET search_path TO sales, public;

-- Inspect and change the current session.
SHOW search_path;
SET TIME ZONE 'UTC';
SET statement_timeout = '5s';

-- PostgreSQL psql inspection commands.
\dt
\d sales.customers
\dn

-- Document an object.
COMMENT ON SCHEMA sales IS 'Orders and payment data';
COMMENT ON TABLE sales.customers IS 'People who place orders';
```

`CREATE DATABASE` creates a database. `CREATE SCHEMA` creates a namespace inside it. `SET` changes the current session only. Use separate migration, application, and read-only roles in real systems.

## 3. Practice schema

Run this schema first. The tables demonstrate one-to-many and many-to-many relationships, keys, defaults, and constraints.

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
	sku TEXT UNIQUE,
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

`order_items` uses a composite primary key because one product should appear once per order. Its `unit_price` is a snapshot of the price paid; it must not change when the catalog price changes.

## 4. DDL: Data Definition Language

DDL creates or changes database objects.

### `CREATE TABLE`

```sql
CREATE TABLE departments (
	department_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
	name TEXT NOT NULL UNIQUE,
	budget NUMERIC(14, 2) DEFAULT 0 CHECK (budget >= 0),
	created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Common data types:

| Type | Use |
| --- | --- |
| `INTEGER`, `BIGINT` | Whole numbers and IDs |
| `NUMERIC(precision, scale)` | Exact money and measurements |
| `TEXT`, `VARCHAR(n)` | Text; use a check constraint when length matters |
| `BOOLEAN` | True/false flags |
| `DATE` | Calendar date without time |
| `TIMESTAMP` | Date and time without timezone |
| `TIMESTAMPTZ` | An instant in time; preferred for events |
| `UUID` | Distributed or public identifiers |
| `JSONB` | Queryable semi-structured data in PostgreSQL |
| `TEXT[]` | PostgreSQL array values; use carefully |

### Constraints

```sql
CREATE TABLE employees (
	employee_id BIGINT PRIMARY KEY,
	email TEXT CONSTRAINT employees_email_unique UNIQUE,
	name TEXT NOT NULL,
	salary NUMERIC(12, 2) CHECK (salary >= 0),
	department_id INTEGER,
	CONSTRAINT employees_department_fk
		FOREIGN KEY (department_id) REFERENCES departments(department_id)
		ON DELETE SET NULL
);
```

- `NOT NULL`: a value is required.
- `UNIQUE`: no two non-null values may be equal by default.
- `PRIMARY KEY`: unique row identity; implies `NOT NULL`.
- `FOREIGN KEY`: prevents references to missing parent rows.
- `CHECK`: rejects values that do not satisfy a Boolean expression.
- `DEFAULT`: supplies a value when the column is omitted; it does not replace an explicit `NULL`.

### `ALTER TABLE`

```sql
ALTER TABLE products ADD COLUMN description TEXT;
ALTER TABLE products ALTER COLUMN description SET DEFAULT '';
ALTER TABLE products ALTER COLUMN description DROP DEFAULT;
ALTER TABLE products DROP COLUMN description;
ALTER TABLE products RENAME COLUMN name TO product_name;
ALTER TABLE products RENAME TO catalog_products;
ALTER TABLE catalog_products RENAME TO products;
```

Add and backfill a required column safely in stages: add it nullable, backfill it, validate it, then make it `NOT NULL`. Large production changes may need batching to avoid long locks.

### `DROP`, `TRUNCATE`, and temporary tables

```sql
CREATE TEMP TABLE product_import (
	sku TEXT,
	name TEXT,
	price NUMERIC(12, 2)
);

TRUNCATE TABLE product_import;
TRUNCATE TABLE product_import RESTART IDENTITY;
DROP TABLE IF EXISTS product_import;
DROP TABLE IF EXISTS old_report CASCADE;
```

| Command | Effect |
| --- | --- |
| `DELETE` | Removes selected rows, can use `WHERE`, and is row-oriented |
| `TRUNCATE` | Removes every row quickly; cannot filter with `WHERE` |
| `DROP` | Removes the object itself, including its definition |
| `CREATE TEMP TABLE` | Creates a session-scoped table |

`CASCADE` also removes dependent objects. Use it only after inspecting dependencies.

## 5. DML: Data Manipulation Language

### `INSERT`

```sql
INSERT INTO customers (email, full_name, city)
VALUES ('aisha@example.com', 'Aisha Khan', 'Pune');

INSERT INTO products (name, category, price, stock_quantity)
VALUES
	('SQL Basics', 'books', 25.00, 10),
	('Keyboard', 'electronics', 75.00, 5);

INSERT INTO customers (email, full_name)
SELECT email, full_name FROM customer_import;

INSERT INTO customers (email, full_name)
VALUES ('same@example.com', 'Same User')
RETURNING customer_id, created_at;
```

Always list target columns. `RETURNING` returns inserted or changed rows without a second query.

### `UPDATE`

```sql
UPDATE products
SET price = price * 1.10,
	active = true
WHERE category = 'books'
  AND active = true
RETURNING product_id, price;
```

Before a production update, run the same `WHERE` clause as a `SELECT`. An `UPDATE` without `WHERE` changes every row.

### `DELETE`

```sql
DELETE FROM orders
WHERE order_id = 42
  AND status = 'pending'
RETURNING order_id;
```

Use soft deletion when history or legal retention matters:

```sql
ALTER TABLE customers ADD COLUMN deleted_at TIMESTAMPTZ;
UPDATE customers SET deleted_at = now() WHERE customer_id = 7;
SELECT * FROM customers WHERE deleted_at IS NULL;
```

### Upsert with `ON CONFLICT`

```sql
INSERT INTO customers (email, full_name, city)
VALUES ('aisha@example.com', 'Aisha Khan', 'Pune')
ON CONFLICT (email) DO UPDATE
SET full_name = EXCLUDED.full_name,
	city = EXCLUDED.city
RETURNING customer_id;
```

`EXCLUDED` means the row that attempted to insert. The unique constraint makes this safe under concurrent requests.

### `MERGE`

Use `MERGE` to synchronize a source table with a target table (PostgreSQL 15+):

```sql
MERGE INTO products AS target
USING product_import AS source ON target.sku = source.sku
WHEN MATCHED THEN
	UPDATE SET name = source.name, price = source.price
WHEN NOT MATCHED THEN
	INSERT (sku, name, category, price)
	VALUES (source.sku, source.name, 'uncategorized', source.price);
```

## 6. DQL: Querying data

### `SELECT`, aliases, expressions, and `DISTINCT`

```sql
SELECT product_id, name AS product_name, price,
	   price * 1.18 AS price_with_tax
FROM products;

SELECT DISTINCT category
FROM products
ORDER BY category;
```

Avoid `SELECT *` in application APIs because schema changes can alter payloads and increase I/O.

### `WHERE` and filtering

```sql
SELECT product_id, name, price
FROM products
WHERE active = true
  AND price BETWEEN 10 AND 100
  AND category IN ('books', 'games')
  AND name ILIKE '%sql%';
```

Useful predicates include `=`, `<>`, `>`, `<`, `>=`, `<=`, `BETWEEN`, `IN`, `LIKE`, `ILIKE` (PostgreSQL), `IS NULL`, `IS NOT NULL`, `EXISTS`, and `NOT EXISTS`.

### `ORDER BY`, `LIMIT`, and `OFFSET`

```sql
SELECT product_id, name, price
FROM products
ORDER BY price DESC, product_id DESC
LIMIT 20 OFFSET 40;
```

Use a deterministic tie-breaker such as the primary key. Large `OFFSET` values become slow; prefer keyset pagination:

```sql
SELECT product_id, name, price
FROM products
WHERE (price, product_id) < (50.00, 100)
ORDER BY price DESC, product_id DESC
LIMIT 20;
```

### `NULL`

```sql
SELECT * FROM customers WHERE city IS NULL;
SELECT COALESCE(city, 'Unknown') AS display_city FROM customers;
SELECT NULLIF(stock_quantity, 0) FROM products;
```

Never use `city = NULL`; use `IS NULL`. Comparisons with `NULL` produce `UNKNOWN`. Be careful with `NOT IN` when its input can contain `NULL`; `NOT EXISTS` is usually safer.

## 7. Operators, expressions, and functions

### Conditional expressions

```sql
SELECT name,
	   CASE
		   WHEN stock_quantity = 0 THEN 'out_of_stock'
		   WHEN stock_quantity < 10 THEN 'low_stock'
		   ELSE 'in_stock'
	   END AS stock_status
FROM products;
```

### String, numeric, and date functions

```sql
SELECT lower(trim(email)), length(full_name), concat(full_name, ' - ', city)
FROM customers;

SELECT round(price * 1.18, 2), abs(-12), greatest(price, 100)
FROM products;

SELECT current_date, now(), date_trunc('month', ordered_at),
	   ordered_at::date
FROM orders;
```

### PostgreSQL JSONB

```sql
CREATE TABLE events (
	event_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
	payload JSONB NOT NULL
);

SELECT payload ->> 'event_type' AS event_type,
	   payload -> 'customer' ->> 'email' AS email
FROM events
WHERE payload @> '{"event_type": "purchase"}';
```

Use structured columns for frequently queried, constrained data. Use `JSONB` for genuinely variable attributes, not as an excuse to avoid modeling.

## 8. Joins

### Inner and outer joins

```sql
-- Matching customers and orders only.
SELECT c.full_name, o.order_id, o.status
FROM customers AS c
JOIN orders AS o ON o.customer_id = c.customer_id;

-- Every customer, including those without orders.
SELECT c.full_name, o.order_id
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.customer_id;

-- Keep customers without paid orders: put the filter in ON.
SELECT c.customer_id, o.order_id
FROM customers AS c
LEFT JOIN orders AS o
	ON o.customer_id = c.customer_id
   AND o.status = 'paid';
```

Join types:

- `INNER JOIN`: only rows with a match.
- `LEFT JOIN`: all left rows plus matching right rows.
- `RIGHT JOIN`: all right rows; usually rewrite as a `LEFT JOIN`.
- `FULL OUTER JOIN`: all rows from both sides.
- `CROSS JOIN`: every combination; use intentionally.
- Self join: a table joined to itself, useful for employee-manager hierarchies.

### Many-to-many joins

```sql
SELECT o.order_id, p.name, oi.quantity, oi.unit_price
FROM orders AS o
JOIN order_items AS oi ON oi.order_id = o.order_id
JOIN products AS p ON p.product_id = oi.product_id;
```

One-to-many joins multiply rows. Use `COUNT(DISTINCT id)` or `EXISTS` when you need to avoid counting the same parent more than once.

## 9. Grouping and aggregation

### Aggregate functions

`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `STRING_AGG`, and `ARRAY_AGG` summarize rows.

```sql
SELECT category,
	   COUNT(*) AS product_count,
	   AVG(price) AS average_price,
	   MIN(price) AS cheapest,
	   MAX(price) AS most_expensive
FROM products
GROUP BY category
HAVING COUNT(*) >= 2
ORDER BY product_count DESC;
```

`WHERE` filters rows before grouping. `HAVING` filters groups after aggregation. `COUNT(*)` counts rows; `COUNT(column)` ignores nulls; `COUNT(DISTINCT column)` removes duplicate values.

```sql
SELECT c.customer_id,
	   COUNT(DISTINCT o.order_id) AS order_count,
	   COALESCE(SUM(oi.quantity * oi.unit_price), 0) AS lifetime_value
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.customer_id
LEFT JOIN order_items AS oi ON oi.order_id = o.order_id
GROUP BY c.customer_id
HAVING COALESCE(SUM(oi.quantity * oi.unit_price), 0) >= 100;
```

## 10. Subqueries, CTEs, and set operations

### Scalar and membership subqueries

```sql
SELECT name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);

SELECT c.customer_id, c.email
FROM customers AS c
WHERE EXISTS (
	SELECT 1 FROM orders AS o
	WHERE o.customer_id = c.customer_id
);

SELECT p.product_id, p.name
FROM products AS p
WHERE NOT EXISTS (
	SELECT 1 FROM order_items AS oi
	WHERE oi.product_id = p.product_id
);
```

`EXISTS` expresses a yes/no relationship and avoids duplicate parent rows. A correlated subquery refers to a row from the outer query.

### Common table expressions with `WITH`

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

CTEs make multi-step queries readable. They are not automatically faster than a subquery; inspect the execution plan.

### Recursive CTE

```sql
WITH RECURSIVE numbers AS (
	SELECT 1 AS number
	UNION ALL
	SELECT number + 1 FROM numbers WHERE number < 5
)
SELECT number FROM numbers;
```

Recursive CTEs are useful for trees, org charts, folder paths, and graphs. Always define a stopping condition and protect against cycles.

### Set operations

```sql
SELECT email FROM customers
UNION
SELECT email FROM newsletter_subscribers;

SELECT email FROM customers
UNION ALL
SELECT email FROM newsletter_subscribers;

SELECT email FROM customers INTERSECT SELECT email FROM newsletter_subscribers;
SELECT email FROM customers EXCEPT SELECT email FROM newsletter_subscribers;
```

`UNION` removes duplicates; `UNION ALL` preserves them and is usually faster. Each query must return compatible column counts and types.

## 11. Window functions

Window functions calculate across related rows without collapsing them like `GROUP BY`.

```sql
SELECT product_id, category, name, price,
	   ROW_NUMBER() OVER (
		   PARTITION BY category ORDER BY price DESC, product_id
	   ) AS position,
	   RANK() OVER (PARTITION BY category ORDER BY price DESC) AS rank,
	   DENSE_RANK() OVER (PARTITION BY category ORDER BY price DESC) AS dense_rank
FROM products;
```

- `ROW_NUMBER()` gives every row a unique position.
- `RANK()` leaves gaps after ties.
- `DENSE_RANK()` does not leave gaps after ties.
- `PARTITION BY` creates independent groups without removing detail rows.

Top two products in each category:

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

Running totals and previous-row comparisons:

```sql
SELECT ordered_at::date AS day,
	   COUNT(*) AS daily_orders,
	   SUM(COUNT(*)) OVER (ORDER BY ordered_at::date) AS running_orders
FROM orders
GROUP BY ordered_at::date;

SELECT order_id, customer_id, ordered_at,
	   LAG(ordered_at) OVER (
		   PARTITION BY customer_id ORDER BY ordered_at
	   ) AS previous_order_at
FROM orders;
```

## 12. TCL: Transactions and concurrency

### Basic transaction commands

```sql
BEGIN;

UPDATE products
SET stock_quantity = stock_quantity - 1
WHERE product_id = 10 AND stock_quantity >= 1;

-- Check the affected-row count in the application.
COMMIT;
-- Use ROLLBACK instead when any required step fails.
```

Transactions provide ACID:

- **Atomicity:** all operations succeed or none do.
- **Consistency:** constraints and business rules remain valid.
- **Isolation:** concurrent work does not expose invalid intermediate states.
- **Durability:** committed work survives a crash.

### Savepoints and isolation

```sql
BEGIN;
SAVEPOINT before_optional_item;
-- optional operation
ROLLBACK TO SAVEPOINT before_optional_item;
COMMIT;

BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- critical read/write operation
COMMIT;
```

Isolation levels commonly discussed are read committed, repeatable read, and serializable. Know dirty reads, non-repeatable reads, phantom reads, lost updates, write skew, deadlocks, and serialization failures. Retry transient deadlocks and serialization failures.

### Row locks

```sql
BEGIN;
SELECT stock_quantity
FROM products
WHERE product_id = 10
FOR UPDATE;

UPDATE products
SET stock_quantity = stock_quantity - 1
WHERE product_id = 10 AND stock_quantity > 0;
COMMIT;
```

`FOR UPDATE` locks selected rows until commit. Keep transactions short, lock rows in a consistent order, and never wait for user input or a remote API inside a write transaction.

## 13. Indexes and performance

### Create and remove indexes

```sql
CREATE INDEX orders_customer_date_idx
ON orders (customer_id, ordered_at DESC);

CREATE UNIQUE INDEX customers_email_idx
ON customers (lower(email));

CREATE INDEX active_products_category_idx
ON products (category, price)
WHERE active = true;

DROP INDEX IF EXISTS orders_customer_date_idx;
```

Indexes speed reads and uniqueness checks but consume storage and slow writes. Composite index order matters. Do not index every column; measure real query patterns.

### Explain a query

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT order_id, ordered_at
FROM orders
WHERE customer_id = 1
ORDER BY ordered_at DESC
LIMIT 20;
```

Compare estimated and actual row counts. Investigate unnecessary sequential scans, expensive sorts, bad join choices, and repeated loops. `EXPLAIN ANALYZE` executes the statement, so do not use it casually on mutating production statements.

## 14. Views, functions, procedures, and triggers

### Views and materialized views

```sql
CREATE VIEW paid_order_totals AS
SELECT o.order_id, o.customer_id,
	   SUM(oi.quantity * oi.unit_price) AS total
FROM orders AS o
JOIN order_items AS oi USING (order_id)
WHERE o.status = 'paid'
GROUP BY o.order_id, o.customer_id;

SELECT * FROM paid_order_totals;
DROP VIEW paid_order_totals;

CREATE MATERIALIZED VIEW daily_sales AS
SELECT ordered_at::date AS day, COUNT(*) AS order_count
FROM orders
WHERE status = 'paid'
GROUP BY ordered_at::date;

REFRESH MATERIALIZED VIEW daily_sales;
```

A view stores a query definition. A materialized view stores results and must be refreshed.

### Functions and procedures

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
DROP FUNCTION order_total(BIGINT);
```

Procedures are called with `CALL` and are suitable for database-side workflows:

```sql
CREATE OR REPLACE PROCEDURE mark_order_paid(input_order_id BIGINT)
LANGUAGE SQL
AS $$
	UPDATE orders SET status = 'paid'
	WHERE order_id = input_order_id AND status = 'pending';
$$;

CALL mark_order_paid(42);
```

### Triggers

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

DROP TRIGGER orders_insert_audit ON orders;
```

Triggers run automatically. Use them for local invariants and auditing, but document hidden side effects carefully.

## 15. DCL: Users, roles, and privileges

Never put real passwords in source-controlled SQL. Use a secret manager.

```sql
CREATE ROLE app_readwrite LOGIN PASSWORD 'set-through-secret-management';
CREATE ROLE reporting NOLOGIN;

GRANT CONNECT ON DATABASE shop TO app_readwrite;
GRANT USAGE ON SCHEMA sales TO app_readwrite;
GRANT SELECT, INSERT, UPDATE ON sales.customers, sales.orders TO app_readwrite;
GRANT reporting TO app_readwrite;

REVOKE UPDATE ON sales.customers FROM app_readwrite;
SET ROLE reporting;
RESET ROLE;
```

Use separate migration, runtime, reporting, and administrator roles. Grant the least privilege required. Also grant sequence privileges when using older sequence-based identity patterns.

## 16. Import, export, and maintenance

### `COPY` and `\copy`

```sql
COPY product_import (sku, name, price)
FROM '/var/lib/postgresql/import/products.csv'
WITH (FORMAT csv, HEADER true);

COPY (
	SELECT category, COUNT(*) AS product_count
	FROM products
	GROUP BY category
) TO '/var/lib/postgresql/export/product_counts.csv'
WITH (FORMAT csv, HEADER true);
```

`COPY` uses files on the database server. `\copy` in `psql` uses files on the client machine. Validate staging data before merging it into permanent tables.

### Statistics and maintenance

```sql
ANALYZE products;
VACUUM (ANALYZE) products;
REINDEX TABLE products;
```

`ANALYZE` refreshes optimizer statistics. `VACUUM` cleans obsolete row versions. `REINDEX` rebuilds indexes and should be scheduled carefully. Backups, restore drills, WAL archiving, replication, and point-in-time recovery are operational responsibilities beyond these commands.

## 17. Design and normalization

### Relationships

- One-to-one: a row relates to at most one row in another table.
- One-to-many: one customer has many orders; store the foreign key on `orders`.
- Many-to-many: many orders contain many products; create a junction table such as `order_items`.

### Normal forms

- **1NF:** atomic values and no repeating groups.
- **2NF:** 1NF plus no partial dependency on part of a composite key.
- **3NF:** 2NF plus no transitive dependency of non-key columns on a key.
- **BCNF:** every determinant is a candidate key.

Normalization prevents insert, update, and delete anomalies. Denormalize only for a measured performance or reporting need, and define how duplicate values stay consistent. Use `NUMERIC` for money, timezone-aware timestamps for instants, and explicit constraints for business invariants.

## 18. Security and application integration

### Parameterized SQL

```text
Bad:  "SELECT * FROM users WHERE email = '" + email + "'"
Good: "SELECT * FROM users WHERE email = $1", [email]
```

Parameters protect values, not table names or sort directions. Allowlist dynamic identifiers and sort fields. Never concatenate untrusted input into SQL.

### Reliable application rules

1. Use a connection pool and always release connections.
2. Put one business invariant in one transaction.
3. Check affected-row counts for conditional updates such as stock decrements.
4. Use unique constraints and idempotency keys for retryable APIs.
5. Set statement and lock timeouts.
6. Use versioned migrations instead of manual production edits.
7. Log query duration and safe metadata, never secrets or sensitive values.
8. Treat unique, foreign-key, check, deadlock, and serialization errors deliberately.

## 19. Command revision checklist

### Complete command availability and explanation

Every command listed below is covered in this handbook. `PostgreSQL` means the command is PostgreSQL-specific or its syntax differs significantly between database systems. `psql` means it is a client command, not SQL sent to the database server.

| Command | Available here | Explanation |
| --- | --- | --- |
| `CREATE DATABASE` | Yes | Creates a new database; normally run by an administrator outside a transaction. |
| `CREATE SCHEMA` | Yes | Creates a namespace for tables, views, functions, and other objects. |
| `CREATE TABLE` | Yes | Defines a table, its columns, data types, defaults, and constraints. |
| `CREATE TEMP TABLE` | Yes | Creates a temporary table that normally disappears when the session ends. |
| `ALTER TABLE` | Yes | Adds, changes, removes, or renames columns and constraints. |
| `DROP` | Yes | Permanently removes a database object such as a table, view, function, or index. |
| `TRUNCATE` | Yes | Removes all rows quickly; it cannot use a row filter such as `WHERE`. |
| `INSERT` | Yes | Adds one or more rows, or adds rows returned by a query. |
| `UPDATE` | Yes | Changes columns in rows selected by its `WHERE` clause. |
| `DELETE` | Yes | Removes rows selected by its `WHERE` clause. |
| `MERGE` | Yes | Synchronizes a source and target using matched and not-matched actions; PostgreSQL 15+. |
| `SELECT` | Yes | Reads rows, expressions, aggregates, or metadata. |
| `FROM` | Yes | Specifies the table, view, CTE, or other source read by `SELECT`. |
| `WHERE` | Yes | Filters individual rows before grouping or window calculations. |
| `DISTINCT` | Yes | Removes duplicate result rows. |
| `JOIN` | Yes | Combines rows from related sources using a join condition. |
| `GROUP BY` | Yes | Groups rows so aggregate functions can calculate one result per group. |
| `HAVING` | Yes | Filters groups after `GROUP BY` and aggregation. |
| `ORDER BY` | Yes | Sorts the final result; add a unique tie-breaker for stable pagination. |
| `LIMIT` | Yes | Restricts the maximum number of returned rows. |
| `OFFSET` | Yes | Skips rows before returning results; large offsets can be slow. |
| `WITH` | Yes | Defines a named common table expression for a multi-step query. |
| `UNION` | Yes | Combines compatible results and removes duplicates. |
| `UNION ALL` | Yes | Combines compatible results while preserving duplicates. |
| `INTERSECT` | Yes | Returns rows that occur in both result sets. |
| `EXCEPT` | Yes | Returns rows in the first result set that are absent from the second. |
| `OVER` | Yes | Turns an aggregate or ranking expression into a window calculation. |
| `EXPLAIN` | Yes | Displays the optimizer's planned operations; `ANALYZE` also executes the statement. |
| `BEGIN` | Yes | Starts a transaction; equivalent to starting a unit of atomic work. |
| `START TRANSACTION` | Yes | Standard SQL spelling for starting a transaction; an alternative to `BEGIN`. |
| `COMMIT` | Yes | Permanently saves all successful changes in the current transaction. |
| `ROLLBACK` | Yes | Discards uncommitted changes in the current transaction. |
| `SAVEPOINT` | Yes | Creates a named point to which part of a transaction can be rolled back. |
| `ROLLBACK TO SAVEPOINT` | Yes | Undoes work after a savepoint while keeping the transaction open. |
| `SET TRANSACTION` | Yes | Sets isolation level or access mode for the current transaction. |
| `LOCK TABLE` | Yes | Takes an explicit table lock; use rarely because it can block concurrent work. |
| `SELECT ... FOR UPDATE` | Yes | Locks selected rows so another transaction cannot update them first. |
| `CREATE INDEX` | Yes | Builds a read access path; indexes improve some reads but add write and storage cost. |
| `DROP INDEX` | Yes | Removes an index that is unnecessary or no longer useful. |
| `CREATE VIEW` | Yes | Saves a query definition that can be queried like a table. |
| `CREATE MATERIALIZED VIEW` | Yes | Stores the result of a query and requires refreshes when source data changes. |
| `REFRESH MATERIALIZED VIEW` | Yes | Recomputes the stored result of a materialized view. |
| `CREATE FUNCTION` | Yes | Defines reusable database logic that returns a value or result set. |
| `CREATE PROCEDURE` | Yes | Defines a callable database workflow invoked with `CALL`. |
| `CALL` | Yes | Executes a stored procedure. |
| `CREATE TRIGGER` | Yes | Registers automatic logic for table events such as insert, update, or delete. |
| `DROP TRIGGER` | Yes | Removes a trigger from a table. |
| `COMMENT ON` | Yes | Stores documentation on a database object or column. |
| `CREATE ROLE` | Yes | Creates a login identity or a group role. |
| `GRANT` | Yes | Gives a role privileges on a database object. |
| `REVOKE` | Yes | Removes previously granted privileges. |
| `SET ROLE` | Yes | Changes the active role for the current session. |
| `RESET ROLE` | Yes | Returns the session to its original authenticated role. |
| `COPY` | Yes | Bulk imports or exports data using files accessible to the database server. |
| `\copy` | Yes | `psql` version of `COPY` that uses files on the client machine. |
| `SHOW` | Yes | Displays the current value of a session or server setting. |
| `SET` | Yes | Changes a session setting such as timezone, search path, or timeout. |
| `ANALYZE` | Yes | Updates table statistics used by the query planner. |
| `VACUUM` | Yes | Cleans obsolete row versions and helps control PostgreSQL table bloat. |
| `REINDEX` | Yes | Rebuilds an index or indexes; schedule carefully on active systems. |

The availability column means the command has an example or explanation in this file. Syntax such as identity columns, `ILIKE`, `JSONB`, `RETURNING`, `ON CONFLICT`, `EXCLUDED`, `MERGE`, and `VACUUM` is PostgreSQL-oriented and may need adaptation in MySQL, SQL Server, Oracle, SQLite, or another DBMS.

### DDL

`CREATE DATABASE`, `CREATE SCHEMA`, `CREATE TABLE`, `CREATE TEMP TABLE`, `ALTER TABLE`, `CREATE INDEX`, `CREATE VIEW`, `CREATE MATERIALIZED VIEW`, `CREATE FUNCTION`, `CREATE PROCEDURE`, `CREATE TRIGGER`, `COMMENT ON`, `TRUNCATE`, `DROP`.

### DML

`INSERT`, multi-row insert, `INSERT ... SELECT`, `UPDATE`, `DELETE`, `RETURNING`, `ON CONFLICT`, `MERGE`, `COPY`.

### DQL

`SELECT`, `FROM`, aliases, `WHERE`, `DISTINCT`, `JOIN`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT`, `OFFSET`, `WITH`, `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT`, `OVER`, `EXPLAIN`.

### TCL

`BEGIN`, `START TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `ROLLBACK TO SAVEPOINT`, `SET TRANSACTION`, `LOCK TABLE`, `SELECT ... FOR UPDATE`.

### DCL and administration

`CREATE ROLE`, `GRANT`, `REVOKE`, `SET ROLE`, `ANALYZE`, `VACUUM`, `REINDEX`, `REFRESH MATERIALIZED VIEW`.

### Safety questions for every command

- Does it read data, change rows, change structure, or change permissions?
- What happens if it fails halfway through?
- Can it run inside a transaction?
- What constraint protects the data?
- What index and query plan does it need?
- Can a retry duplicate the operation?
- How will it be backed up, monitored, and rolled back?

