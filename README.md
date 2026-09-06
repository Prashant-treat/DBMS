# DBMS Learning Roadmap

A practical, chapter-by-chapter path for learning database management systems, preparing for interviews, and building the knowledge expected in backend, data, and software engineering roles.

## How to Use This Roadmap

1. Study the chapters in order. Complete the SQL practice before moving to internals.
2. For every chapter, write queries, draw diagrams, and explain the idea without notes.
3. Use the priority labels to spend time according to your target role.
4. Build the capstone project at the end and keep your schema, queries, indexes, and design decisions in a public repository.

### Priority Legend

- **P0 - Essential:** expected in most software and backend interviews.
- **P1 - High value:** frequently used on the job or asked for mid-level roles.
- **P2 - Specialized:** important for database, distributed systems, and senior roles.
- **P3 - Optional:** useful context after the core path is complete.

## Roadmap at a Glance

| Phase | Chapters | Outcome | Suggested pace |
| --- | --- | --- | --- |
| Foundations | 0-2 | Model data and write reliable SQL | 1-2 weeks |
| Querying | 3-5 | Solve reporting and interview problems | 2-3 weeks |
| Internals | 6-9 | Explain correctness and performance | 2-3 weeks |
| Production | 10-12 | Design reliable, scalable services | 2-4 weeks |
| Specialization | 13-15 | Choose a database career direction | Ongoing |

## Chapter 0: Prerequisites

### [P0] Programming and Problem Solving

- Be comfortable with one language: Python, Java, JavaScript/TypeScript, Go, or C++.
- Know arrays, hash maps, trees, graphs, recursion, Big-O analysis, and basic debugging.
- Practice reading API documentation and writing small command-line programs.

### [P0] Operating Systems and Networking Basics

- Processes, threads, files, memory, sockets, HTTP, and TCP/IP.
- Understand why disk access is slower than memory access and why network calls fail.

**Resources:**

- [Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/)
- [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/)

## Chapter 1: Database Fundamentals

### [P0] What a DBMS Does

- Database, DBMS, RDBMS, table, row, column, schema, instance, and metadata.
- File systems versus databases; why databases provide concurrency, durability, and recovery.
- OLTP versus OLAP; relational, document, key-value, wide-column, graph, and time-series databases.

### [P0] Relational Model and Keys

- Relations, tuples, attributes, domains, candidate keys, primary keys, foreign keys, and surrogate keys.
- Entity integrity and referential integrity.

### [P0] Relational Algebra and Calculus

- Selection, projection, rename, union, difference, Cartesian product, joins, and division.
- Tuple relational calculus and domain relational calculus as the theoretical basis of SQL.

**Resources:**

- [CMU 15-445/645 Database Systems](https://15445.courses.cs.cmu.edu/)
- [Database System Concepts - Silberschatz, Korth, Sudarshan](https://www.db-book.com/)

## Chapter 2: Data Modeling and Normalization

### [P0] ER Modeling

- Entities, attributes, relationships, cardinality, participation, weak entities, and associative tables.
- Convert an ER diagram into relational tables.

### [P0] Normalization

- Functional dependencies, closure, candidate keys, 1NF, 2NF, 3NF, and BCNF.
- Update, insert, and delete anomalies.
- Denormalization tradeoffs: read performance, consistency, storage, and operational complexity.

**Practice:** Design a marketplace schema for users, products, carts, orders, payments, shipments, and reviews. State every constraint and relationship.

**Resources:**

- [Stanford Database Course](https://online.stanford.edu/courses/soe-ydatabases-databases)
- [Vertabelo Academy SQL and database design articles](https://academy.vertabelo.com/blog/)

## Chapter 3: SQL Foundations

### [P0] Data Definition and Data Manipulation

- `CREATE`, `ALTER`, `DROP`, `INSERT`, `UPDATE`, `DELETE`, and `TRUNCATE`.
- Data types, `NULL`, defaults, `CHECK`, `UNIQUE`, primary keys, and foreign keys.

### [P1] Database Objects

- Views, materialized views, sequences, identity columns, schemas, synonyms, stored procedures, functions, and triggers.
- Choose application logic, a stored routine, or a trigger deliberately; document hidden side effects.

### [P0] Querying Tables

- `SELECT`, `WHERE`, `ORDER BY`, `LIMIT`, aliases, expressions, `DISTINCT`, and `CASE`.
- Comparison, logical, string, date, and aggregate functions.

### [P0] Joins and Aggregation

- Inner, left, right, full, cross, and self joins.
- `GROUP BY`, `HAVING`, counting correctly with `NULL`, and avoiding accidental duplicate rows.

**Practice:** Solve at least 50 SQL problems and recreate each query from memory a week later.

**Resources:**

- [SQLBolt interactive lessons](https://sqlbolt.com/)
- [PostgreSQL Tutorial](https://www.postgresql.org/docs/current/tutorial.html)
- [LeetCode Database problems](https://leetcode.com/problemset/database/)

## Chapter 4: Advanced SQL and Interview Patterns

### [P0] Subqueries and Common Table Expressions

- Correlated and non-correlated subqueries, `EXISTS`, `IN`, and `WITH` clauses.
- Recursive CTEs for hierarchies and graph-like data.

### [P0] Window Functions

- `OVER`, `PARTITION BY`, ordering, frames, `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`, and running totals.

### [P0] Set Operations and Query Design

- `UNION`, `UNION ALL`, `INTERSECT`, and `EXCEPT`.
- Top-N per group, gaps and islands, retention, cohorts, deduplication, and time-series comparisons.

**Resources:**

- [Mode SQL Tutorial](https://mode.com/sql-tutorial/)
- [Advanced SQL Window Functions - PostgreSQL docs](https://www.postgresql.org/docs/current/tutorial-window.html)
- [DataLemur SQL interview questions](https://datalemur.com/questions)

## Chapter 5: Transactions and Concurrency

### [P0] ACID and Transaction Boundaries

- Atomicity, consistency, isolation, durability, `BEGIN`, `COMMIT`, `ROLLBACK`, and savepoints.
- Transaction boundaries in application code and why long transactions are dangerous.

### [P0] Isolation Levels and Anomalies

- Read uncommitted, read committed, repeatable read, snapshot isolation, and serializable.
- Dirty reads, non-repeatable reads, phantom reads, lost updates, write skew, and serialization failures.

### [P1] Locks and Deadlocks

- Shared and exclusive locks, row/page/table locks, optimistic versus pessimistic concurrency, lock waits, and deadlock detection.

### [P1] Serializability and MVCC

- Conflict and view serializability, precedence graphs, two-phase locking, strict 2PL, timestamp ordering, and validation-based control.
- Multi-version concurrency control, snapshots, vacuum/garbage collection, and the difference between isolation and durability.

**Practice:** Run two concurrent sessions in PostgreSQL and reproduce a lost update, a deadlock, and a serialization failure.

**Resources:**

- [PostgreSQL Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [Jepsen](https://jepsen.io/consistency) for consistency concepts and real-world case studies

## Chapter 6: Storage and Database Internals

### [P1] Pages, Records, and Buffer Management

- Heap files, pages, slotted pages, records, free space, buffer pools, and cache eviction.

### [P1] Write-Ahead Logging and Recovery

- WAL, redo, undo, checkpoints, crash recovery, durability, and log sequence numbers.

### [P2] Database Engine Architecture

- Parser, binder, optimizer, executor, storage manager, catalog, and client protocol.
- Volcano/iterator execution and materialization.

### [P1] Query Processing

- Relational algebra trees, logical versus physical plans, selection and projection pushdown, join order, nested-loop join, hash join, and sort-merge join.

**Resource:** [CMU Database Group YouTube playlist](https://www.youtube.com/@CMUDatabaseGroup/playlists)

## Important Concepts Quick Reference

Use this table for revision. A good interview answer should define the concept, explain the tradeoff, and give one practical example.

| Concept | What to know | Priority and placement |
| --- | --- | --- |
| Primary key | Uniquely identifies a row; cannot be `NULL` | P0, schema design; all application roles |
| Foreign key | Enforces a valid relationship between tables | P0, schema design; backend and DBA |
| Candidate key | Minimal attribute set that uniquely identifies a row | P0, relational model; interviews |
| Functional dependency | Attribute relationship used to find keys and normalize tables | P0, normalization; database interviews |
| Normalization | Reduces redundancy and modification anomalies | P0, data modeling; all engineering roles |
| Denormalization | Duplicates data deliberately for read speed or simpler access | P1, production design; backend and data engineering |
| Relational algebra | Formal operations behind query execution | P0, fundamentals; academic and database interviews |
| `NULL` and three-valued logic | Unknown is not zero or an empty string; affects comparisons and aggregates | P0, SQL; every role using SQL |
| View | Saved logical query that hides complexity or limits access | P1, SQL objects; backend and DBA |
| Materialized view | Stored query result refreshed on a schedule or event | P1, performance and analytics; data roles |
| Trigger | Database-side action caused by an insert, update, or delete | P1, SQL objects; know tradeoffs before using |
| Stored procedure/function | Reusable server-side database logic | P1, SQL objects; DBA and backend |
| ACID | Atomicity, consistency, isolation, and durability | P0, transactions; every backend interview |
| Commit and rollback | Make a transaction durable or undo its changes | P0, transactions; application development |
| Isolation level | Defines which concurrent changes a transaction can observe | P0, concurrency; backend and DBA |
| Serializability | Concurrent result is equivalent to some serial order | P1, concurrency theory; database engineering |
| MVCC | Keeps row versions so readers and writers can often proceed together | P1, concurrency internals; PostgreSQL and distributed systems |
| Two-phase locking | Growing phase acquires locks; shrinking phase releases them | P1, concurrency theory; database interviews |
| Deadlock | Transactions wait in a cycle; prevent, detect, or retry | P0, concurrency; production backend systems |
| B-tree index | Ordered index for equality, range, and sorted access | P0, indexing; all backend roles |
| Composite index | Index over multiple columns; column order controls useful predicates | P0, indexing; query performance |
| Covering index | Index contains all columns needed by a query | P1, optimization; backend and database engineering |
| Selectivity and cardinality | How unique values are and how many rows an operation produces | P1, optimization; performance work |
| Query plan | Steps chosen by the optimizer to execute a query | P0, performance; backend and DBA |
| Cost-based optimizer | Compares estimated plans using statistics and cost models | P1, internals; database engineering |
| WAL | Logs changes before data pages are written, enabling durability and recovery | P1, storage and recovery; DBA and database engineering |
| Checkpoint | Records a recovery boundary and reduces crash-recovery work | P1, recovery; DBA and SRE |
| Buffer pool | Memory cache for database pages | P1, storage internals; database engineering |
| Replication | Maintains copies of data for availability, reads, or disaster recovery | P1, scaling; backend, DBA, and SRE |
| Partitioning | Splits one logical table into smaller physical pieces | P1, scaling; data and backend engineering |
| Fragmentation | Splits rows or columns across fragments that can be reconstructed | P1, distributed design; DBMS coursework and interviews |
| Sharding | Distributes partitions across independent database nodes | P1, distributed systems; senior backend and platform roles |
| CAP tradeoff | Under a partition, a distributed system chooses consistency or availability | P1, distributed systems; system design |
| Quorum | Requires enough replicas to acknowledge reads or writes | P2, distributed databases; senior roles |
| Cache-aside | Application reads cache, then database, and invalidates or updates cache | P1, production patterns; backend |
| N+1 query | One query loads a collection, then one query per item | P0, application performance; backend |
| Connection pool | Reuses bounded database connections instead of opening one per request | P0, application operations; backend and SRE |
| Idempotency | Repeating an operation has the same intended effect | P0, payments and retries; backend |
| RPO and RTO | Maximum acceptable data loss and recovery time | P1, disaster recovery; DBA and SRE |
| Star schema | Fact table surrounded by descriptive dimension tables | P1, analytics; data analysts and engineers |
| CDC | Publishes data changes for downstream consumers | P1, data integration; data engineering |
| SQL injection | Untrusted input changes query meaning | P0, security; every application role |

## Chapter 7: Indexing and Query Performance

### [P0] B-Tree and Composite Indexes

- B-tree structure, equality and range scans, selectivity, leftmost-prefix rules, and covering indexes.
- Composite, partial, unique, expression, and descending indexes.

### [P1] Hash, Bitmap, and Specialized Indexes

- Hash indexes, bitmap indexes, full-text indexes, spatial indexes, and inverted indexes.

### [P0] Query Plans and Optimization

- `EXPLAIN`, `EXPLAIN ANALYZE`, sequential scans, index scans, joins, cardinality estimates, statistics, and cost-based optimization.
- Avoid functions on indexed columns, unnecessary columns, unbounded results, and N+1 queries.

**Practice:** Pick five slow queries, capture plans before and after an index or query rewrite, and record the measured change.

**Resources:**

- [Use The Index, Luke!](https://use-the-index-luke.com/)
- [PostgreSQL EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
- [High Performance SQLite course](https://highperformancesqlite.com/)

## Chapter 8: Reliability, Backup, and Recovery

### [P1] Failure Modes and Availability

- Process crash, disk failure, network partition, bad deployment, data corruption, and operator error.
- Availability, durability, RPO, RTO, SLI, SLO, and error budgets.

### [P1] Backup and Restore

- Full, incremental, and logical backups; point-in-time recovery; restore testing; retention and encryption.

### [P1] Observability

- Latency, throughput, error rate, connections, lock waits, cache hit ratio, replication lag, and storage growth.

**Practice:** Document a recovery runbook and test restoring a local database from backup.

## Chapter 9: Database Security

### [P0] Access Control and Safe Queries

- Authentication, authorization, roles, least privilege, ownership, grants, and revocation.
- Parameterized queries, SQL injection, secret management, TLS, and audit logs.

### [P1] Data Protection

- Encryption in transit and at rest, masking, tokenization, retention, deletion, and sensitive-data classification.

**Resource:** [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

## Chapter 10: Scaling and Distributed Databases

### [P1] Replication

- Primary/replica, synchronous/asynchronous replication, failover, read-after-write consistency, and replication lag.

### [P1] Partitioning and Sharding

- Range, hash, list, and consistent-hash partitioning; shard keys; hotspots; rebalancing; cross-shard queries.
- Horizontal fragmentation (split rows), vertical fragmentation (split columns), derived fragmentation, and the correctness tradeoffs of reconstructing fragmented data.

### [P2] Distributed Transactions and Consensus

- Two-phase commit, quorum reads/writes, CAP tradeoffs, Raft, leader election, and eventual consistency.

**Resources:**

- [Designing Data-Intensive Applications](https://dataintensive.net/)
- [Martin Kleppmann's distributed systems lectures](https://www.youtube.com/@kleppmann/playlists)
- [CockroachDB Architecture](https://www.cockroachlabs.com/docs/stable/architecture/overview)

## Chapter 11: NoSQL and Polyglot Persistence

### [P1] Choosing a Data Model

- Access-pattern-first design; document, key-value, wide-column, graph, and time-series models.
- Denormalization, embedded documents, secondary indexes, TTL, and consistency choices.

### [P1] Common Systems

- PostgreSQL or MySQL for relational workloads, Redis for caching and ephemeral data, MongoDB for document workloads, and Kafka for event streams.
- Understand when not to add another datastore.

**Resources:**

- [Redis University](https://university.redis.com/)
- [MongoDB University](https://learn.mongodb.com/)
- [Apache Kafka documentation](https://kafka.apache.org/documentation/)

## Chapter 12: Application and Production Patterns

### [P0] Database Access in Applications

- Connection pooling, migrations, transactions in services, prepared statements, timeouts, retries, and idempotency.

### [P1] Caching and Data Consistency

- Cache-aside, write-through, write-behind, invalidation, TTL, stampede prevention, and stale data.

### [P1] Queues, Outbox, and CDC

- Transactional outbox, change data capture, at-least-once delivery, deduplication, ordering, and exactly-once claims.

### [P1] Pagination and API Queries

- Offset versus keyset pagination, stable ordering, filtering, sorting, rate limits, and preventing unbounded queries.

## Chapter 13: Analytics and Data Warehousing

### [P1] OLAP Modeling

- Facts, dimensions, grain, star schema, snowflake schema, slowly changing dimensions, and surrogate keys.

### [P1] Warehousing and ETL/ELT

- Batch versus streaming, staging, idempotent pipelines, data quality, lineage, partitioning, columnar storage, and lakehouse concepts.

**Resources:**

- [Kimball Group resources](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/)
- [Data Engineering Zoomcamp](https://github.com/DataTalksClub/data-engineering-zoomcamp)

## Chapter 14: Database Design Interviews

### [P0] A Repeatable Interview Process

1. Clarify entities, reads, writes, scale, latency, consistency, and retention.
2. Define access patterns and choose a data model.
3. Draw tables, keys, constraints, and indexes.
4. Describe transactions and failure handling.
5. Estimate traffic, storage, and query costs.
6. Add caching, replication, partitioning, and observability only when the requirements justify them.

### [P0] Problems to Practice

- URL shortener, chat application, news feed, ride sharing, payments, inventory, ticket booking, file metadata, and metrics ingestion.

**Resource:** [System Design Primer](https://github.com/donnemartin/system-design-primer)

## Chapter 15: Role-Based Priorities

| Target role | Master first | Then learn | Demonstrate |
| --- | --- | --- | --- |
| Backend / Software Engineer | SQL, schema design, joins, transactions, indexes, APIs | Caching, replication, queues, migrations | A production-style CRUD service with measured queries |
| Data Analyst | SQL, joins, aggregation, windows, data quality | Warehousing, dimensional modeling, BI tools | A dashboard with documented metrics and assumptions |
| Data Engineer | SQL, partitioning, ETL/ELT, warehousing | Kafka, CDC, orchestration, lakehouse systems | An idempotent batch or streaming pipeline |
| Database Administrator | Transactions, backup, recovery, security, monitoring | Replication, failover, capacity planning, upgrades | A backup/restore and incident runbook |
| Database Engineer | Storage, indexes, query optimizer, concurrency, recovery | Engine internals, distributed systems, compilers | A benchmark, query-plan analysis, or storage project |
| Platform / SRE Engineer | Reliability, observability, replication, scaling | Kubernetes operators, automation, disaster recovery | An SLO-backed database service |

## Recommended Learning Resources

### Courses and Video Playlists

- [CMU 15-445 Database Systems](https://15445.courses.cs.cmu.edu/)
- [CMU Database Group YouTube](https://www.youtube.com/@CMUDatabaseGroup/playlists)
- [Stanford Databases](https://online.stanford.edu/courses/soe-ydatabases-databases)
- [freeCodeCamp SQL and database videos](https://www.youtube.com/@freecodecamp/search?query=sql%20database)

### Books

- [Database System Concepts](https://www.db-book.com/)
- [Designing Data-Intensive Applications](https://dataintensive.net/)
- [Database Internals](https://www.oreilly.com/library/view/database-internals/9781492040347/)

### Documentation, Blogs, and Practice Sites

- [PostgreSQL documentation](https://www.postgresql.org/docs/)
- [MySQL documentation](https://dev.mysql.com/doc/)
- [Use The Index, Luke!](https://use-the-index-luke.com/)
- [SQLBolt](https://sqlbolt.com/)
- [LeetCode Database](https://leetcode.com/problemset/database/)
- [DataLemur](https://datalemur.com/questions)
- [Percona Database Performance Blog](https://www.percona.com/blog/)
- [Brandur's database posts](https://brandur.org/articles)

## Capstone Project

Build a small order-management system using PostgreSQL and one application language.

- Model users, products, inventory, carts, orders, payments, and shipments.
- Add constraints, migrations, seed data, and realistic indexes.
- Implement checkout as a transaction with an idempotency key.
- Add pagination, search, caching, and an outbox event.
- Use `EXPLAIN ANALYZE` to tune five queries.
- Add a backup, restore test, monitoring dashboard, and failure runbook.
- Write an architecture document explaining consistency, scaling, security, and tradeoffs.

## Completion Checklist

- [ ] Design a normalized schema from requirements.
- [ ] Write joins, CTEs, windows, and recursive queries without copying solutions.
- [ ] Explain ACID, isolation anomalies, locks, and deadlocks.
- [ ] Read an execution plan and justify an index.
- [ ] Perform a backup and a verified restore.
- [ ] Explain replication, partitioning, caching, and sharding tradeoffs.
- [ ] Complete 75-100 SQL interview problems.
- [ ] Finish and document the capstone project.