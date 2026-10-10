# SQL: The Complete End-to-End Notes (Theory Edition)

> **Goal of this file:** be the *only* SQL reference you need. Not a handbook of syntax, but a **teacher in a file**: it explains *why* things work the way they do, so you can speak about performance, design and trade-offs like someone who has actually run these systems.
>
> **Primary dialect:** ANSI SQL + **SQL Server (T-SQL)**. Differences for **PostgreSQL**, **MySQL** and **Oracle** are flagged wherever they matter.
>
> **Version note:** byte sizes, limits and defaults below are the documented values for current mainstream versions (SQL Server 2019/2022, PostgreSQL 14+, MySQL 8). Always confirm against the official docs for your exact version (links in the last section).

## Table of Contents

- [0. Read This First](#0-read-this-first)
- [1. Database Fundamentals](#1-database-fundamentals)
- [2. SQL Categories](#2-sql-categories)
- [3. Sample Dataset](#3-sample-dataset)
- [4. How the Database Stores Data (Storage Internals)](#4-how-the-database-stores-data-storage-internals)
- [5. Data Types, Sizes and Performance](#5-data-types-sizes-and-performance)
- [6. Strings, Unicode and Collation](#6-strings-unicode-and-collation)
- [7. Numeric Precision, Scale and Arithmetic](#7-numeric-precision-scale-and-arithmetic)
- [8. Date and Time in Depth](#8-date-and-time-in-depth)
- [9. DDL: Defining Structure](#9-ddl-defining-structure)
- [10. Constraints and Keys](#10-constraints-and-keys)
- [11. Identity, Sequences and GUIDs](#11-identity-sequences-and-guids)
- [12. DML: Modifying Data](#12-dml-modifying-data)
- [13. SELECT Basics and Logical Execution Order](#13-select-basics-and-logical-execution-order)
- [14. Filtering: WHERE and Operators](#14-filtering-where-and-operators)
- [15. NULL Handling](#15-null-handling)
- [16. Sorting and Limiting](#16-sorting-and-limiting)
- [17. Aggregates, GROUP BY, HAVING](#17-aggregates-group-by-having)
- [18. Joins](#18-joins)
- [19. Set Operators](#19-set-operators)
- [20. Subqueries](#20-subqueries)
- [21. CTEs and Recursive CTEs](#21-ctes-and-recursive-ctes)
- [22. Window Functions](#22-window-functions)
- [23. Built-in Functions](#23-built-in-functions)
- [24. CASE Expressions](#24-case-expressions)
- [25. PIVOT, UNPIVOT and Conditional Aggregation](#25-pivot-unpivot-and-conditional-aggregation)
- [26. Views and Materialized Views](#26-views-and-materialized-views)
- [27. Indexes](#27-indexes)
- [28. Statistics, Execution Plans and Query Optimization](#28-statistics-execution-plans-and-query-optimization)
- [29. Stored Procedures, Functions and Triggers](#29-stored-procedures-functions-and-triggers)
- [30. Transactions, ACID, Isolation and Locking](#30-transactions-acid-isolation-and-locking)
- [31. Transaction Log, Recovery and Backups](#31-transaction-log-recovery-and-backups)
- [32. Normalization and Denormalization](#32-normalization-and-denormalization)
- [33. Data Modeling and Warehousing](#33-data-modeling-and-warehousing)
- [34. Temp Tables, Table Variables and Dynamic SQL](#34-temp-tables-table-variables-and-dynamic-sql)
- [35. JSON and XML in SQL](#35-json-and-xml-in-sql)
- [36. Partitioning](#36-partitioning)
- [37. Security (DCL)](#37-security-dcl)
- [38. Error Handling](#38-error-handling)
- [39. Advanced Analytical Patterns](#39-advanced-analytical-patterns)
- [40. Subscription and Churn Analytics Patterns](#40-subscription-and-churn-analytics-patterns)
- [41. Dialect Differences Cheat Sheet](#41-dialect-differences-cheat-sheet)
- [42. Best Practices and Anti-Patterns](#42-best-practices-and-anti-patterns)
- [43. Interview Q&A](#43-interview-qa)
- [44. One-Page Cheat Sheet](#44-one-page-cheat-sheet)
- [45. Where to Learn More, and a Study Roadmap](#45-where-to-learn-more-and-a-study-roadmap)


---

## 0. Read This First

### 0.1 Legend used throughout

| Marker | Meaning |
|--------|---------|
| 🧠 | **Theory**: the "why" in plain language |
| ⚠️ | **Gotcha**: a mistake people make in real projects and interviews |
| ⚡ | **Performance** impact |
| 🔀 | **Dialect** difference (SQL Server vs PostgreSQL vs MySQL vs Oracle) |
| 🎯 | **Interview line**: a sentence you can say out loud |

### 0.2 The three questions behind every SQL topic

When you study any feature, ask:

1. **What problem does it solve?** (so you know when to use it)
2. **What does the engine physically do?** (so you can reason about speed)
3. **How does it break?** (NULLs, duplicates, ties, time zones, overflow, locking)

If you can answer all three for a topic, you *know* the topic. Most "SQL knowledge" that fails in interviews is topic knowledge with only answer 1.

### 0.3 The mindset shift: SQL is declarative

In Python or Java you tell the computer **how** to get a result, step by step. In SQL you describe **what** result you want, and the **optimizer** (a cost-based planner inside the database) decides how: which index, which join algorithm, which order. That is why two queries that look completely different can run identically, and two queries that look almost the same can differ by 1000x.

🎯 *"SQL is declarative: I describe the result, the optimizer chooses the plan based on statistics and available indexes. My job is to write queries and design schemas that give it good options."*

---

## 1. Database Fundamentals

### 1.1 Data, Database, DBMS, RDBMS, SQL

| Term | Meaning |
|------|---------|
| **Data** | Raw facts: `Ravi`, `25`, `2024-03-01` |
| **Database** | An organized collection of related data (the files on disk plus the structure around them) |
| **DBMS** | Software that creates, stores, secures and queries a database (handles concurrency, crashes, permissions) |
| **RDBMS** | A DBMS built on the **relational model**: data lives in **tables (relations)** linked through keys. SQL Server, PostgreSQL, MySQL, Oracle |
| **SQL** | **Structured Query Language**, the standard language for talking to an RDBMS. Pronounced "S-Q-L" or "sequel" |

🧠 A DBMS is not just "a place to put data". The reason we do not store everything in Excel or text files is what a DBMS gives us: **integrity** (constraints), **concurrency** (many users at once without corrupting data), **durability** (survives a crash), **security** (who can see what) and **performance** (indexes and optimizer).

### 1.2 What actually happens when you press Execute

```text
 Your query text
      │
      ▼
 1. PARSER        → checks syntax ("is this valid SQL?")
      ▼
 2. BINDER        → resolves names (does table/column exist? do I have permission? what are the data types?)
      ▼
 3. OPTIMIZER     → looks in the PLAN CACHE first; if no plan, generates many candidate plans,
                    estimates the COST of each using STATISTICS, picks the cheapest
      ▼
 4. EXECUTOR      → runs the plan operator by operator (scan, seek, join, sort, aggregate…)
      ▼
 5. STORAGE ENGINE→ reads 8 KB pages from the BUFFER POOL (memory) or disk, takes locks,
                    writes changes to the TRANSACTION LOG first (write-ahead logging)
      ▼
 Result set returned to the client
```

Things worth remembering from this pipeline:

- Errors come from different stages: a typo like `SELEC` is a **parse** error; a wrong column name is a **bind** error; divide-by-zero or conversion failure is a **runtime** error. This is why a query can "compile" and still fail halfway.
- The optimizer is **cost-based**: it does not know the true row counts; it *estimates* them from **statistics**. Bad estimates → bad plans (see §28).
- **Plan cache**: compiling a plan is expensive, so it is cached and reused. This is why parameterized queries are fast and why **parameter sniffing** exists.
- Databases are **I/O-bound** more than CPU-bound. Reading a page from memory takes microseconds; from disk, milliseconds. Almost every performance trick in this file is a way to **touch fewer pages**.

### 1.3 The relational model in plain words

- A **relation** (table) is a **set of tuples** (rows) sharing the same **attributes** (columns).
- Each attribute has a **domain** (data type + constraints): what values are legal.
- In *theory*, a relation is a **set**: no duplicates and **no inherent order**. In *practice*, SQL tables are **bags** (duplicates allowed unless a key prevents them), which is why we add primary keys.
- ⚠️ **Rows have no guaranteed order.** If you do not write `ORDER BY`, the order you see is whatever was convenient for the engine today and can change tomorrow (after an index is added, a parallel plan is used, or data is rebuilt). Never rely on "it always comes back sorted".

### 1.4 Core terminology

| Term | Also called | Meaning |
|------|-------------|---------|
| Table | Relation | Rows and columns |
| Row | Tuple, record | One entry |
| Column | Attribute, field | One property of every row |
| Cardinality | | Number of **rows** (also used for "how many distinct values", see statistics) |
| Degree | | Number of **columns** |
| Schema | | A namespace/container for objects (tables, views, procedures) and a security boundary |
| Primary key | PK | Uniquely identifies a row; NOT NULL |
| Foreign key | FK | Column(s) referencing a key in another table; enforces referential integrity |
| Index | | Separate sorted structure for fast lookup |
| Constraint | | A rule the database enforces on the data |
| Transaction | | A unit of work that is all-or-nothing |

### 1.5 Object hierarchy and naming (where things live)

**SQL Server:** `Server instance → Database → Schema → Object`. A fully qualified name is `server.database.schema.object` (four-part names are for linked servers), usually written as `database.schema.object` such as `company_db.dbo.employees`.

- Default schema is `dbo`. Always **qualify** objects with the schema (`dbo.employees`): it avoids ambiguity and improves plan-cache reuse.
- **System databases:** `master` (server-level metadata, logins), `model` (template for new databases), `msdb` (SQL Agent jobs, backup history), `tempdb` (temporary objects, sorts, spills, row versions; **recreated on every restart**), plus the hidden `resource` database.

🔀 **PostgreSQL:** `Cluster → Database → Schema → Object` (default schema `public`). Cannot query across databases directly.
🔀 **MySQL:** *schema and database are the same thing.*
🔀 **Oracle:** a **schema is a user**: each user owns a schema with the same name.

### 1.6 Relationships

| Type | Example | How to implement |
|------|---------|------------------|
| One-to-One | User ↔ Passport | FK that is also `UNIQUE` (or shared PK) |
| One-to-Many | Customer → Orders | FK on the **many** side |
| Many-to-Many | Students ↔ Courses | **Junction (bridge) table** with two FKs; PK is often the pair |

🧠 You can never store many-to-many directly in two tables, because a cell would need to hold a list (violating 1NF). The bridge table turns it into two one-to-many relationships.

### 1.7 SQL vs NoSQL

| | SQL (RDBMS) | NoSQL |
|--|-------------|-------|
| Schema | Fixed, enforced on write ("schema-on-write") | Flexible, often interpreted on read |
| Scaling | Vertical first; horizontal is harder | Designed for horizontal scale |
| Consistency | Strong ACID | Often eventual consistency (BASE) |
| Joins | First-class | Usually avoided or manual |
| Best for | Structured data, relationships, transactions, analytics | Huge scale, flexible/nested data, simple access patterns |

🎯 *"It is not SQL versus NoSQL; it is access patterns and consistency needs. If I need multi-row transactions and ad-hoc analytical queries, a relational database is the right default."*

---

## 2. SQL Categories

| Category | Full form | Commands | Purpose |
|----------|-----------|----------|---------|
| **DDL** | Data Definition Language | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME` | Define/change **structure** |
| **DML** | Data Manipulation Language | `INSERT`, `UPDATE`, `DELETE`, `MERGE` | Change **data** |
| **DQL** | Data Query Language | `SELECT` | Read data |
| **DCL** | Data Control Language | `GRANT`, `REVOKE`, `DENY` (T-SQL) | **Permissions** |
| **TCL** | Transaction Control Language | `BEGIN TRAN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Control transactions |

🧠 `SELECT` is sometimes counted under DML. Both views are accepted; say which one you are using.

### 2.1 DELETE vs TRUNCATE vs DROP

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| Category | DML | DDL | DDL |
| Removes | Chosen rows (or all) | **All** rows | The whole object (data + structure + indexes + permissions) |
| `WHERE` | ✅ | ❌ | ❌ |
| Logging | Every row logged | Only page/extent deallocations logged (minimal) | Metadata |
| Identity counter | Keeps going | **Reset** to seed | Gone |
| Triggers | Fire | Do **not** fire | Do not fire |
| Blocked by FK reference | Only if child rows exist | **Yes** (cannot truncate a table referenced by an FK, even if empty) | Must drop FK first |
| Rollback | ✅ | ✅ in SQL Server and PostgreSQL (inside a transaction); ❌ MySQL/Oracle (implicit commit) | Depends on engine |
| Permission needed | `DELETE` | `ALTER` on table | `ALTER` on schema / owner |
| Speed on big tables | Slow (row by row, log growth) | Very fast | Very fast |

🧠 TRUNCATE is fast not because it "skips the log" (a common myth) but because it logs only *"these pages are now free"* instead of *"this row was deleted"* a million times.

⚠️ Deleting 100 million rows with one `DELETE` can fill the transaction log, escalate to a table lock and run for hours. Delete in **batches** (§12.4) or partition and switch out.

---

## 3. Sample Dataset

Every example uses this dataset. Run it once to practice.

```sql
CREATE TABLE departments (
    dept_id     INT PRIMARY KEY,
    dept_name   VARCHAR(50) NOT NULL,
    location    VARCHAR(50)
);

CREATE TABLE employees (
    emp_id      INT PRIMARY KEY,
    emp_name    VARCHAR(100) NOT NULL,
    dept_id     INT,
    manager_id  INT,
    salary      DECIMAL(10,2),
    hire_date   DATE,
    email       VARCHAR(100),
    FOREIGN KEY (dept_id)    REFERENCES departments(dept_id),
    FOREIGN KEY (manager_id) REFERENCES employees(emp_id)
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    cust_name   VARCHAR(100),
    city        VARCHAR(50),
    signup_date DATE
);

CREATE TABLE orders (
    order_id    INT PRIMARY KEY,
    customer_id INT,
    order_date  DATE,
    amount      DECIMAL(10,2),
    status      VARCHAR(20),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

INSERT INTO departments VALUES (1,'HR','Hyderabad'),(2,'IT','Bangalore'),(3,'Finance','Mumbai'),(4,'Marketing',NULL);

INSERT INTO employees VALUES
(1,'Asha',2,NULL,150000,'2018-01-15','asha@co.com'),
(2,'Ravi',2,1,90000,'2019-03-10','ravi@co.com'),
(3,'Meera',1,1,70000,'2020-07-01',NULL),
(4,'Kiran',3,1,85000,'2021-11-20','kiran@co.com'),
(5,'Sai',2,2,60000,'2022-02-14','sai@co.com'),
(6,'Divya',NULL,2,55000,'2023-05-05','divya@co.com');

INSERT INTO customers VALUES (1,'Arjun','Hyderabad','2023-01-10'),(2,'Priya','Mumbai','2023-02-15'),(3,'Rahul','Delhi','2023-03-20'),(4,'Sneha','Hyderabad','2023-04-25');

INSERT INTO orders VALUES
(101,1,'2024-01-05',2500,'Delivered'),
(102,1,'2024-02-11',1800,'Delivered'),
(103,2,'2024-02-14',3200,'Cancelled'),
(104,3,'2024-03-01',1500,'Delivered'),
(105,1,'2024-03-18',4100,'Pending');
```

Things deliberately built into this data so you can see edge cases: Divya has **no department** (NULL FK), Meera has **no email** (NULL), Marketing has **no employees** (anti-join), Asha has **no manager** (top of the hierarchy), customer Sneha has **no orders**.

---

## 4. How the Database Stores Data (Storage Internals)

This section is the foundation for *why data type size matters*. Spend time here; it makes the rest click.

### 4.1 Files

A SQL Server database is stored in at least two kinds of files:

- **Data file(s)** `.mdf` (primary) / `.ndf` (secondary): the tables and indexes.
- **Transaction log** `.ldf`: a sequential record of every change, used for rollback, crash recovery and backups.

### 4.2 Pages and extents

- The **page is the fundamental unit of I/O**: **8 KB = 8,192 bytes**.
  - 96 bytes page header.
  - Up to **8,060 bytes** of data for a single in-row record.
  - A **row-offset array** at the end of the page costs **2 bytes per row** (the "slot").
  - Effective usable space for rows is about **8,096 bytes** per page.
- An **extent** is **8 contiguous pages = 64 KB**. Space is allocated to tables/indexes in extents (uniform extents by default in modern versions).
- The engine **never reads a single row from disk**; it reads the whole 8 KB page containing the row into memory (the **buffer pool**).

🧠 Think of the page as a physical page of a notebook and a row as one line on it. To read one line you must pick up the whole page. If lines are short, one page holds many lines and you pick up fewer pages for the same data.

### 4.3 Why smaller rows are faster

```text
Rows per page ≈ 8096 / (row_size + 2)
```

| Row size | Rows per page | Pages for 10 million rows | Approx. size |
|----------|---------------|---------------------------|--------------|
| 100 bytes | 79 | ~126,600 | ~990 MB |
| 200 bytes | 40 | ~250,000 | ~1.9 GB |

Double the row width and you roughly double the pages: twice the disk, twice the memory in the buffer pool, twice the I/O for every scan, twice the backup time. ⚡ This is the real reason we choose the **smallest data type that safely fits**.

### 4.4 Row anatomy (SQL Server, in-row, simplified)

| Part | Size | Notes |
|------|------|-------|
| Row header | **4 bytes** | status bits + pointer to null bitmap |
| Fixed-length columns | sum of their sizes | `INT`, `DATE`, `DECIMAL`, `CHAR(n)`... stored first |
| Null bitmap | **2 bytes + ceil(columns/8)** | one bit per column; this is why NULL costs almost nothing in fixed columns |
| Variable-length column count + offset array | **2 bytes + 2 bytes per variable column** | this is the overhead of `VARCHAR`/`NVARCHAR` |
| Variable-length data | actual bytes | stored last |
| (Optional) versioning tag | 14 bytes | added when row versioning is on (RCSI/SNAPSHOT) |

**Worked example: the `orders` table** with `order_id INT, customer_id INT, order_date DATE, amount DECIMAL(10,2), status VARCHAR(20)`, where status is `'Delivered'` (9 chars):

```text
Fixed data     = 4 + 4 + 3 + 9                = 20 bytes     (DECIMAL(10,2) = 9 bytes)
Variable data  = 2 (count) + 1×2 (offset) + 9 = 13 bytes
Null bitmap    = 2 + ceil(5/8) = 2 + 1        =  3 bytes
Row header                                    =  4 bytes
                                       ROW SIZE = 40 bytes      → 8096/(40+2) = 192 rows/page
```

Now replace `status VARCHAR(20)` with a `TINYINT status_id` pointing to a small lookup table:

```text
Fixed data = 4+4+3+9+1 = 21 | variable = 0 | null bitmap = 3 | header = 4  → 28 bytes → 8096/30 = 269 rows/page
```

| | VARCHAR(20) status | TINYINT status_id | Saving |
|--|--|--|--|
| Row size | 40 B | 28 B | 30% |
| Rows/page | 192 | 269 | +40% |
| 100M rows | ≈ 4.0 GB | ≈ 2.8 GB | **≈ 1.2 GB** (and the same saving in every index that includes the column) |

This is the kind of arithmetic that makes your performance answers credible. (Figures ignore fill factor and index overhead; the principle is what matters.)

### 4.5 Heap vs clustered table

- **Heap**: a table with **no clustered index**. Rows are stored in no particular order, found via a Row ID (file:page:slot).
- **Clustered table**: rows are physically stored **in the order of the clustered index key**; the leaf level of the index *is* the data.

### 4.6 Large values

- In-row limit is 8,060 bytes. Variable-length columns that would exceed it can be moved to **ROW_OVERFLOW** pages; `VARCHAR(MAX)`, `NVARCHAR(MAX)`, `VARBINARY(MAX)`, `XML` can be pushed to **LOB** pages (up to 2 GB). Reading such a column costs extra page reads (a pointer chase).
- Avoid `SELECT *` on tables with big text/JSON/blob columns: you pay for pages you do not need.

### 4.7 The buffer pool, logical vs physical reads

- **Physical read**: page fetched from disk into memory (slow).
- **Logical read**: page read from memory (fast, but still costs CPU and latches).
- `SET STATISTICS IO ON` shows *logical reads*, the best single number for comparing query versions. Fewer logical reads ≈ less work.

### 4.8 Other engines (same ideas, different numbers)

| | Page size | Notes |
|--|-----------|-------|
| SQL Server | 8 KB | described above |
| **PostgreSQL** | 8 KB | Heap files; each row ("tuple") has a **23-byte header** (padded to 24) + a 4-byte line pointer; values over ~2 KB are compressed/moved to **TOAST** tables; **column order affects padding** (put wide fixed types first to save bytes); updates create new tuple versions (MVCC) → dead tuples cleaned by **VACUUM** |
| **MySQL InnoDB** | **16 KB** | The table **is** the clustered index on the PK, and **every secondary index stores the PK value** as its row locator, so a fat PK (e.g., a long VARCHAR or UUID string) bloats *every* index |
| Oracle | 8 KB default (block) | Heap-organized tables by default; index-organized tables are an option |

---

## 5. Data Types, Sizes and Performance

> 🧠 **Why this section matters:** a data type decides **(a)** which values are legal, **(b)** how many **bytes** every value costs on disk, in memory and in every index, **(c)** how fast comparisons and sorts are, and **(d)** how implicit conversions behave. Picking `BIGINT` where `INT` fits, or `NVARCHAR(4000)` where `CHAR(2)` fits, is a quiet tax paid on every query forever.

### 5.1 SQL Server data types, with storage

#### Exact numerics

| Type | Storage | Range / notes |
|------|---------|---------------|
| `BIT` | **1 bit** (up to 8 BIT columns share 1 byte; 9–16 share 2 bytes...) | 0, 1, NULL |
| `TINYINT` | **1 byte** | 0 to 255 (**unsigned**) |
| `SMALLINT` | **2 bytes** | −32,768 to 32,767 |
| `INT` | **4 bytes** | −2,147,483,648 to 2,147,483,647 (about ±2.1 billion) |
| `BIGINT` | **8 bytes** | −2⁶³ to 2⁶³−1 (about ±9.2 × 10¹⁸) |
| `DECIMAL(p,s)` / `NUMERIC(p,s)` | **5 bytes** for p 1–9<br>**9 bytes** for p 10–19<br>**13 bytes** for p 20–28<br>**17 bytes** for p 29–38 | p up to **38**; default `DECIMAL` = (18,0). Exact. Use for money and anything that must add up |
| `MONEY` | **8 bytes** | 4 decimal places fixed; known for rounding surprises in division/multiplication |
| `SMALLMONEY` | **4 bytes** | ±214,748.3647 |

#### Approximate numerics

| Type | Storage | Notes |
|------|---------|-------|
| `REAL` | **4 bytes** | = `FLOAT(24)`, ~7 digits precision |
| `FLOAT(n)` | **4 bytes** if n = 1–24, **8 bytes** if n = 25–53 | `FLOAT` alone = `FLOAT(53)` ~15 digits. **Binary** floating-point: cannot represent 0.1 exactly. Never for money |

#### Character strings

| Type | Storage | Max | Notes |
|------|---------|-----|-------|
| `CHAR(n)` | **n bytes** always (padded with spaces) | 8,000 | Fixed width |
| `VARCHAR(n)` | **actual bytes + 2** | 8,000 | One byte per char in single-byte code pages (with `_UTF8` collations in 2019+, 1–4 bytes per char) |
| `VARCHAR(MAX)` | actual + 2 | 2 GB | Large values go off-row (LOB) |
| `NCHAR(n)` | **2 × n bytes** | 4,000 chars | Unicode, UTF-16, fixed |
| `NVARCHAR(n)` | **2 × actual chars + 2** | 4,000 chars | Unicode, variable |
| `NVARCHAR(MAX)` | 2 × actual chars + 2 | 2 GB (about 1 billion characters) | Large values go off-row (LOB) |
| `TEXT`, `NTEXT`, `IMAGE` | | | **Deprecated.** Use the `MAX` types |

#### Date and time

| Type | Storage | Range | Accuracy |
|------|---------|-------|----------|
| `DATE` | **3 bytes** | 0001-01-01 to 9999-12-31 | 1 day |
| `TIME(p)` | **3 bytes** (p 0–2), **4 bytes** (p 3–4), **5 bytes** (p 5–7) | 00:00:00 to 23:59:59.9999999 | up to 100 ns |
| `SMALLDATETIME` | **4 bytes** | 1900-01-01 to 2079-06-06 | 1 minute |
| `DATETIME` | **8 bytes** | **1753-01-01** to 9999-12-31 | ~3.33 ms (rounds to .000, .003, .007) |
| `DATETIME2(p)` | **6 bytes** (p 0–2), **7 bytes** (p 3–4), **8 bytes** (p 5–7) | 0001-01-01 to 9999-12-31 | up to 100 ns; default p = 7 |
| `DATETIMEOFFSET(p)` | **8 / 9 / 10 bytes** (same p brackets) | as DATETIME2 | stores the **UTC offset** (+05:30), not a named time zone |

#### Binary, identifiers, other

| Type | Storage | Notes |
|------|---------|-------|
| `BINARY(n)` / `VARBINARY(n)` | n bytes / actual + 2 | max 8,000; `VARBINARY(MAX)` up to 2 GB |
| `UNIQUEIDENTIFIER` | **16 bytes** | GUID/UUID; shown as 36-char text |
| `ROWVERSION` (`TIMESTAMP`) | **8 bytes** | Auto-incrementing binary counter per database for optimistic concurrency. **Not** a date/time despite the old name |
| `XML` | up to 2 GB | |
| `SQL_VARIANT` | up to 8,016 bytes | Avoid |
| `HIERARCHYID`, `GEOGRAPHY`, `GEOMETRY` | CLR types | Specialized |

### 5.2 PostgreSQL storage

| Type | Storage | Notes |
|------|---------|-------|
| `SMALLINT` / `INTEGER` / `BIGINT` | 2 / 4 / 8 bytes | No unsigned types |
| `SERIAL` / `BIGSERIAL` | 4 / 8 bytes | Pseudo-types: integer + sequence + default |
| `NUMERIC(p,s)` | **variable**: ~2 bytes per 4 decimal digits + 3–8 bytes overhead | Up to 131,072 digits before the point and 16,383 after |
| `REAL` / `DOUBLE PRECISION` | 4 / 8 bytes | Binary floating point |
| `MONEY` | 8 bytes | Locale-dependent; generally avoid, use `NUMERIC` |
| `CHAR(n)` | n chars padded (+ header) | Padded, no performance benefit over VARCHAR in PG |
| `VARCHAR(n)` / `TEXT` | **1-byte header + data** if ≤ 126 bytes, **4-byte header + data** beyond; up to **1 GB**; compressed/TOASTed when large | **`TEXT` and `VARCHAR` perform identically**; the `(n)` is just a check |
| `BOOLEAN` | 1 byte | true/false/NULL |
| `DATE` | 4 bytes | |
| `TIME` / `TIMETZ` | 8 / 12 bytes | |
| `TIMESTAMP` / `TIMESTAMPTZ` | 8 bytes each | `TIMESTAMPTZ` stores **UTC** and converts to the session time zone on display. Recommended default for event times |
| `INTERVAL` | 16 bytes | |
| `UUID` | 16 bytes | |
| `BYTEA` | 1 or 4 bytes + data | up to 1 GB |
| `JSONB` | variable | Binary, indexable (GIN); prefer over `JSON` |

### 5.3 MySQL (InnoDB) storage

| Type | Storage | Notes |
|------|---------|-------|
| `TINYINT` / `SMALLINT` / `MEDIUMINT` / `INT` / `BIGINT` | **1 / 2 / 3 / 4 / 8 bytes** | Support `UNSIGNED` (doubles the positive range: INT UNSIGNED up to 4,294,967,295) |
| `INT(11)` | still 4 bytes | ⚠️ The number is only a *display width* (and deprecated since 8.0.17); it does **not** limit values |
| `DECIMAL(p,s)` | packed: **4 bytes per 9 digits**, leftovers 1–4 bytes | Max precision 65, scale 30 |
| `FLOAT` / `DOUBLE` | 4 / 8 bytes | |
| `CHAR(n)` | up to 255 chars | |
| `VARCHAR(n)` | actual + **1 byte** (if ≤ 255 bytes) or **2 bytes** | Whole **row** limit is 65,535 bytes shared across columns; `utf8mb4` can use up to **4 bytes per character** |
| `TEXT` / `MEDIUMTEXT` / `LONGTEXT` | up to 64 KB / 16 MB / 4 GB | Stored off-page when large |
| `DATE` | **3 bytes** | 1000-01-01 to 9999-12-31 |
| `DATETIME` | **5 bytes** (+0–3 for fractional seconds) | 1000-01-01 to 9999-12-31, **no time zone conversion** |
| `TIMESTAMP` | **4 bytes** (+0–3) | Stored as UTC, converted to session zone; range **1970-01-01 to 2038-01-19** (the "Year 2038 problem") |
| `YEAR` | 1 byte | |
| `BOOLEAN` | alias of `TINYINT(1)` | |
| `ENUM` / `SET` | 1–2 bytes / 1–8 bytes | Stored as integers; adding values later can need a table rebuild |
| `BINARY(16)` | 16 bytes | Best way to store UUIDs (not `CHAR(36)`, which is 36+ bytes) |

### 5.4 Oracle (quick facts)

- `NUMBER(p,s)`: **variable 1–22 bytes**; used for all numeric needs. There is no native `INT`; it maps to `NUMBER(38)`.
- `VARCHAR2(n)`: up to 4,000 bytes (32,767 with extended mode). Prefer `VARCHAR2` over `VARCHAR`.
- `DATE`: **7 bytes and includes a time part** (to the second), a frequent surprise. `TIMESTAMP`: 7–11 bytes.
- `CLOB`/`BLOB` for large objects. Booleans: `NUMBER(1)` (native `BOOLEAN` only from 23c).
- ⚠️ In Oracle, an **empty string `''` is NULL**.

### 5.5 How to choose: the decision rules

| Need | Choose | Why |
|------|--------|-----|
| Yes/No flag | `BIT` (SQL Server), `BOOLEAN` (PG/MySQL) | 1 bit vs 1–4 bytes |
| Small code/status (≤ 255 values) | `TINYINT` + lookup table | 1 byte vs `VARCHAR(20)` ≈ 10–20 bytes, and it is FK-checked, so no typos like 'Delvered' |
| Row id for a table that will stay under ~2 billion rows | `INT` | 4 bytes; half the key size of BIGINT, in the table **and** in every index |
| Row id that could pass 2.1 billion (events, logs, telemetry) | `BIGINT` | Migrating INT → BIGINT on a huge table later is painful and risky |
| Money, prices, rates, balances | `DECIMAL(19,4)` (9 bytes) or `DECIMAL(p,2)` | Exact arithmetic |
| Measurements where tiny error is fine (sensors, ML scores) | `FLOAT`/`REAL` | Fast, compact |
| Date only (birth date, order date) | `DATE` (3 bytes) | `DATETIME` wastes 5 bytes per value and invites time-part bugs |
| Event timestamp | `DATETIME2(0..3)` in **UTC** (6–7 bytes) or `DATETIMEOFFSET` | Smaller than default precision-7 if you do not need nanoseconds |
| Fixed-length code (ISO country `IN`, currency `INR`, PIN code, Aadhaar 12 digits) | `CHAR(n)` | No 2-byte variable-length overhead |
| Names, emails, free text | `VARCHAR(n)` (or `NVARCHAR(n)` if multi-language) | Pay for what you use |
| Telugu/Hindi/Tamil/Arabic/emoji text | `NVARCHAR` (or UTF-8 `VARCHAR` in SQL Server 2019+) | Needs Unicode |
| Phone numbers, PIN codes, account numbers | **String, never INT** | Leading zeros, `+91`, you never do arithmetic on them |
| UUID/GUID | `UNIQUEIDENTIFIER` / `UUID` / `BINARY(16)` | Never `VARCHAR(36)` (36+ bytes and slower compares) |

### 5.6 What data types do to performance (speak about this in interviews)

1. **Density → I/O.** Smaller rows = more rows per 8 KB page = fewer page reads for scans and range queries (§4.3).
2. **Memory.** The buffer pool caches pages. Narrow tables mean more of your hot data fits in RAM.
3. **Index size multiplies the effect.** In SQL Server, every non-clustered index stores the **clustered key** as its row locator. A 16-byte GUID clustered key adds 16 bytes to *every row of every non-clustered index*; a 4-byte INT adds 4. In MySQL InnoDB the same is true for the PK.
4. **Index key limits.** SQL Server: clustered index key ≤ **900 bytes**, non-clustered ≤ **1,700 bytes** (2016+). Wide `NVARCHAR(500)` keys quickly hit this.
5. **Comparisons and sorts.** Integer compares are one CPU instruction; string compares go through **collation rules** and are far slower. Joining on `INT` beats joining on `VARCHAR`.
6. **Memory grants.** For variable-length columns, SQL Server *assumes they are about half full* when reserving memory for sorts/hashes. Declaring `VARCHAR(8000)` "just in case" makes the optimizer reserve far more memory than needed, causing **excessive memory grants** and concurrency problems. Declare realistic lengths.
7. **Implicit conversion.** When two sides of a comparison have different types, SQL Server converts the one with **lower data type precedence** to the higher one. If the *column* gets converted, the index often cannot be used for a seek (see §5.7).
8. **Random keys fragment indexes.** A random GUID (`NEWID()`) as a clustered key causes inserts to land all over the index, forcing **page splits** (a full page is split in two, leaving ~50% empty pages), fragmentation, and extra log. `NEWSEQUENTIALID()` (or UUIDv7, ULID in app code) inserts at the end like an identity.
9. **Over-wide types hurt you twice.** `BIGINT` instead of `INT` for 100M rows is 400 MB extra per column per index copy.

### 5.7 Implicit conversion and data type precedence

SQL Server (simplified order, highest wins): `datetime2/datetime/date > float > decimal > money > bigint > int > smallint > tinyint > bit > nvarchar > varchar > char > binary`...

```sql
-- email is VARCHAR(100) and indexed
WHERE email = N'asha@co.com'     -- N'' literal is NVARCHAR (higher precedence)
-- → the COLUMN is implicitly converted to NVARCHAR for every row → index seek becomes scan (esp. with SQL collations)

WHERE email = 'asha@co.com'      -- matches column type → seek ✅
WHERE phone = 9876543210         -- phone is VARCHAR, literal is INT → column converted to INT → scan, and may even throw a conversion error
WHERE phone = '9876543210'       -- ✅
```

How to spot it: the execution plan shows a **warning** (`CONVERT_IMPLICIT`) on the operator. Fix by matching types (also match parameter types in your application code: ORMs that send everything as `NVARCHAR` cause this often).

### 5.8 Quick sizing exercise (practice this)

> "A table has 50 million rows with columns `id BIGINT`, `status NVARCHAR(20)` (avg 8 chars), `created DATETIME`, `amount FLOAT`. Suggest improvements."

Reasonable answer: `id` → `INT` if <2.1B rows (saves 4 B); `status` → `TINYINT` + lookup (from 2×8+2 = 18 B to 1 B); `created` → `DATETIME2(0)` (6 B vs 8 B) or `DATE` if time is not needed (3 B); `amount` → `DECIMAL(12,2)` (9 B) for exactness. Saved roughly **4 + 17 + 2 − 1 = 22 bytes/row ≈ 1.1 GB** on 50M rows, again in every index containing those columns, and `FLOAT` was a correctness bug for money anyway.

---

## 6. Strings, Unicode and Collation

### 6.1 CHAR vs VARCHAR vs NVARCHAR in practice

| | `CHAR(n)` | `VARCHAR(n)` | `NVARCHAR(n)` |
|--|-----------|--------------|---------------|
| Length | Fixed n; pads with spaces | Variable up to n | Variable up to n chars |
| Bytes | n | actual + 2 | 2×actual + 2 |
| Languages | Code page of the collation | same | **All** Unicode |
| Best for | Codes of constant length | Most text | Multilingual text |

⚠️ The `n` in `VARCHAR(n)` does **not** reserve space. `VARCHAR(100)` holding "Ravi" uses 4 + 2 bytes.
⚠️ `n` in `NVARCHAR(n)` is **characters** (2-byte units), not bytes. Characters outside the BMP (some emoji, rare scripts) take **2 UTF-16 code units (4 bytes)**, so `LEN` can differ from what a human counts; use `_SC` (supplementary character) collations when it matters.

### 6.2 The `N` prefix

```sql
INSERT INTO t(name) VALUES ('తెలుగు');    -- into NVARCHAR: the literal is first treated as VARCHAR → becomes '??????' (data loss)
INSERT INTO t(name) VALUES (N'తెలుగు');   -- ✅ Unicode literal
```

### 6.3 Collation: the rulebook for comparing and sorting text

A **collation** defines (a) the **code page** for `VARCHAR`, (b) **sort order** and (c) **comparison rules**. Name anatomy: `SQL_Latin1_General_CP1_CI_AS`

| Part | Meaning |
|------|---------|
| `Latin1_General` / `SQL_Latin1_General` | Alphabet and sorting rules (`SQL_` prefix = legacy SQL-collation; `Latin1_General_100_CI_AS` = Windows collation, preferred) |
| `CI` / `CS` | Case-**in**sensitive / Case-sensitive (`'a' = 'A'` true/false) |
| `AI` / `AS` | Accent-insensitive / sensitive (`'e' = 'é'`) |
| `KS`, `WS` | Kana / width sensitive |
| `_UTF8` | UTF-8 encoding for VARCHAR (2019+) |
| `BIN`, `BIN2` | Binary comparison: fastest, no linguistic rules |

Collation can be set at **server, database, column and expression** level, and it affects: `=`, `LIKE`, `ORDER BY`, `GROUP BY`, `DISTINCT`, `UNIQUE` constraints, `JOIN`s.

```sql
SELECT * FROM employees WHERE emp_name = 'ravi' COLLATE Latin1_General_CS_AS;   -- case-sensitive just here
```

⚠️ **Collation conflict** ("Cannot resolve the collation conflict between X and Y") happens when comparing columns with different collations, classically between a user table and a **temp table** (which inherits `tempdb`'s collation). Fix with `COLLATE DATABASE_DEFAULT` in the temp table definition.
⚡ Applying `COLLATE` to a column in a `WHERE` makes the predicate non-SARGable.

🔀 **Case sensitivity by engine:** SQL Server and MySQL → default **case-insensitive**; PostgreSQL and Oracle → **case-sensitive** (use `ILIKE` / `LOWER()` / `citext`).

### 6.4 Trailing spaces and length functions

- SQL Server follows ANSI padding for `=`: `'abc' = 'abc   '` is **TRUE**; but `LIKE 'abc'` does **not** ignore trailing spaces.
- `LEN('abc   ')` = **3** (ignores trailing spaces); `DATALENGTH('abc   ')` = **6** (bytes; for NVARCHAR double that).
- 🔀 MySQL: collations are `PAD SPACE` (ignore trailing spaces in comparison) except the `utf8mb4_0900_*` default collations in 8.0 which are `NO PAD`.
- 🔀 MySQL `utf8` is a **3-byte** legacy alias that cannot store emoji. Always use `utf8mb4`.
- 🔀 PostgreSQL `char(n)` pads and ignores trailing spaces in comparison; `varchar`/`text` do not.

### 6.5 String pitfalls

- Concatenation with NULL: `'a' + NULL` → NULL in T-SQL (`CONCAT` treats NULL as empty). 🔀 In Oracle `||` treats NULL as empty; in PostgreSQL `||` returns NULL.
- Strings that look like numbers sort alphabetically: `'10' < '9'`. Cast to a number for ordering.
- `ISNUMERIC()` is unreliable ('$', ',', '.', 'e' pass as numeric). Use `TRY_CAST(x AS INT) IS NOT NULL`.
- Leading wildcard in `LIKE '%x'` kills index seeks (§14.2).
- Never store lists in one column (`'1,2,3'`): violates 1NF and is unindexable.

---

## 7. Numeric Precision, Scale and Arithmetic

### 7.1 Precision and scale

`DECIMAL(p, s)`: **p** = total digits, **s** = digits to the right of the decimal point. Integer digits allowed = p − s.

| Declaration | Largest value | Storage (SQL Server) |
|-------------|---------------|----------------------|
| `DECIMAL(5,2)` | 999.99 | 5 B |
| `DECIMAL(10,2)` | 99,999,999.99 | 9 B |
| `DECIMAL(19,4)` | 999,999,999,999,999.9999 | 9 B |
| `DECIMAL(38,10)` | 28 integer digits | 17 B |

⚠️ Storing `1234.567` into `DECIMAL(5,2)` → rounds to 1234.57 which needs 6 digits → **arithmetic overflow error**.

### 7.2 Exact vs approximate

```sql
SELECT 0.1 + 0.2;                         -- DECIMAL literals → 0.3 exactly
SELECT CAST(0.1 AS FLOAT) + CAST(0.2 AS FLOAT);   -- 0.30000000000000004 (binary approximation)
```
FLOAT stores a binary fraction; 0.1 has no exact binary form, just as 1/3 has no exact decimal form. Summing money in FLOAT drifts by tiny amounts and reconciliations stop matching.

### 7.3 Integer arithmetic gotchas

```sql
SELECT 7 / 2;        -- 3     (INT / INT = INT in SQL Server, PostgreSQL)
SELECT 7 / 2.0;      -- 3.5
SELECT 7 * 1.0 / 2;  -- 3.5
-- 🔀 MySQL: 7 / 2 = 3.5000 ; integer division is 7 DIV 2.  Oracle: 7/2 = 3.5
SELECT 7 % 2;        -- 1 (modulo)
```

- ⚠️ In SQL Server `AVG` of an `INT` column returns **INT** (truncated): `AVG(1,2)` = **1**. Use `AVG(CAST(col AS DECIMAL(18,4)))` or `AVG(col * 1.0)`.
- ⚠️ `SUM(int_col)` returns **INT** and can **overflow** at 2,147,483,647 even if every value is small: use `SUM(CAST(col AS BIGINT))`.
- ⚠️ `COUNT(*)` returns **INT**; use `COUNT_BIG(*)` for tables beyond 2.1 billion rows.
- Percentage calculation: `amount * 100.0 / total` (the `.0` forces decimal).

### 7.4 How SQL Server decides the result type of decimal math

For `e1 (p1,s1)` and `e2 (p2,s2)`:

| Operation | Result precision | Result scale |
|-----------|------------------|--------------|
| `e1 + e2` | max(s1,s2) + max(p1−s1, p2−s2) + 1 | max(s1,s2) |
| `e1 * e2` | p1 + p2 + 1 | s1 + s2 |
| `e1 / e2` | p1 − s1 + s2 + max(6, s1 + p2 + 1) | max(6, s1 + p2 + 1) |

If precision would exceed 38, SQL Server reduces the **scale** (not below 6), which can silently round. This is why a long chain of multiplications and divisions on `DECIMAL(38,x)` can give "wrong looking" results; cast intermediate values deliberately.

### 7.5 Rounding

- `ROUND(x, d)`: SQL Server/Oracle round **half away from zero** (2.5 → 3, −2.5 → −3). Python's `round()` and some languages use banker's rounding (half to even), so values can differ between SQL and application code.
- `ROUND(x, d, 1)` in T-SQL **truncates** instead of rounding.
- `CAST(2.99 AS INT)` → **2** (decimal → int truncates); `CAST(2.995 AS DECIMAL(3,2))` → **3.00** (decimal → decimal with smaller scale rounds).
- `CEILING`, `FLOOR` return the same type family; remember `FLOOR(-1.5) = -2`.

---

## 8. Date and Time in Depth

### 8.1 Choosing the type

| Need | Type | Why |
|------|------|-----|
| Calendar date only | `DATE` | 3 bytes, no time-part trouble |
| Timestamp, server-local, no zone needed | `DATETIME2(3)` | 7 bytes, millisecond accuracy |
| Timestamp across zones | `DATETIME2` in **UTC**, convert in the app/reporting layer, or `DATETIMEOFFSET` | Avoid daylight-saving ambiguity |
| Time of day | `TIME(0)` | 3 bytes |
| Legacy | `DATETIME`, `SMALLDATETIME` | Avoid for new work: less range, odd rounding, 1753 lower bound |

### 8.2 The DATETIME rounding trap

`DATETIME` has ~3.33 ms accuracy: `'2024-03-31 23:59:59.999'` is rounded to `'2024-04-01 00:00:00.000'`. That is why inclusive end dates break:

```sql
-- ❌ BETWEEN with datetime end-of-day
WHERE order_ts BETWEEN '2024-03-01' AND '2024-03-31 23:59:59.999'   -- .999 rounds up into April on DATETIME!

-- ✅ always use half-open ranges: >= start AND < next_start
WHERE order_ts >= '2024-03-01' AND order_ts < '2024-04-01'
```
🎯 *"I never use BETWEEN on time-stamped data; I use `>= start AND < next period start`. It works for every precision and every type."*

### 8.3 Safe date literals

- `'20240301'` (`YYYYMMDD`): **always** interpreted correctly by every type and language setting.
- `'2024-03-01'` is safe for `DATE`, `DATETIME2`, `DATETIMEOFFSET`, but for **`DATETIME`/`SMALLDATETIME` it depends on `SET LANGUAGE`/`DATEFORMAT`** (under some settings it can be read as year-day-month).
- ISO 8601 with a `T` (`'2024-03-01T10:30:00'`) is safe for all types.
- Never use ambiguous formats like `'03/04/2024'` (3 April or March 4?).

### 8.4 Date arithmetic

```sql
-- T-SQL
DATEADD(DAY, 7, order_date)
DATEDIFF(DAY, d1, d2)              -- boundary count, not elapsed time
EOMONTH(order_date)                -- last day of month
DATEFROMPARTS(2024, 3, 1)

-- Start of month (works on all versions)
DATEFROMPARTS(YEAR(d), MONTH(d), 1)
-- SQL Server 2022+: DATETRUNC(MONTH, d)
-- PostgreSQL: DATE_TRUNC('month', d)   MySQL: DATE_FORMAT(d, '%Y-%m-01')
```

⚠️ **DATEDIFF counts boundaries crossed**, not full units:
`DATEDIFF(YEAR,'2023-12-31','2024-01-01')` = **1** (one year boundary crossed in one day), and `DATEDIFF(MONTH,'2024-01-31','2024-02-01')` = **1**.

**Correct age in years:**
```sql
DATEDIFF(YEAR, birth_date, GETDATE())
  - CASE WHEN DATEADD(YEAR, DATEDIFF(YEAR, birth_date, GETDATE()), birth_date) > GETDATE() THEN 1 ELSE 0 END
```
(If this year's birthday has not happened yet, subtract one.)

### 8.5 Weeks, quarters, fiscal years

- Week boundaries depend on `SET DATEFIRST` (1 = Monday; US default 7 = Sunday). Use `ISO_WEEK` for ISO weeks (`DATEPART(ISO_WEEK, d)`).
- Indian financial year (April to March), named by ending year (Apr 2024 → FY2025): `YEAR(DATEADD(MONTH, 9, d))`. Fiscal quarter: `((MONTH(d) + 8) % 12) / 3 + 1`.

### 8.6 Time zones

- **Store UTC**, convert for display. Local times are ambiguous twice a year in DST regions (India has no DST, but your users or servers elsewhere may).
- T-SQL: `SYSUTCDATETIME()`, `AT TIME ZONE 'UTC' AT TIME ZONE 'India Standard Time'`.
- 🔀 PostgreSQL `TIMESTAMPTZ` stores UTC internally and renders in the session `TimeZone`. It does **not** store the original zone.
- 🔀 MySQL `TIMESTAMP` converts to/from UTC using the session zone; `DATETIME` does not convert.
- `GETDATE()` returns server local time; `GETUTCDATE()` returns UTC. `CURRENT_TIMESTAMP` is the ANSI equivalent of `GETDATE()`.

### 8.7 Dates and indexes

```sql
WHERE YEAR(order_date) = 2024                         -- ❌ function on column → scan
WHERE order_date >= '20240101' AND order_date < '20250101'   -- ✅ seek
```
`CAST(datetime_col AS DATE) = '2024-03-01'` is a special case: SQL Server can still seek on it. Do not rely on that elsewhere.


---

## 9. DDL: Defining Structure

### 9.1 CREATE

```sql
CREATE DATABASE company_db;
CREATE SCHEMA hr;

CREATE TABLE hr.projects (
    project_id   INT IDENTITY(1,1) PRIMARY KEY,     -- SQL Server auto-number
    project_name VARCHAR(100) NOT NULL,
    start_date   DATE DEFAULT GETDATE(),
    budget       DECIMAL(12,2) CHECK (budget >= 0)
);
-- 🔀 PostgreSQL: project_id INT GENERATED ALWAYS AS IDENTITY   (or SERIAL, older style)
-- 🔀 MySQL:      project_id INT AUTO_INCREMENT
-- 🔀 Oracle 12c+: project_id NUMBER GENERATED ALWAYS AS IDENTITY
```

**Copy a table:**
```sql
SELECT * INTO emp_backup FROM employees;               -- T-SQL: structure + data (no constraints, no indexes!)
SELECT * INTO emp_empty  FROM employees WHERE 1 = 0;   -- structure only
CREATE TABLE emp_backup AS SELECT * FROM employees;    -- PostgreSQL / MySQL / Oracle
```
⚠️ `SELECT INTO` / `CREATE TABLE AS` copy column definitions only. PKs, FKs, defaults and indexes are **not** copied.

### 9.2 ALTER

```sql
ALTER TABLE employees ADD phone VARCHAR(15);                       -- add column
ALTER TABLE employees ALTER COLUMN phone VARCHAR(20) NULL;         -- change type (T-SQL)
--   PostgreSQL: ALTER COLUMN phone TYPE VARCHAR(20)     MySQL: MODIFY phone VARCHAR(20)
ALTER TABLE employees DROP COLUMN phone;
ALTER TABLE employees ADD CONSTRAINT uq_email UNIQUE (email);
ALTER TABLE employees DROP CONSTRAINT uq_email;
EXEC sp_rename 'employees.emp_name', 'full_name', 'COLUMN';        -- T-SQL rename
--   PostgreSQL / MySQL 8: ALTER TABLE employees RENAME COLUMN emp_name TO full_name;
```

🧠 What ALTER costs depends on whether it is a **metadata-only** change or a **size-of-data** operation (rewrites every row):

| Change | Cost on a huge table (SQL Server) |
|--------|-----------------------------------|
| Add a **nullable** column | Metadata only, instant |
| Add a NOT NULL column **with a default** | Metadata only in 2012+ (runtime constant default) |
| **Increase** `VARCHAR(n)` length | Metadata only |
| **Decrease** length / change type `INT → BIGINT` | **Rewrites the table**, long lock, big log |
| Drop a column | Metadata only (space reclaimed on rebuild) |

⚠️ `ALTER COLUMN` **resets nullability** to the default (NULL) if you do not restate `NOT NULL`. Always repeat the full definition.
⚠️ Schema changes take a **schema-modification (Sch-M) lock** that blocks everything else on the table; run them in quiet windows.

### 9.3 DROP and TRUNCATE
```sql
DROP TABLE IF EXISTS emp_backup;        -- IF EXISTS: SQL Server 2016+, PG, MySQL
TRUNCATE TABLE emp_empty;
```

### 9.4 Computed columns, sequences, synonyms

```sql
-- Computed column: value derived from other columns
ALTER TABLE employees ADD annual_salary AS (salary * 12);                -- virtual: calculated on read
ALTER TABLE employees ADD annual_salary2 AS (salary * 12) PERSISTED;     -- stored on disk, indexable
-- 🔀 PG 12+/MySQL: GENERATED ALWAYS AS (salary * 12) STORED
```
A virtual column costs CPU on every read and nothing on disk; a persisted one costs disk and write time but can be indexed. ⚡ Computed columns are a clean way to make a function-on-column predicate **SARGable** (index the persisted computed column).

A **synonym** is an alias for another object (`CREATE SYNONYM dbo.emp FOR hr_db.dbo.employees`), handy for migrations. A **sequence** is a standalone number generator (§11).

### 9.5 Naming conventions (pick one, be consistent)

`snake_case`, singular or plural table names (decide once), `pk_`, `fk_`, `ix_`, `uq_`, `ck_`, `df_` prefixes for constraints/indexes, never spaces or reserved words, no `tbl_` or `sp_` prefixes (`sp_` makes SQL Server search `master` first for procedures).

---

## 10. Constraints and Keys

🧠 A constraint is a **rule enforced by the database itself**, not by application code. Applications have bugs, multiple apps share one DB, and people run ad-hoc scripts, so the DB is the only reliable last line of defence for integrity.

### 10.1 The constraints

| Constraint | Purpose | Notes |
|-----------|---------|-------|
| `NOT NULL` | Value is mandatory | Also saves nothing on storage but gives the optimizer information |
| `UNIQUE` | No duplicate values | 🔀 SQL Server allows **one** NULL; PostgreSQL, MySQL, Oracle (single column) allow **many** NULLs. Backed by a unique index |
| `PRIMARY KEY` | `NOT NULL` + `UNIQUE`; one per table | SQL Server: creates a **clustered** index by default (if none exists). Can be composite |
| `FOREIGN KEY` | Value must exist in the referenced table (or be NULL) | Referenced columns must be a PK or UNIQUE |
| `CHECK` | Validates a condition | ⚠️ A CHECK **passes when the result is UNKNOWN** (NULL). `CHECK (salary > 0)` allows NULL salary; add `NOT NULL` too |
| `DEFAULT` | Value used when none is supplied | In SQL Server a default is a **named object**; name it so it can be dropped |

### 10.2 Foreign key behaviour

```sql
FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
    ON DELETE CASCADE        -- delete children automatically
    ON UPDATE CASCADE;
-- Other actions: SET NULL | SET DEFAULT | NO ACTION (default; errors if children exist) | RESTRICT (PG/MySQL)
```

Hard-won facts:

- ⚡ **SQL Server and PostgreSQL do not automatically index FK columns** (MySQL InnoDB does). Without an index on `orders.customer_id`, deleting a customer must **scan the whole orders table** to check for children, and joins on that column are slow. Index your FK columns.
- `NULL` in an FK column means "no parent", and is allowed (Divya has no department).
- `ON DELETE CASCADE` is convenient and dangerous (one delete wipes thousands of rows silently). SQL Server forbids **multiple cascade paths** to the same table (error 1785).
- Self-referencing FK (`manager_id → emp_id`) is how hierarchies are modelled.
- Constraints can be created `WITH NOCHECK` (existing rows not validated) but then they are marked **not trusted**, and the optimizer cannot use them. Re-validate with `WITH CHECK CHECK CONSTRAINT`.
- 🔀 PostgreSQL supports `DEFERRABLE INITIALLY DEFERRED` (check at commit instead of per statement), useful for circular inserts.

### 10.3 The vocabulary of keys

| Key | Definition |
|-----|-----------|
| **Super key** | Any set of columns that uniquely identifies a row (can contain extras) |
| **Candidate key** | A **minimal** super key: remove any column and uniqueness is lost |
| **Primary key** | The candidate key you chose; NOT NULL |
| **Alternate key** | Candidate keys not chosen (enforce with UNIQUE) |
| **Composite key** | A key made of 2+ columns |
| **Natural key** | A real-world identifier (email, Aadhaar, PAN, ISBN) |
| **Surrogate key** | A meaningless system-generated id (identity, sequence, GUID) |
| **Foreign key** | Points to a PK/UNIQUE of another (or the same) table |

**Example:** `employees(emp_id, email, pan)`: `{emp_id}`, `{email}`, `{pan}` are candidate keys; `{emp_id, email}` is a super key but not candidate (not minimal).

### 10.4 Surrogate vs natural: how to argue it

- **Natural keys** carry meaning but *change* (people change emails, PANs get corrected), are often wide strings, and cascading a change through child tables is costly.
- **Surrogate keys** are narrow (INT/BIGINT), stable and fast to join; they carry no meaning, so you **still enforce the natural key with a UNIQUE constraint**.
- Warehouses *always* use surrogate keys (to support SCD2 history, §33).

🎯 *"Surrogate primary key for joins and stability, plus a unique constraint on the natural key for integrity."*

**PK vs UNIQUE:** PK: one per table, no NULLs, clustered by default (SQL Server). UNIQUE: many per table, NULL rules vary, non-clustered by default.

---

## 11. Identity, Sequences and GUIDs

### 11.1 IDENTITY (SQL Server)

```sql
CREATE TABLE t (id INT IDENTITY(1,1) PRIMARY KEY, v VARCHAR(10));  -- seed 1, increment 1
SELECT SCOPE_IDENTITY();            -- last identity generated in THIS scope (use this)
SELECT @@IDENTITY;                  -- last in session, any scope (a trigger can change it! avoid)
SELECT IDENT_CURRENT('t');          -- last for table, any session
SET IDENTITY_INSERT t ON;           -- allow explicit id values (one table at a time)
DBCC CHECKIDENT ('t', RESEED, 0);   -- reset counter
```

⚠️ **Identity values are not gapless.** They are consumed even if the insert rolls back, if the row is deleted, or if the server restarts (SQL Server pre-allocates a cache and can skip values after a crash). Never use identity for invoice numbers that must be consecutive; use a controlled counter table or sequence with locking, and accept legal requirements may need their own process.
⚠️ `TRUNCATE` resets identity; `DELETE` does not.
⚠️ `INT` identity runs out at **2,147,483,647**. At 1,000 inserts/second (about 86.4 million per day) that takes only around 25 days. Plan for `BIGINT` when growth is high.

### 11.2 SEQUENCE objects

```sql
CREATE SEQUENCE seq_invoice START WITH 1000 INCREMENT BY 1 CACHE 50;
SELECT NEXT VALUE FOR seq_invoice;                                  -- T-SQL
-- PostgreSQL: SELECT nextval('seq_invoice');    Oracle: seq_invoice.NEXTVAL
INSERT INTO invoices(id, ...) VALUES (NEXT VALUE FOR seq_invoice, ...);
```
Sequences are independent of any table, shareable across tables, and can be fetched **before** the insert (useful when you need the id first).

🔀 **PostgreSQL:** `SERIAL` is old style (integer + sequence + default); modern style is `GENERATED ALWAYS AS IDENTITY`. 🔀 **MySQL:** `AUTO_INCREMENT`; after a restart in older versions the counter could be recomputed from `MAX(id)+1`.

### 11.3 GUID / UUID keys

| Aspect | `INT` identity | Random GUID (`NEWID()`) | Sequential GUID |
|--------|----------------|-------------------------|-----------------|
| Size | 4 B | 16 B | 16 B |
| Generated by | DB | App or DB | DB (`NEWSEQUENTIALID()`) / app (UUIDv7) |
| Insert pattern | Always at end | **Random** → page splits, fragmentation | At end |
| Guessable | Yes | No | Partly |
| Merge across systems | Collisions | Safe | Safe |

🧠 *Why random keys hurt:* a clustered index is sorted by key. New rows with random keys land in random pages; when a target page is full, the engine **splits** it (moves half the rows to a new page). Result: pages half empty (wasted memory and I/O), physical disorder and more log. Sequential keys always append to the last page.

**Rule of thumb:** use a narrow ever-increasing clustered key (INT/BIGINT identity), and if you need GUIDs for outside use, store them as a **non-clustered unique** column.

---

## 12. DML: Modifying Data

### 12.1 INSERT

```sql
INSERT INTO departments (dept_id, dept_name, location) VALUES (5, 'Sales', 'Chennai');   -- always list columns
INSERT INTO departments VALUES (6,'Legal','Pune'), (7,'Ops','Delhi');                    -- multi-row (up to 1000 row values in T-SQL)
INSERT INTO emp_backup SELECT * FROM employees WHERE dept_id = 2;                        -- from a query
```
- Always list columns: if someone adds a column later, positional inserts break.
- ⚡ Bulk loads: `BULK INSERT`, `bcp`, `INSERT ... SELECT` with `TABLOCK` (can be **minimally logged** under SIMPLE/BULK_LOGGED recovery), PostgreSQL `COPY`, MySQL `LOAD DATA`. One multi-row insert is far faster than a thousand single-row inserts (each is its own transaction/log flush unless wrapped).

### 12.2 UPDATE

```sql
UPDATE employees SET salary = salary * 1.10 WHERE dept_id = 2;

-- Update using a JOIN (T-SQL)
UPDATE e SET e.salary = e.salary + 5000
FROM employees e JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_name = 'IT';
-- 🔀 PostgreSQL: UPDATE employees e SET ... FROM departments d WHERE e.dept_id = d.dept_id AND d.dept_name='IT';
-- 🔀 MySQL:      UPDATE employees e JOIN departments d ON ... SET e.salary = ...
```
⚠️ **Habit that saves careers:** run the same `WHERE` as a `SELECT` first, then:
```sql
BEGIN TRAN;
UPDATE ... ;
SELECT @@ROWCOUNT;      -- is this the number I expected?
-- COMMIT;  or  ROLLBACK;
```
⚠️ An `UPDATE` without `WHERE` updates **every row**.
🧠 An UPDATE is physically "delete old + insert new" if the row grows beyond the page (with *forwarding pointers* on heaps); in PostgreSQL every UPDATE writes a **new row version**, leaving a dead one for VACUUM.

### 12.3 DELETE and de-duplication

```sql
DELETE FROM employees WHERE emp_id = 6;

-- Delete duplicates, keep one (T-SQL: delete through a CTE)
WITH d AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY email ORDER BY emp_id) rn FROM employees WHERE email IS NOT NULL
)
DELETE FROM d WHERE rn > 1;
-- 🔀 PostgreSQL: DELETE FROM employees WHERE ctid IN (SELECT ctid FROM (SELECT ctid, ROW_NUMBER() OVER (PARTITION BY email ORDER BY emp_id) rn FROM employees) x WHERE rn > 1);
```
⚠️ `PARTITION BY email` puts all NULL emails in the same partition; filter `email IS NOT NULL` (as above) unless you really want to delete rows with missing emails.

### 12.4 Batching big changes

```sql
WHILE 1 = 1
BEGIN
    DELETE TOP (10000) FROM big_log WHERE created < DATEADD(YEAR, -2, GETDATE());
    IF @@ROWCOUNT = 0 BREAK;
END;
```
Why: each batch is a short transaction, so the log can be reused (in SIMPLE/after log backups), locks stay row/page-level (escalation to a table lock happens around **5,000 locks** per statement), and other sessions are not blocked for hours. Needs an index on the filter column.

### 12.5 MERGE (upsert) and its alternatives

```sql
-- SQL Server / Oracle
MERGE INTO target AS t
USING source AS s ON t.id = s.id
WHEN MATCHED THEN UPDATE SET t.name = s.name
WHEN NOT MATCHED BY TARGET THEN INSERT (id, name) VALUES (s.id, s.name)
WHEN NOT MATCHED BY SOURCE THEN DELETE;               -- T-SQL only; end the statement with ;

-- 🔀 PostgreSQL
INSERT INTO target (id, name) VALUES (1,'A') ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;
-- 🔀 MySQL
INSERT INTO target (id, name) VALUES (1,'A') ON DUPLICATE KEY UPDATE name = VALUES(name);
```
⚠️ SQL Server's `MERGE` has a history of bugs and **race conditions under concurrency** (two sessions both see "not matched" and both insert → PK violation). Mitigate with `MERGE ... WITH (HOLDLOCK)`, or use explicit `UPDATE` then `INSERT ... WHERE NOT EXISTS` inside a transaction. Many teams ban it in OLTP code.
⚠️ The source must not contain duplicate join keys, or MERGE errors ("attempted to update the same row more than once").

### 12.6 Capturing affected rows

```sql
DELETE FROM orders OUTPUT DELETED.* WHERE status = 'Cancelled';          -- T-SQL
INSERT INTO t (v) OUTPUT INSERTED.id VALUES ('x');                        -- get generated ids for multi-row inserts
DELETE FROM orders WHERE status = 'Cancelled' RETURNING *;                -- 🔀 PostgreSQL / Oracle
```

---

## 13. SELECT Basics and Logical Execution Order

```sql
SELECT [DISTINCT] [TOP n] columns / expressions
FROM table
[JOIN ... ON ...]
[WHERE condition]
[GROUP BY columns]
[HAVING group_condition]
[ORDER BY columns]
[OFFSET ... FETCH | LIMIT];
```

### 13.1 Logical order of evaluation (the most useful concept in SQL)

You **write** `SELECT` first, but the engine **logically** evaluates it almost last:

| Step | Clause | What happens |
|------|--------|--------------|
| 1 | `FROM` / `JOIN` / `ON` | Identify and combine source rows |
| 2 | `WHERE` | Filter rows (before grouping) |
| 3 | `GROUP BY` | Form groups |
| 4 | `HAVING` | Filter groups |
| 5 | `SELECT` | Compute expressions, aliases, window functions |
| 6 | `DISTINCT` | Remove duplicates |
| 7 | `ORDER BY` | Sort |
| 8 | `TOP` / `OFFSET-FETCH` / `LIMIT` | Cut the result |

Consequences you can now *derive* instead of memorize:

- A **column alias** from `SELECT` can't be used in `WHERE`/`GROUP BY`/`HAVING` (not created yet) but **can** in `ORDER BY` (runs after SELECT). 🔀 MySQL and PostgreSQL allow aliases in `GROUP BY`; SQL Server does not.
- **Window functions** are computed in step 5, so they can't be in `WHERE`/`HAVING`: wrap in a CTE/subquery or use `QUALIFY` where available.
- **Aggregates** can't be in `WHERE` (groups don't exist yet) → use `HAVING`.
- `WHERE` removes rows **before** aggregation, so it makes the aggregate cheaper; `HAVING` removes after. Put row-level filters in `WHERE`.

> ⚠️ This is *logical* order. The optimizer may physically reorder things (push filters down, reorder joins) as long as the result is identical.

### 13.2 Basics

```sql
SELECT * FROM employees;                                  -- avoid * in real code
SELECT emp_name, salary FROM employees;
SELECT emp_name AS name, salary * 12 AS annual FROM employees;
SELECT DISTINCT dept_id FROM employees;
SELECT DISTINCT dept_id, manager_id FROM employees;       -- distinct over the combination of columns
SELECT 'Hello' AS greeting, 1 + 1 AS result;              -- no FROM needed (Oracle: FROM DUAL)
```

**Why `SELECT *` is discouraged:** more I/O and network, prevents covering-index plans, breaks when columns are added/reordered, drags in big columns, and hides what a query depends on.

### 13.3 DISTINCT vs GROUP BY
Both de-duplicate. `DISTINCT` is shorthand for "group by all selected columns". Use `GROUP BY` when you need aggregates; use `DISTINCT` for a plain unique list. ⚠️ `DISTINCT` used to "fix" duplicate rows after a join is hiding a join bug (fan-out); find the cause instead.

### 13.4 Boolean evaluation order is NOT guaranteed
`WHERE x <> 0 AND 10 / x > 1` is **not** guaranteed to short-circuit; the optimizer may evaluate either side first and you get divide-by-zero. Use `CASE`, `NULLIF`, or `TRY_CAST`: `WHERE 10 / NULLIF(x,0) > 1`. Same for `WHERE ISNUMERIC(col)=1 AND CAST(col AS INT) > 5`.

---

## 14. Filtering: WHERE and Operators

### 14.1 Operators

| Type | Operators |
|------|-----------|
| Comparison | `=  <>  !=  >  <  >=  <=` (`<>` is the ANSI form; `!=` also works in most engines) |
| Logical | `AND  OR  NOT`; precedence **NOT > AND > OR**, so use parentheses |
| Range | `BETWEEN a AND b` (**inclusive** both ends; `BETWEEN 10 AND 5` returns nothing) |
| List | `IN (...)`, `NOT IN (...)` |
| Pattern | `LIKE`, `NOT LIKE` |
| NULL | `IS NULL`, `IS NOT NULL` |
| Existence | `EXISTS`, `NOT EXISTS` |
| Quantified | `ANY / SOME / ALL` |

```sql
SELECT * FROM employees WHERE salary > 70000 AND dept_id = 2;
SELECT * FROM employees WHERE dept_id = 1 OR dept_id = 2 AND salary > 65000;     -- AND binds first! reads as dept=1 OR (dept=2 AND salary>65000)
SELECT * FROM employees WHERE (dept_id = 1 OR dept_id = 2) AND salary > 65000;   -- what you probably meant
```

### 14.2 LIKE wildcards

| Wildcard | Meaning |
|----------|---------|
| `%` | zero or more characters |
| `_` | exactly one character |
| `[abc]`, `[a-f]` | 🔀 T-SQL only: one char from the set/range |
| `[^abc]` | 🔀 T-SQL only: one char not in set |

```sql
WHERE emp_name LIKE 'A%'          -- starts with A  (can use an index)
WHERE emp_name LIKE '%a'          -- ends with a    (scan)
WHERE emp_name LIKE '%av%'        -- contains       (scan)
WHERE emp_name LIKE '_a%'         -- second letter a
WHERE emp_name LIKE '[A-M]%'      -- T-SQL
WHERE col LIKE '%50\%%' ESCAPE '\'   -- literal percent sign
```
⚡ `LIKE 'abc%'` is **SARGable** (the index is sorted, so the engine seeks to 'abc' and reads until it stops matching). A leading wildcard cannot seek: a sorted list is no help when you do not know the first letters. For "contains" search use **Full-Text Search**, PostgreSQL `pg_trgm` + GIN, or search engines.
🔀 `ILIKE` = case-insensitive LIKE in PostgreSQL.

### 14.3 BETWEEN and time-stamped data
See §8.2: use `>= start AND < next_start`.

### 14.4 IN vs EXISTS

| | `IN` | `EXISTS` |
|--|----|----------|
| Question asked | "Is value in this list?" | "Does at least one matching row exist?" |
| NULL behaviour | `NOT IN` + a NULL in the list → **no rows** ⚠️ | Safe |
| Performance | Fine for small lists | Can stop at the first match (semi-join) |

```sql
SELECT * FROM customers c WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);
```
The `SELECT 1` is convention only; `EXISTS` ignores the select list. Modern optimizers often produce the same plan for IN and EXISTS; the **NULL safety** difference is the reason to prefer `NOT EXISTS` over `NOT IN`.

**The NOT IN trap with our data:**
```sql
SELECT * FROM departments WHERE dept_id NOT IN (SELECT dept_id FROM employees);
-- employees.dept_id contains a NULL (Divya) → "dept_id <> NULL" is UNKNOWN for every row → returns ZERO rows
SELECT * FROM departments d WHERE NOT EXISTS (SELECT 1 FROM employees e WHERE e.dept_id = d.dept_id);  -- ✅ returns Marketing
```

### 14.5 ANY / ALL
```sql
WHERE salary > ALL (SELECT salary FROM employees WHERE dept_id = 1)   -- greater than the maximum (if the list is empty, ALL is TRUE)
WHERE salary > ANY (SELECT salary FROM employees WHERE dept_id = 1)   -- greater than the minimum (if empty, ANY is FALSE)
```

---

## 15. NULL Handling

**NULL means "unknown / missing / not applicable"**, not zero, not empty string, not false. SQL uses **three-valued logic**: TRUE, FALSE, **UNKNOWN**. `WHERE` keeps only rows where the condition is TRUE; UNKNOWN rows are dropped just like FALSE.

| Expression | Result |
|-----------|--------|
| `NULL = NULL` | UNKNOWN |
| `NULL <> 5` | UNKNOWN |
| `NULL + 5` | NULL |
| `NULL AND FALSE` | FALSE |
| `NULL AND TRUE` | UNKNOWN |
| `NULL OR TRUE` | TRUE |
| `NOT UNKNOWN` | UNKNOWN |
| `5 NOT IN (1, NULL)` | UNKNOWN → row excluded ⚠️ |

```sql
WHERE email IS NULL;   WHERE email IS NOT NULL;

COALESCE(email, phone, 'N/A')    -- first non-NULL, ANSI, any number of args; result type follows data type precedence
ISNULL(email, 'N/A')             -- T-SQL, 2 args; result takes the type/length of the FIRST argument (can truncate!)
NULLIF(a, b)                     -- NULL if a = b else a  → safe divide: total / NULLIF(count, 0)
-- 🔀 IFNULL (MySQL), NVL (Oracle).  🔀 PG / SQL Server 2022: a IS DISTINCT FROM b  (NULL-safe comparison)
```
⚠️ `ISNULL(short_varchar_col, 'a long replacement')` truncates the replacement to the column's length; `COALESCE` does not.

### 15.1 NULL everywhere else

- **Aggregates:** `COUNT(*)` counts rows; `COUNT(col)` counts non-NULLs. `SUM/AVG/MIN/MAX` ignore NULLs, so `AVG` = sum of non-NULL ÷ **count of non-NULL** (not ÷ rows). On an empty set `COUNT` = 0 but `SUM`/`AVG` = NULL (use `COALESCE(SUM(x),0)`).
- **GROUP BY / DISTINCT / UNION / INTERSECT / EXCEPT** treat NULLs as **equal** (one group). But `=` in JOIN/WHERE does not, so NULL never joins with NULL.
- **ORDER BY:** SQL Server/MySQL treat NULL as the **lowest** value (first in ASC); PostgreSQL/Oracle treat it as the **highest** (last in ASC). Control with `NULLS FIRST/LAST` (PG, Oracle) or `ORDER BY CASE WHEN col IS NULL THEN 1 ELSE 0 END, col`.
- **Constraints:** `CHECK` passes on UNKNOWN; `UNIQUE` NULL rules vary (§10.1); FK with NULL = no parent.
- **Strings:** `'a' + NULL` = NULL in T-SQL (use `CONCAT`).
- **Storage:** In fixed-length columns NULL still occupies the full column width in SQL Server; for variable-length it costs ~0 bytes of data. The null bitmap (1 bit per column) tracks it.
- **Indexes:** SQL Server and PostgreSQL index NULLs; Oracle B-tree indexes do **not** include rows where all key columns are NULL (so `WHERE col IS NULL` can't use that index).
- **Design:** NULL is information ("unknown"). Do not use magic values (`-1`, `'1900-01-01'`, `'N/A'`) in columns that feed aggregates or FKs; they corrupt averages and joins.

---

## 16. Sorting and Limiting

```sql
SELECT * FROM employees ORDER BY salary DESC, emp_name ASC;
SELECT * FROM employees ORDER BY 3;                        -- by position: avoid, breaks when SELECT list changes

-- Top-N
SELECT TOP 3 * FROM employees ORDER BY salary DESC;                    -- T-SQL
SELECT TOP 3 WITH TIES * FROM employees ORDER BY salary DESC;          -- includes ties at the cutoff (needs ORDER BY)
SELECT TOP 10 PERCENT * FROM employees ORDER BY salary DESC;
SELECT * FROM employees ORDER BY salary DESC LIMIT 3;                  -- PostgreSQL / MySQL
SELECT * FROM employees ORDER BY salary DESC FETCH FIRST 3 ROWS ONLY;  -- ANSI, Oracle 12c+, PG, SQL Server (with OFFSET)

-- Pagination
SELECT * FROM employees ORDER BY emp_id OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;   -- T-SQL (ORDER BY required)
-- PG/MySQL: ... ORDER BY emp_id LIMIT 10 OFFSET 20;
```

### 16.1 Rules and traps

- Without `ORDER BY`, order is **undefined**. `ORDER BY` inside views, derived tables, subqueries and CTEs is **not allowed** in T-SQL (unless combined with `TOP`/`OFFSET`) and is not honoured by the outer query anyway.
- **Ties make pagination unstable.** `ORDER BY salary` with duplicate salaries can return different rows on page 2 and skip or repeat some. Always add a unique tie-breaker: `ORDER BY salary DESC, emp_id`.
- You may `ORDER BY` a column that is not in the `SELECT` list (except with `DISTINCT`).
- Sorting is expensive (O(n log n), may **spill to tempdb** when memory grant is insufficient). An index in the right order removes the sort entirely.
- **OFFSET is O(offset):** `OFFSET 1000000` still reads and discards a million rows. For deep paging use **keyset (seek) pagination**:

```sql
-- "next page after the last row I showed (last_order_id = 5000)"
SELECT TOP 20 * FROM orders WHERE order_id > 5000 ORDER BY order_id;     -- uses the index, constant cost per page
```

---

## 17. Aggregates, GROUP BY, HAVING

### 17.1 Aggregate functions

| Function | Purpose | Notes |
|----------|---------|-------|
| `COUNT(*)` | all rows | `COUNT(1)` is identical |
| `COUNT(col)` | non-NULL values | |
| `COUNT(DISTINCT col)` | distinct non-NULL | expensive: sort/hash; `APPROX_COUNT_DISTINCT` for big data |
| `SUM, AVG, MIN, MAX` | standard | ignore NULLs; type gotchas in §7.3 |
| `STRING_AGG(col, ', ') WITHIN GROUP (ORDER BY col)` | concatenate | SQL Server 2017+, PG. 🔀 MySQL `GROUP_CONCAT`, Oracle `LISTAGG` |
| `STDEV, VAR` (T-SQL) | statistics | `STDEVP/VARP` for population |

**Scalar aggregate:** an aggregate with no `GROUP BY` always returns **exactly one row**, even on an empty table (`SELECT COUNT(*) ...` = 0, `SUM` = NULL). With `GROUP BY`, an empty input gives **zero** rows.

### 17.2 GROUP BY

```sql
SELECT dept_id, COUNT(*) AS emp_cnt, AVG(salary) AS avg_sal, MAX(salary) AS max_sal
FROM employees
GROUP BY dept_id;
```
**Rule:** every column in `SELECT` must be either aggregated or in `GROUP BY` (🔀 MySQL with `ONLY_FULL_GROUP_BY` off silently picks an arbitrary value, a classic source of wrong results). Physically done by a **Stream Aggregate** (needs sorted input, e.g., via index) or a **Hash Aggregate**.

### 17.3 HAVING vs WHERE

| WHERE | HAVING |
|-------|--------|
| Filters **rows** before grouping | Filters **groups** after aggregation |
| Cannot use aggregates | Can use aggregates |
| Runs earlier, shrinks data for cheaper aggregation | Runs later |

```sql
SELECT dept_id, AVG(salary) avg_sal
FROM employees
WHERE hire_date >= '2019-01-01'     -- row filter
GROUP BY dept_id
HAVING AVG(salary) > 60000;         -- group filter
```

### 17.4 GROUPING SETS, ROLLUP, CUBE

```sql
GROUP BY ROLLUP (dept_id, manager_id)    -- (dept,mgr) + (dept) subtotals + () grand total   → n+1 grouping levels
GROUP BY CUBE (dept_id, manager_id)      -- all combinations                                  → 2^n levels
GROUP BY GROUPING SETS ((dept_id), (manager_id), ())   -- exactly the sets you list

SELECT dept_id, GROUPING(dept_id) AS is_subtotal, SUM(salary)   -- 1 means "this NULL is a rollup marker", not real data
FROM employees GROUP BY ROLLUP (dept_id);
```
`GROUPING()` is how you tell a real NULL group from a subtotal row.

### 17.5 Common patterns

```sql
-- Find duplicates
SELECT email, COUNT(*) FROM employees GROUP BY email HAVING COUNT(*) > 1;

-- Conditional counts without extra queries
SELECT dept_id, COUNT(CASE WHEN salary > 70000 THEN 1 END) AS high_earners FROM employees GROUP BY dept_id;
-- (COUNT ignores the NULLs produced by CASE without ELSE; that is the trick)
```

---

## 18. Joins

🧠 A join combines rows from two tables **horizontally** based on a condition. Logically it is "take every pair of rows (a Cartesian product), keep the pairs where the condition is true". Engines obviously don't do it that way, but the *result* is defined that way, which explains everything below.

### 18.1 Types

| Join | Returns |
|------|---------|
| `INNER JOIN` | Pairs that match |
| `LEFT [OUTER] JOIN` | All left rows; right columns NULL when no match |
| `RIGHT [OUTER] JOIN` | Mirror of left (rarely needed; swap the table order and use LEFT) |
| `FULL [OUTER] JOIN` | All rows from both; NULLs where no match |
| `CROSS JOIN` | Every combination: m × n rows |
| `SELF JOIN` | A table joined with itself (aliases required) |
| `NATURAL JOIN` / `USING` | Auto-join on same-named columns (`NATURAL` is dangerous: a later added column silently changes the join) |
| Semi join | Left rows that **have** a match: `EXISTS` / `IN` (no duplication) |
| Anti join | Left rows that **have no** match: `NOT EXISTS` / `LEFT JOIN ... IS NULL` |

```text
 INNER = A ∩ B            LEFT = A only + A∩B          RIGHT = B only + A∩B
 FULL  = A only + A∩B + B only       LEFT ANTI = A only  (LEFT JOIN … WHERE b.key IS NULL)
```

### 18.2 Examples

```sql
-- INNER
SELECT e.emp_name, d.dept_name
FROM employees e INNER JOIN departments d ON e.dept_id = d.dept_id;       -- Divya is missing (NULL dept)

-- LEFT: keep every employee
SELECT e.emp_name, d.dept_name
FROM employees e LEFT JOIN departments d ON e.dept_id = d.dept_id;

-- Anti join: departments with no employees
SELECT d.dept_name FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
WHERE e.emp_id IS NULL;                                                   -- test a NOT NULL column of the right table

-- FULL
SELECT e.emp_name, d.dept_name
FROM employees e FULL OUTER JOIN departments d ON e.dept_id = d.dept_id;  -- 🔀 not in MySQL: LEFT JOIN UNION RIGHT JOIN

-- SELF: employee with manager
SELECT e.emp_name AS employee, m.emp_name AS manager
FROM employees e LEFT JOIN employees m ON e.manager_id = m.emp_id;

-- Multiple joins
SELECT c.cust_name, o.order_id, o.amount
FROM customers c JOIN orders o ON o.customer_id = c.customer_id
WHERE o.status = 'Delivered';
```

### 18.3 ON vs WHERE in outer joins ⚠️

```sql
-- Keeps ALL customers; the status filter applies while joining (Sneha and Priya still appear, with NULL orders)
SELECT c.cust_name, o.order_id
FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id AND o.status = 'Delivered';

-- Silently becomes an INNER JOIN: WHERE runs after the join and drops the NULL-extended rows
SELECT c.cust_name, o.order_id
FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.status = 'Delivered';
```
For **inner joins** the two placements are equivalent; for **outer joins** they are not. Rule: conditions on the **preserved** side go in `WHERE`; conditions on the **optional** side go in `ON`.

### 18.4 Join algorithms (how the engine does it)

| Algorithm | How | Best when |
|-----------|-----|-----------|
| **Nested Loops** | For each row of the outer input, seek matching rows in the inner input | Small outer input + **index on the inner** join column. Cost ≈ outer rows × seek cost |
| **Hash Join** | Build a hash table from the smaller input, probe with the larger | Large unsorted inputs, equality joins, no useful index. Needs memory (may spill to tempdb) |
| **Merge Join** | Walk two inputs **sorted** on the join key in step | Both inputs already sorted (e.g., by indexes) |

🧠 You do not choose; the optimizer does from statistics. Your levers: indexes, accurate statistics, SARGable predicates, and query shape.

### 18.5 Pitfalls

- **Row explosion (fan-out):** joining on a non-unique key on *both* sides multiplies rows. Example: customer 1 has 3 orders, and if you also join `payments` (2 per order) before summing, order amounts get counted repeatedly → **inflated totals**. Pre-aggregate each side to the grain you join on, *then* join.
- **NULLs never match** in `=` joins.
- **Implicit cartesian products** from old comma-style joins (`FROM a, b WHERE a.id = b.id`) when the WHERE condition is forgotten. Use explicit `JOIN ... ON`.
- **Join column types must match** (§5.7).
- Not every join key is unique: check the grain before you join (`GROUP BY key HAVING COUNT(*) > 1`).

### 18.6 CROSS APPLY / OUTER APPLY (LATERAL)

A normal join cannot reference the left row inside a subquery on the right; `APPLY` can. It's "run this subquery for each left row".

```sql
-- Top 2 orders per customer
SELECT c.cust_name, t.order_id, t.amount
FROM customers c
CROSS APPLY (SELECT TOP 2 * FROM orders o WHERE o.customer_id = c.customer_id ORDER BY amount DESC) t;
-- OUTER APPLY keeps customers with no orders (like LEFT JOIN). 🔀 PG/MySQL/Oracle: CROSS JOIN LATERAL / LEFT JOIN LATERAL
```
Also used to call inline TVFs per row and to **unpivot** (`CROSS APPLY (VALUES (...),(...)) v(col, val)`).

---

## 19. Set Operators

| Operator | Returns | Duplicates |
|----------|---------|------------|
| `UNION` | Rows in either | **Removed** (needs sort/hash → slower) |
| `UNION ALL` | Rows in either | Kept (faster) |
| `INTERSECT` | Rows in both | Distinct |
| `EXCEPT` (🔀 Oracle `MINUS`) | Rows in first, not in second | Distinct |

**Rules:** same number of columns; compatible types (resolved by data type precedence); **column names come from the first query**; `ORDER BY` only once at the very end; NULLs are treated as equal. **Precedence:** `INTERSECT` binds tighter than `UNION`/`EXCEPT`; use parentheses.

```sql
SELECT city FROM customers UNION SELECT location FROM departments;
SELECT dept_id FROM employees EXCEPT SELECT dept_id FROM departments;      -- NULL-safe, unlike NOT IN
```
- `UNION` vs `JOIN`: UNION stacks rows **vertically** (same columns); JOIN adds columns **horizontally**.
- Default to `UNION ALL`, and use `UNION` only when you truly need de-duplication (and consider whether duplicates are possible at all).
- `EXCEPT`/`INTERSECT` compare **all** selected columns, which is handy for diffing two tables (data migration validation): `SELECT * FROM a EXCEPT SELECT * FROM b` and the reverse.
- 🔀 MySQL gained `INTERSECT`/`EXCEPT` only in **8.0.31**.

---

## 20. Subqueries

A query nested inside another.

### 20.1 Types

| Type | Returns | Typical place |
|------|---------|---------------|
| Scalar | exactly 1 row, 1 column | `SELECT`, `WHERE`, `SET` |
| Row | 1 row, many columns | `WHERE (a,b) = (SELECT ...)` (PG/MySQL/Oracle) |
| Multi-row | many rows, 1 column | `IN`, `ANY`, `ALL`, `EXISTS` |
| Derived table | a table | `FROM (SELECT ...) AS t` (alias required) |
| Correlated | references the outer query | evaluated *logically* per outer row |

```sql
SELECT emp_name FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);          -- scalar

SELECT * FROM employees WHERE dept_id IN (SELECT dept_id FROM departments WHERE location = 'Hyderabad');

SELECT dept_id, avg_sal FROM (SELECT dept_id, AVG(salary) avg_sal FROM employees GROUP BY dept_id) t WHERE avg_sal > 70000;

-- Correlated: employees above their own department average
SELECT e.emp_name, e.salary FROM employees e
WHERE e.salary > (SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id);

-- Scalar subquery in SELECT
SELECT c.cust_name, (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.customer_id) AS order_cnt FROM customers c;
```

⚠️ A scalar subquery that returns **more than one row** fails at runtime ("Subquery returned more than 1 value"). Zero rows → NULL.
🧠 "Correlated = runs once per row" is the *logical* model. Optimizers often **unnest** correlated subqueries into joins, so don't assume it is slow, but do check the plan when it is.

### 20.2 Nth highest salary (the classic)

```sql
-- 1) DENSE_RANK: handles ties properly (3rd highest DISTINCT salary)
SELECT DISTINCT salary FROM (SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) rk FROM employees) t WHERE rk = 3;

-- 2) Subquery (2nd highest)
SELECT MAX(salary) FROM employees WHERE salary < (SELECT MAX(salary) FROM employees);

-- 3) OFFSET/FETCH over distinct salaries (T-SQL)
SELECT DISTINCT salary FROM employees ORDER BY salary DESC OFFSET 2 ROWS FETCH NEXT 1 ROWS ONLY;
```
Discuss: what if there are ties? what if fewer than N distinct values (returns empty, or NULL if wrapped in a scalar)? That is what the interviewer is really testing.

### 20.3 Subquery vs JOIN vs CTE
- Equivalent logic often compiles to the **same plan**. Choose for **clarity**.
- Need columns from both tables → JOIN. Only checking existence → EXISTS. Reusing/naming steps → CTE.
- `IN (subquery)` with possible NULLs and `NOT IN` → switch to `EXISTS`/`NOT EXISTS`.

---

## 21. CTEs and Recursive CTEs

### 21.1 CTE (Common Table Expression)

A CTE is a **named result set that exists only for the one statement that follows it**. Think of it as giving a name to a subquery so the main query reads top-to-bottom like a story.

```sql
WITH dept_avg AS (
    SELECT dept_id, AVG(salary) AS avg_sal FROM employees GROUP BY dept_id
),
high_depts AS (
    SELECT dept_id FROM dept_avg WHERE avg_sal > 70000
)
SELECT e.* FROM employees e JOIN high_depts h ON e.dept_id = h.dept_id;
```

- Benefits: readability, modular steps, a CTE can reference earlier CTEs, it can be referenced multiple times, supports recursion, and you can `UPDATE`/`DELETE` through it (T-SQL).
- ⚠️ A statement before `WITH` must end with `;` in T-SQL (the classic "Incorrect syntax near WITH").
- 🧠 **A CTE is not a temp table.** In SQL Server the CTE is just a macro: if you reference it three times, its query may be **executed three times**. If an expensive intermediate result is reused, a `#temp` table (with statistics and indexes) is often better.
- 🔀 PostgreSQL 12+: non-recursive CTEs referenced once are inlined; those referenced multiple times are materialized by default; override with `AS MATERIALIZED / NOT MATERIALIZED`. (Before 12, CTEs were always an optimization fence.)
- CTE vs view: a CTE lives for one statement; a view is stored in the database.

### 21.2 Recursive CTE

Structure: **anchor member** `UNION ALL` **recursive member** (refers to the CTE itself). It repeats the recursive part using the previous iteration's rows until no new rows appear.

```sql
-- Org chart with level and path
WITH org AS (
    SELECT emp_id, emp_name, manager_id, 0 AS lvl, CAST(emp_name AS VARCHAR(500)) AS path
    FROM employees WHERE manager_id IS NULL                              -- anchor: the top
    UNION ALL
    SELECT e.emp_id, e.emp_name, e.manager_id, o.lvl + 1, CAST(o.path + ' > ' + e.emp_name AS VARCHAR(500))
    FROM employees e JOIN org o ON e.manager_id = o.emp_id               -- recursive: children of previous level
)
SELECT * FROM org OPTION (MAXRECURSION 100);       -- default limit 100 levels; 0 = unlimited (danger on cycles)

-- Numbers 1..10 and a date series
WITH n AS (SELECT 1 AS num UNION ALL SELECT num + 1 FROM n WHERE num < 10) SELECT num FROM n;
WITH d AS (SELECT CAST('2024-01-01' AS DATE) dt UNION ALL SELECT DATEADD(DAY,1,dt) FROM d WHERE dt < '2024-01-31') SELECT dt FROM d;
```
- 🔀 PostgreSQL/MySQL need the keyword `WITH RECURSIVE`. Oracle can use `CONNECT BY` for hierarchies too.
- ⚠️ Data containing a **cycle** (A manages B manages A) makes recursion infinite until the limit; guard with a path check or `MAXRECURSION`.
- Types of the anchor and recursive columns must match exactly (hence the `CAST` to the same `VARCHAR(500)`).

---

## 22. Window Functions

🧠 Aggregates with `GROUP BY` **collapse** rows. A window function does the calculation **across a set of related rows but keeps every row**. Picture each row looking through a "window" at its neighbours.

```sql
function() OVER (
    [PARTITION BY col, ...]                       -- split rows into independent groups (like GROUP BY, but without collapsing)
    [ORDER BY col [ASC|DESC]]                     -- order inside each partition
    [ROWS | RANGE BETWEEN frame_start AND frame_end]   -- which rows around the current row to include
)
```

### 22.1 Categories

| Category | Functions |
|----------|-----------|
| Ranking | `ROW_NUMBER, RANK, DENSE_RANK, NTILE, PERCENT_RANK, CUME_DIST` |
| Offset / value | `LAG, LEAD, FIRST_VALUE, LAST_VALUE, NTH_VALUE` |
| Aggregate as window | `SUM, AVG, COUNT, MIN, MAX` + `OVER` |

### 22.2 Ranking functions

| Salary | ROW_NUMBER | RANK | DENSE_RANK |
|--------|-----------|------|-----------|
| 100 | 1 | 1 | 1 |
| 100 | 2 | 1 | 1 |
| 90 | 3 | 3 | 2 |
| 80 | 4 | 4 | 3 |

- `ROW_NUMBER`: unique 1,2,3… ties broken **arbitrarily**, so add a tie-breaker in `ORDER BY` to be deterministic.
- `RANK`: ties share a rank, then **gaps** (1,1,3).
- `DENSE_RANK`: ties share a rank, **no gaps** (1,1,2).
- `NTILE(n)`: splits ordered rows into n nearly equal buckets (quartiles = `NTILE(4)`); bucket sizes differ by at most 1.

```sql
SELECT emp_name, dept_id, salary,
  ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn,
  RANK()       OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rk,
  DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS drk,
  NTILE(4)     OVER (ORDER BY salary DESC) AS quartile
FROM employees;
```

### 22.3 LAG / LEAD

```sql
SELECT order_id, order_date, amount,
  LAG(amount, 1, 0) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_amt,   -- (col, offset, default)
  LEAD(amount)      OVER (PARTITION BY customer_id ORDER BY order_date) AS next_amt,
  amount - LAG(amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS change
FROM orders;
```
Without the default, the first row's `LAG` is NULL (nothing before it).

### 22.4 Aggregates as windows and frames

```sql
SELECT order_date, amount,
  SUM(amount) OVER (ORDER BY order_date, order_id ROWS UNBOUNDED PRECEDING)           AS running_total,
  AVG(amount) OVER (ORDER BY order_date, order_id ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_3,
  SUM(amount) OVER ()                                    AS grand_total,
  amount * 100.0 / SUM(amount) OVER ()                   AS pct_of_total,
  SUM(amount) OVER (PARTITION BY customer_id)            AS cust_total
FROM orders;
```

**Frame clause:**

| Frame | Meaning |
|-------|---------|
| `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | start of partition → this row (running total) |
| `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` | centred 3-row window |
| `ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING` | this row → end |

🧠 **ROWS vs RANGE:** `ROWS` counts **physical rows**. `RANGE` works on **values**: all rows with the *same ORDER BY value* (peers) are included together.
⚠️ When you write `ORDER BY` but **no frame**, the default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. With ties in the ORDER BY column, peers get the **same** running total (and it is slower, with a disk-based spool in SQL Server). Specify `ROWS UNBOUNDED PRECEDING` explicitly for running totals.
⚠️ `LAST_VALUE` with the default frame returns the **current row's** value (the frame ends at the current row). Use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`.
⚠️ The "ranking" functions and `LAG/LEAD` ignore frames; they are positional.

```sql
SELECT emp_name, dept_id, salary,
  FIRST_VALUE(emp_name) OVER (PARTITION BY dept_id ORDER BY salary DESC) AS top_earner,
  LAST_VALUE(emp_name)  OVER (PARTITION BY dept_id ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS lowest_earner
FROM employees;
```

### 22.5 Top-N per group, filtering on window results

Window functions run at the `SELECT` step, so they can't appear in `WHERE`:
```sql
SELECT * FROM (
  SELECT e.*, ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC, emp_id) rn FROM employees e
) t WHERE rn <= 2;

-- Snowflake, BigQuery, Databricks, Teradata: QUALIFY filters windows directly
SELECT * FROM employees QUALIFY ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) = 1;
```
Which ranking function for "top-N"? `ROW_NUMBER` = exactly N rows; `RANK`/`DENSE_RANK` = includes ties (may return more than N).

⚡ A window function with `PARTITION BY a ORDER BY b` is fastest when an index on `(a, b)` exists (no sort needed).
🔀 SQL Server `RANGE` frames support only `UNBOUNDED` and `CURRENT ROW` bounds (not `RANGE 5 PRECEDING`).

---

## 23. Built-in Functions

### 23.1 String functions

| Purpose | T-SQL | PostgreSQL / MySQL |
|---------|-------|-------------------|
| Length | `LEN` (ignores trailing spaces), `DATALENGTH` (bytes) | `LENGTH` (🔀 MySQL: bytes), `CHAR_LENGTH` (chars) |
| Case | `UPPER, LOWER` | same |
| Trim | `TRIM, LTRIM, RTRIM` | same |
| Substring | `SUBSTRING(s, start, len)` (1-based; `len` required) | `SUBSTRING`, `SUBSTR` |
| Left / right | `LEFT(s,n), RIGHT(s,n)` | same |
| Find | `CHARINDEX(find, s [,start])`, `PATINDEX('%pat%', s)` | `POSITION(find IN s)`, `STRPOS`, `INSTR` |
| Replace | `REPLACE(s, old, new)` | same |
| Concatenate | `CONCAT`, `CONCAT_WS(sep, ...)`, `+` | `CONCAT`, `\|\|` (PG/Oracle) |
| Pad | `RIGHT('000'+CAST(n AS VARCHAR),3)` | `LPAD, RPAD` |
| Repeat | `REPLICATE` | `REPEAT` |
| Split | `STRING_SPLIT(s, ',')` (2016+; add `enable_ordinal` in 2022) | `STRING_TO_ARRAY`, `SPLIT_PART` |
| Format | `FORMAT(val,'N2')` (slow; CLR-based) | `TO_CHAR` |

```sql
SELECT UPPER(emp_name), LEN(emp_name), LEFT(emp_name,3),
       SUBSTRING(email, CHARINDEX('@',email) + 1, 100) AS domain,
       REPLACE(email,'@co.com','@new.com'), CONCAT(emp_name,' - ',dept_id), TRIM('  hi  ')
FROM employees;
```
⚠️ Positions are **1-based**. `CHARINDEX` returns 0 when not found (not -1).
⚡ `FORMAT()` in SQL Server is notoriously slow; use `CONVERT`/`CAST` on big sets.

### 23.2 Numeric functions
`ABS, CEILING/CEIL, FLOOR, ROUND, POWER, SQRT, SIGN, EXP, LOG, MOD / %, RAND()`. See §7 for rounding and integer-division behaviour. `RAND()` returns the **same value for every row** within one statement unless seeded per row; use `NEWID()`/`CHECKSUM(NEWID())` for per-row randomness (e.g., random sample: `ORDER BY NEWID()`; slow on big tables, use `TABLESAMPLE`).

### 23.3 Date and time functions

| Purpose | T-SQL | PostgreSQL | MySQL |
|---------|-------|-----------|-------|
| Now | `GETDATE(), SYSDATETIME(), GETUTCDATE()` | `NOW()` | `NOW()` |
| Today | `CAST(GETDATE() AS DATE)` | `CURRENT_DATE` | `CURDATE()` |
| Add | `DATEADD(MONTH,1,d)` | `d + INTERVAL '1 month'` | `DATE_ADD(d, INTERVAL 1 MONTH)` |
| Difference | `DATEDIFF(DAY,d1,d2)` | `d2 - d1`, `AGE()` | `DATEDIFF(d2,d1)` (days only) |
| Part | `DATEPART, YEAR(), MONTH(), DAY(), DATENAME` | `EXTRACT(YEAR FROM d)` | `EXTRACT`, `YEAR()` |
| Truncate | `DATETRUNC` (2022) | `DATE_TRUNC('month', d)` | `DATE_FORMAT` |
| End of month | `EOMONTH(d)` | `DATE_TRUNC('month',d) + INTERVAL '1 month - 1 day'` | `LAST_DAY(d)` |
| Format | `FORMAT`, `CONVERT(VARCHAR,d,23)` | `TO_CHAR(d,'YYYY-MM-DD')` | `DATE_FORMAT(d,'%Y-%m-%d')` |
| Validate | `TRY_CONVERT, ISDATE` | | |

```sql
SELECT DATEFROMPARTS(YEAR(order_date), MONTH(order_date), 1) AS month_start, SUM(amount)
FROM orders GROUP BY DATEFROMPARTS(YEAR(order_date), MONTH(order_date), 1);

WHERE order_date >= DATEADD(DAY, -30, CAST(GETDATE() AS DATE))        -- last 30 days, SARGable
```
More traps (boundary counting, DATETIME rounding, literals, time zones) are in §8.

### 23.4 Conversion functions

```sql
CAST(amount AS INT)                       -- ANSI
CONVERT(VARCHAR(10), order_date, 120)     -- T-SQL with style code (120 = yyyy-mm-dd hh:mi:ss, 23 = yyyy-mm-dd)
TRY_CAST('abc' AS INT)                    -- NULL instead of an error (T-SQL 2012+)
TRY_CONVERT(INT, 'abc')
PARSE('12/03/2024' AS DATE USING 'en-GB') -- culture aware (slow)
-- 🔀 PostgreSQL: '123'::INT, TO_DATE('2024-03-01','YYYY-MM-DD'), TO_NUMBER, TO_TIMESTAMP
```
- Implicit conversion silently follows **type precedence** (§5.7) and may kill index usage.
- `TRY_*` returns NULL on failure: perfect for cleaning dirty data (`WHERE TRY_CAST(x AS INT) IS NULL` finds the bad rows).

### 23.5 System and misc functions (T-SQL)

`ISNULL, COALESCE, NULLIF, IIF(cond, a, b), CHOOSE(i, a, b, c), NEWID(), NEWSEQUENTIALID(), @@ROWCOUNT, @@ERROR, @@TRANCOUNT, @@VERSION, DB_NAME(), SUSER_SNAME(), HASHBYTES('SHA2_256', x), CHECKSUM, GREATEST/LEAST (2022+, PG, MySQL)`

---

## 24. CASE Expressions

```sql
-- Searched CASE
SELECT emp_name, salary,
  CASE WHEN salary >= 100000 THEN 'High'
       WHEN salary >= 70000  THEN 'Medium'
       ELSE 'Low' END AS band
FROM employees;

-- Simple CASE (equality only)
SELECT CASE dept_id WHEN 1 THEN 'HR' WHEN 2 THEN 'IT' ELSE 'Other' END FROM employees;

ORDER BY CASE WHEN status = 'Pending' THEN 0 ELSE 1 END, order_date;                   -- custom sort
SELECT SUM(CASE WHEN status='Delivered' THEN amount ELSE 0 END) FROM orders;             -- conditional aggregation
UPDATE employees SET salary = CASE WHEN dept_id = 2 THEN salary * 1.10 ELSE salary * 1.05 END;
```
- Branches are tested **top to bottom** and the **first true** wins; no `ELSE` → NULL.
- All branches must be **type-compatible**; the result type follows data type precedence (mixing `INT` and `VARCHAR` branches can error with "Conversion failed" in the numeric direction).
- `CASE` is an **expression** (returns a value), not a statement; it is the **only** reliable way to control evaluation order inside a query (§13.4).
- Simple `CASE x WHEN NULL` never matches, because it compares with `=`; write `WHEN x IS NULL`.

---

## 25. PIVOT, UNPIVOT and Conditional Aggregation

🧠 **Pivot** turns row values into columns (long → wide). **Unpivot** does the reverse (wide → long).

```sql
-- Portable and preferred: conditional aggregation
SELECT customer_id,
  SUM(CASE WHEN status='Delivered' THEN amount END) AS delivered,
  SUM(CASE WHEN status='Pending'   THEN amount END) AS pending,
  SUM(CASE WHEN status='Cancelled' THEN amount END) AS cancelled
FROM orders GROUP BY customer_id;

-- T-SQL / Oracle PIVOT
SELECT * FROM (SELECT customer_id, status, amount FROM orders) src
PIVOT (SUM(amount) FOR status IN ([Delivered],[Pending],[Cancelled])) p;

-- UNPIVOT (columns → rows)
SELECT customer_id, status, amount FROM pivoted_tbl
UNPIVOT (amount FOR status IN ([Delivered],[Pending],[Cancelled])) u;     -- note: UNPIVOT drops NULL rows

-- Portable unpivot
SELECT customer_id, v.status, v.amount FROM pivoted_tbl
CROSS APPLY (VALUES ('Delivered', delivered), ('Pending', pending), ('Cancelled', cancelled)) v(status, amount);
```
- Pivot columns must be **known when you write the query**. For dynamic columns, build the statement with **dynamic SQL** (§34.3) and `QUOTENAME()`.
- In a pivot, every column not mentioned in `FOR`/aggregate becomes an implicit `GROUP BY`: select only the columns you want in the source subquery.
- Presentation-style pivots are often better done in the BI tool (Power BI, Excel).

---

## 26. Views and Materialized Views

### 26.1 View
A **stored SELECT**: a virtual table, no data stored. Using a view is replaced by its query text during optimization (it is *expanded*, like a macro).

```sql
CREATE VIEW dbo.vw_emp_dept AS
SELECT e.emp_id, e.emp_name, d.dept_name FROM dbo.employees e JOIN dbo.departments d ON e.dept_id = d.dept_id;
SELECT * FROM dbo.vw_emp_dept WHERE dept_name = 'IT';        -- the filter is pushed into the view's query
ALTER VIEW dbo.vw_emp_dept AS ...;    DROP VIEW dbo.vw_emp_dept;
```
**Uses:** simplify complex joins, **security** (expose only some columns/rows), abstraction (change tables, keep the view's shape), backward compatibility.
**Rules/gotchas:**
- `ORDER BY` not allowed in a view without `TOP`/`OFFSET` (and still isn't guaranteed for the outer query).
- `SELECT *` in a view is **frozen at creation**: new table columns won't appear until the view is refreshed (`sp_refreshview`) or altered.
- **Updatable views:** simple single-table views (no aggregates/DISTINCT/GROUP BY/UNION) accept INSERT/UPDATE/DELETE. `WITH CHECK OPTION` rejects changes that would make the row disappear from the view. Complex views can be made updatable with `INSTEAD OF` triggers.
- `WITH SCHEMABINDING` forbids changing underlying tables in ways that break the view; required for **indexed views**.
- Stacking views on views on views leads to enormous hidden query plans. Keep layers shallow.

### 26.2 Materialized / indexed views
Store the **result physically** so reads are instant, at the cost of storage and refresh work.

```sql
-- PostgreSQL / Oracle
CREATE MATERIALIZED VIEW mv_sales AS SELECT ...;
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales;       -- needs a unique index; readers are not blocked

-- SQL Server indexed view: automatically maintained on every write to the base table
CREATE VIEW dbo.vw_sales WITH SCHEMABINDING AS
SELECT customer_id, SUM(amount) AS total, COUNT_BIG(*) AS cnt FROM dbo.orders GROUP BY customer_id;   -- COUNT_BIG(*) is required with GROUP BY
CREATE UNIQUE CLUSTERED INDEX ix_vw_sales ON dbo.vw_sales(customer_id);
```
Note: `SUM(amount)` over a nullable column additionally requires `ISNULL(...)`; many restrictions apply (no outer joins, no subqueries, deterministic only).

| View | Materialized view |
|------|-------------------|
| No storage | Stores data |
| Always current | Stale until refreshed (PG/Oracle); always current for SQL Server indexed views |
| Same cost as the underlying query | Fast reads; slower writes (indexed view) or scheduled refresh |

---

## 27. Indexes

🧠 **An index is like the index at the back of a textbook:** a small, sorted lookup structure that tells you which pages to open, so you do not read the whole book. The price is that the index must be updated every time the book changes.

### 27.1 The B-tree structure

```text
                    [ Root page ]                  1 page
                 /       |        \
        [Intermediate pages]    …                  (levels)
         /     |     \
   [Leaf pages: sorted keys + pointers or full rows]   (the bulk of the index)
```
- Lookup cost is **O(log n)**: about 3–4 page reads can find one row among **hundreds of millions** (each 8 KB page holds hundreds of keys, so fan-out is large). A scan would read all pages.
- Leaf pages are **linked**, so range scans (`BETWEEN`, `ORDER BY`) read sequentially after one seek.
- 🔀 PostgreSQL B-tree is the default; also **Hash, GIN** (arrays/JSONB/full text), **GiST** (geometry, ranges), **BRIN** (huge append-only tables, tiny index), **SP-GiST**. SQL Server: rowstore B-tree, **columnstore**, spatial, XML, full-text.

### 27.2 Types

| Type | Description |
|------|-------------|
| **Clustered** | **Is** the table: leaf pages hold the full rows, sorted by the key. **One per table.** Created by the PK by default (SQL Server). Choose a key that is **narrow, unique, static, ever-increasing** |
| **Non-clustered** | Separate structure: key columns + a **row locator** (the clustering key, or RID for heaps). Up to 999 per table (SQL Server). |
| **Unique** | Enforces uniqueness |
| **Composite (multi-column)** | Sorted by col1, then col2 within col1... **Column order matters** |
| **Covering** | Includes every column a query needs (key + `INCLUDE`) so no lookup to the table |
| **Filtered / partial** | Indexes only a subset of rows (`WHERE status = 'Pending'`): smaller and cheaper |
| **Columnstore** | Data stored **by column**, heavily compressed in segments of ~1 million rows: 10x compression and fast aggregations. For data warehouses and analytics |
| **Full-text** | Word/language-aware search |
| Hash, bitmap | Engine-specific (Oracle bitmap for low-cardinality DW columns; MySQL MEMORY hash) |

```sql
CREATE CLUSTERED INDEX ix_orders_date ON orders(order_date);
CREATE NONCLUSTERED INDEX ix_orders_cust ON orders(customer_id) INCLUDE (amount, status);
CREATE UNIQUE NONCLUSTERED INDEX ux_emp_email ON employees(email) WHERE email IS NOT NULL;  -- filtered unique: allows many NULLs (SQL Server workaround)
CREATE NONCLUSTERED INDEX ix_pending ON orders(order_date) WHERE status = 'Pending';
CREATE NONCLUSTERED COLUMNSTORE INDEX ncci_orders ON orders(order_date, amount);
DROP INDEX ix_orders_cust ON orders;
ALTER INDEX ix_orders_cust ON orders REBUILD;       -- new copy: removes fragmentation, updates stats (fullscan)
ALTER INDEX ix_orders_cust ON orders REORGANIZE;    -- online, light-weight, leaf-level only
```

### 27.3 What the plan shows

| Operator | Meaning | Cost shape |
|----------|---------|-----------|
| **Table / Clustered Index Scan** | Reads every page | O(n); fine for small tables or when most rows are needed |
| **Index Seek** | B-tree navigation to specific key range | O(log n + rows returned) ✅ |
| **Key Lookup / RID Lookup** | Index found the row, but the query needs more columns → hop to the base table (one random read per row) | Cheap for a few rows; **disastrous for thousands** |
| **Index Scan** | Reads the whole index (narrower than the table) | Sometimes better than scanning the table |

🧠 The optimizer weighs lookups against scans. Each lookup is a random page read, so the **tipping point** is when the lookups would touch roughly a quarter to a third of the table's *pages*; that is often well under 5% of the *rows*, and beyond it the optimizer prefers a scan. That's why a non-covering index on a low-selectivity column is often ignored, and why **covering** indexes are powerful.

### 27.4 Composite index design

An index on `(a, b, c)` is a phone book sorted by *last name, then first name, then city*.

- **Leftmost prefix rule:** it supports predicates on `a`, `a,b`, `a,b,c`, but **not** `b` alone (you can't look up by first name in a book sorted by last name).
- **Equality columns first, range column last:** `WHERE a = 1 AND b = 2 AND c > 10` → `(a, b, c)`. After a range, the next column can't be used to seek.
- Put the more **selective** (more distinct values) column first only when predicates are all equality; otherwise follow the equality-first rule.
- The same index serves `ORDER BY a, b` without a sort. Direction (`ASC/DESC`) matters for mixed sort directions.
- `INCLUDE` columns are stored only in the leaf level, not in the sort key (so key size stays small).

### 27.5 SARGability (Search ARGument ABLE)

A predicate is SARGable when the engine can use it to **seek** into the index. Rule: **keep the indexed column "naked" on one side.**

| ❌ Non-SARGable | ✅ SARGable rewrite |
|----------------|--------------------|
| `WHERE YEAR(order_date) = 2024` | `WHERE order_date >= '20240101' AND order_date < '20250101'` |
| `WHERE LEFT(name,1) = 'A'` | `WHERE name LIKE 'A%'` |
| `WHERE salary * 12 > 100000` | `WHERE salary > 100000 / 12.0` |
| `WHERE ISNULL(col,0) = 5` | `WHERE col = 5` (or `col = 5 OR col IS NULL` as needed) |
| `WHERE col LIKE '%abc'` | Full-text, trigram (`pg_trgm`), or a persisted reversed column |
| `WHERE CONVERT(VARCHAR, id) = '5'` | `WHERE id = 5` |
| `WHERE col1 = @p OR @p IS NULL` ("catch-all") | Dynamic SQL, or `OPTION (RECOMPILE)` |

### 27.6 Design guidance

- Index columns used in `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY`, and **FK columns**.
- **Selectivity** = distinct values ÷ rows. High selectivity (email, id) → great; low selectivity (gender, bit flag) → rarely helpful alone (but good as a *filtered* index or as a trailing column).
- Every index **slows writes** (each INSERT/UPDATE/DELETE must maintain every index), uses disk and memory, and lengthens backups and maintenance. Balance **read vs write** workload: OLTP wants few lean indexes; reporting/DW wants many.
- A good PK: `INT`/`BIGINT IDENTITY`. Bad PK: random GUID, wide strings.
- Remove **unused** and **duplicate/overlapping** indexes (`(a)` is redundant if `(a,b)` exists, with exceptions).
- Updating an indexed column costs more than updating a non-indexed one.

### 27.7 Fragmentation and fill factor

- **Page split**: a row insert/grow hits a full page → the page is split in two, each about half full. Result: **internal fragmentation** (empty space) and **logical fragmentation** (pages out of physical order → slower scans on spinning disks, minor on SSD).
- **Fill factor** (e.g., 90) leaves free space on leaf pages at rebuild time to absorb inserts in the middle of the index; default 100 is fine for ever-increasing keys.
- Rule of thumb (from SQL Server docs): fragmentation **5–30% → REORGANIZE**, **>30% → REBUILD**, ignore for indexes under ~1,000 pages. On SSDs the *page density* matters more than ordering. Update statistics regularly.
- Maintenance: `sys.dm_db_index_physical_stats`, `sys.dm_db_index_usage_stats` (is the index ever read?), `sys.dm_db_missing_index_details` (suggestions, **not** commands: review for overlaps).

---

## 28. Statistics, Execution Plans and Query Optimization

### 28.1 How the optimizer thinks

- Parse → bind → **optimize (cost-based)** → execute. The optimizer does **not** try every plan; it explores many, scores them with a **cost model** (estimated I/O + CPU), and stops when "good enough" (or when time budget runs out).
- Its inputs: **statistics** (histograms of value distribution per indexed/auto-created column: up to 200 steps), row counts, available indexes, constraints (trusted FK/CHECK let it eliminate joins), and the query text.
- **Cardinality estimate** = predicted rows at each operator. Everything downstream (join type, memory grant, parallelism) hinges on it. **Most slow queries are caused by bad estimates, not by missing indexes.**
- **Stale statistics** (data changed a lot since last update) → wrong estimates → wrong plans. `UPDATE STATISTICS dbo.orders;` or `EXEC sp_updatestats;`. Auto-update triggers after roughly 20% of rows change for big tables (lower threshold with newer compat levels), which is late on huge tables.
- **Plan cache:** compiled plans are reused for the same query text/parameters shape. Benefit: no recompilation. Risk: **parameter sniffing** (§28.5).

### 28.2 Reading execution plans

```sql
SET STATISTICS IO ON;  SET STATISTICS TIME ON;      -- T-SQL: logical reads and CPU/elapsed ms
-- SSMS: "Include Actual Execution Plan" (Ctrl+M)
EXPLAIN ANALYZE SELECT ...;                         -- PostgreSQL (executes the query!)
EXPLAIN SELECT ...;                                 -- MySQL / PG estimated plan
```
Read **right to left, top to bottom** (data flows from the right leaf operators leftwards). Look for:

1. **Estimated vs actual rows** big mismatch → stats/sniffing problem.
2. **Scans on large tables** when you expected seeks.
3. **Key lookups** with a high row count.
4. **Sort / Hash Match spills to tempdb** (yellow warning icon).
5. **Thick arrows** (many rows) early in the plan. Filter earlier.
6. **Implicit conversion** warnings (§5.7).
7. **Parallelism** operators (Gather Streams) on small queries, or skew.
8. **Missing index** hint (suggestion only).
9. Operator cost % is the *estimated* cost even in the "actual" plan; trust the row counts and logical reads more than the percentages.

### 28.3 The optimization checklist

1. Select only the columns you need (no `SELECT *`).
2. Keep predicates **SARGable**; no functions on indexed columns.
3. Create the **right indexes** (join/filter/sort columns; covering where it pays).
4. Match **data types** across joins and parameters.
5. `EXISTS`/`NOT EXISTS` over `IN`/`NOT IN` where NULLs or large lists are involved.
6. `UNION ALL` instead of `UNION` when duplicates are impossible or acceptable.
7. Prefer **set-based** logic over cursors/WHILE loops/row-by-row functions.
8. Avoid scalar UDFs on big sets (use inline table-valued functions; SQL Server 2019 can inline some scalar UDFs).
9. Avoid unneeded `DISTINCT` / `ORDER BY` (sorting is expensive).
10. Keep **statistics** current.
11. **Batch** big modifications.
12. Break monster queries into steps with `#temp` tables when the optimizer's estimates compound badly.
13. Avoid `OR` across different columns (try `UNION ALL`) and catch-all `@p IS NULL OR col = @p` patterns.
14. Use **partitioning** and **columnstore** for large analytic tables.
15. Limit rows early: `TOP`, pagination, filters before joins.
16. Check **memory grants** and tempdb spills.

### 28.4 Hints (last resort)
`OPTION (RECOMPILE)`, `OPTION (MAXDOP 1)`, `OPTION (OPTIMIZE FOR (@p = 1))`, `WITH (INDEX(ix))`, `OPTION (HASH JOIN)`, `WITH (NOLOCK)`. Hints freeze decisions that may become wrong as data grows; understand *why* the optimizer is wrong first.

### 28.5 Parameter sniffing

The first time a parameterized query runs, the optimizer **sniffs** the parameter value and builds a plan tuned to it; that plan is cached and reused for *all* values.

> Example: `WHERE customer_id = @id`. First call uses a customer with 3 orders → plan: index seek + lookups. Next call uses a customer with 3 million orders → same plan → millions of lookups → very slow. Same query, same code, "sometimes slow".

Fixes: `OPTION (RECOMPILE)` (fresh plan every time; compile cost), `OPTIMIZE FOR UNKNOWN`/local variables (average plan), `OPTIMIZE FOR (@p = x)`, split into separate procedures for skewed cases, Query Store plan forcing, **Parameter Sensitive Plan optimization** (SQL Server 2022).

### 28.6 Monitoring (SQL Server)
**Query Store** (history of plans and runtimes; regression detection), DMVs: `sys.dm_exec_query_stats`, `sys.dm_exec_requests`, `sys.dm_os_wait_stats` (what the server is waiting on: I/O, locks, CPU), `sys.dm_db_missing_index_details`; Extended Events; Activity Monitor; `sp_who2`; community tools `sp_WhoIsActive`, Brent Ozar's `sp_Blitz` family. 🔀 PostgreSQL: `pg_stat_statements`, `EXPLAIN (ANALYZE, BUFFERS)`, `auto_explain`; MySQL: `EXPLAIN ANALYZE`, slow query log, Performance Schema.

---

## 29. Stored Procedures, Functions and Triggers

### 29.1 Stored procedure

A **named, stored batch of SQL** with parameters, control flow and error handling, compiled and cached.

```sql
CREATE OR ALTER PROCEDURE dbo.usp_GetEmployees
    @dept_id    INT,
    @min_salary DECIMAL(10,2) = 0,        -- default value
    @total      INT OUTPUT                -- output parameter
AS
BEGIN
    SET NOCOUNT ON;                       -- suppress "N rows affected" messages (less network chatter)
    SELECT * FROM dbo.employees WHERE dept_id = @dept_id AND salary >= @min_salary;
    SET @total = @@ROWCOUNT;
END;
GO
DECLARE @cnt INT;
EXEC dbo.usp_GetEmployees @dept_id = 2, @total = @cnt OUTPUT;
SELECT @cnt;
```
**Benefits:** plan reuse; **security** (grant `EXECUTE` without table access: *ownership chaining*); less network traffic; encapsulated business logic; **parameterized → protects from SQL injection**; one place to fix.
**Costs:** logic in the DB is harder to version/test, scales on the (expensive) database server.
It **can** do DML, DDL, transactions, TRY/CATCH, dynamic SQL, return several result sets and a return code (`RETURN` int).

### 29.2 User-defined functions

| Type | Returns | Notes |
|------|---------|-------|
| **Scalar UDF** | One value | Called **per row**: historically a performance killer (no parallelism, hidden cost). SQL Server 2019+ can inline simple ones |
| **Inline TVF (iTVF)** | Table from a single `SELECT` | Expanded into the calling query like a parameterized view ✅ best |
| **Multi-statement TVF** | Table built with statements | Optimizer guesses 1 row (100 rows from SQL Server 2014 on) → poor plans; avoid |

```sql
CREATE FUNCTION dbo.fn_Annual(@sal DECIMAL(10,2)) RETURNS DECIMAL(12,2) AS BEGIN RETURN @sal * 12; END;
CREATE FUNCTION dbo.fn_EmpByDept(@dept INT) RETURNS TABLE AS RETURN (SELECT * FROM dbo.employees WHERE dept_id = @dept);
SELECT dbo.fn_Annual(salary) FROM employees;
SELECT * FROM dbo.fn_EmpByDept(2);
```

### 29.3 Procedure vs function

| Stored procedure | Function |
|------------------|----------|
| Return optional (int code, result sets, OUTPUT params) | **Must** return a value or table |
| Can modify data, DDL, transactions | Cannot modify persistent data (T-SQL) |
| `EXEC proc` | Used **inside** `SELECT/WHERE/FROM` |
| TRY/CATCH allowed | TRY/CATCH not allowed |
| Can call functions and procedures | Cannot call procedures |
| Can use temp tables | Cannot (table variables only) |

### 29.4 Triggers

Code that runs **automatically** when data changes.

| Kind | When |
|------|------|
| `AFTER` (`FOR`) | After the DML succeeds and constraints pass |
| `INSTEAD OF` | Replaces the DML (typical for updatable views) |
| DDL trigger | On CREATE/ALTER/DROP |
| Logon trigger | On login |

```sql
CREATE TRIGGER trg_emp_audit ON dbo.employees AFTER UPDATE AS
BEGIN
    SET NOCOUNT ON;
    INSERT INTO dbo.emp_audit (emp_id, old_salary, new_salary, changed_on)
    SELECT d.emp_id, d.salary, i.salary, SYSDATETIME()
    FROM deleted d JOIN inserted i ON d.emp_id = i.emp_id
    WHERE d.salary <> i.salary;
END;
```
- Pseudo-tables **`inserted`** (new values) and **`deleted`** (old values; for UPDATE both exist).
- ⚠️ Triggers fire **once per statement**, not once per row: `inserted` may contain 1 or 10,000 rows. Write set-based code; never assume a single row.
- ⚠️ Hidden logic: invisible to the app developer, slows DML, can cascade. Use for **auditing** and enforcement that cannot be declarative; prefer constraints, temporal tables, or CDC where possible.
- 🔀 PostgreSQL triggers call a **trigger function** and use `NEW`/`OLD` row variables, and can be `FOR EACH ROW` or `FOR EACH STATEMENT`.

### 29.5 T-SQL control flow

```sql
DECLARE @i INT = 1;
WHILE @i <= 3 BEGIN PRINT @i; SET @i += 1; END;
IF EXISTS (SELECT 1 FROM employees WHERE salary > 100000) PRINT 'Yes'; ELSE PRINT 'No';

-- Cursor (row-by-row, avoid when a set-based solution exists)
DECLARE @id INT;
DECLARE c CURSOR LOCAL FAST_FORWARD FOR SELECT emp_id FROM employees;
OPEN c; FETCH NEXT FROM c INTO @id;
WHILE @@FETCH_STATUS = 0 BEGIN /* work */ FETCH NEXT FROM c INTO @id; END;
CLOSE c; DEALLOCATE c;
```
🧠 Think in **sets**, not loops: a cursor processing 1 million rows issues 1 million tiny operations; the equivalent `UPDATE ... JOIN` is a single plan.

---

## 30. Transactions, ACID, Isolation and Locking

### 30.1 Transaction

A **transaction** is a logical unit of work that either **fully happens or not at all**.

```sql
BEGIN TRANSACTION;
    UPDATE accounts SET bal = bal - 500 WHERE id = 1;
    UPDATE accounts SET bal = bal + 500 WHERE id = 2;
COMMIT;                -- make permanent        (or ROLLBACK; to undo everything)
SAVE TRANSACTION sp1;  ROLLBACK TRANSACTION sp1;     -- partial rollback (T-SQL). PG/Oracle: SAVEPOINT sp1; ROLLBACK TO sp1;
```
- **Autocommit** (default in SQL Server/PG/MySQL): each statement is its own transaction. `BEGIN TRAN` turns that off until `COMMIT`/`ROLLBACK`. 🔀 Oracle starts a transaction implicitly and requires `COMMIT`.
- SQL Server **"nested transactions" are an illusion**: an inner `COMMIT` only decrements `@@TRANCOUNT`; a `ROLLBACK` anywhere rolls back **everything**. Use savepoints for partial undo.
- Keep transactions **short**: they hold locks and block the log from truncating.

### 30.2 ACID

| Property | Meaning | Mechanism |
|----------|---------|-----------|
| **A**tomicity | All or nothing | Undo information in the transaction log |
| **C**onsistency | Moves from one valid state to another (constraints, triggers hold) | Constraints + the application's own rules |
| **I**solation | Concurrent transactions don't see each other's half-done work | Locks or row versioning (MVCC) |
| **D**urability | After commit, survives a crash | **Write-ahead logging (WAL)**: log records are flushed to disk **before** commit returns |

🧠 Consistency is partly *your* job: the DB guarantees the constraints you declared, not the business rules you didn't.

### 30.3 Concurrency problems

| Problem | Description | Example |
|---------|-------------|---------|
| **Dirty read** | Reading another transaction's **uncommitted** data | T1 updates salary, T2 reads it, T1 rolls back: T2 saw a value that never existed |
| **Non-repeatable read** | Re-reading the same row gives a different value | T1 reads salary twice; T2 updates and commits in between |
| **Phantom read** | Re-running a query returns a different **set** of rows | T1 counts orders; T2 inserts one and commits; T1 counts again |
| **Lost update** | Two transactions read, then both write, the 2nd overwrites the 1st | Two users edit the same record |
| **Write skew** | Two transactions each check a rule using data the other changes (snapshot isolation allows this) | Two doctors both go off-call because each saw the other on-call |

### 30.4 Isolation levels

| Level | Dirty | Non-repeatable | Phantom | Notes |
|-------|:----:|:-------------:|:------:|-------|
| READ UNCOMMITTED | possible | possible | possible | = `NOLOCK`. 🔀 PostgreSQL treats it as READ COMMITTED |
| **READ COMMITTED** | no | possible | possible | **Default** in SQL Server, PostgreSQL, Oracle |
| REPEATABLE READ | no | no | possible* | Default in **MySQL InnoDB** (*InnoDB's gap locks and PG's snapshot prevent most phantoms) |
| SERIALIZABLE | no | no | no | Highest; key-range locks (SQL Server), SSI (PostgreSQL) |
| SNAPSHOT (SQL Server) | no | no | no | Row versioning; sees data as of transaction start; detects update conflicts |

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM employees WITH (NOLOCK);                       -- = READ UNCOMMITTED for this table
ALTER DATABASE company_db SET READ_COMMITTED_SNAPSHOT ON;    -- RCSI: READ COMMITTED using row versions: readers don't block writers
ALTER DATABASE company_db SET ALLOW_SNAPSHOT_ISOLATION ON;   -- allows SET TRANSACTION ISOLATION LEVEL SNAPSHOT
```
⚠️ **`NOLOCK` is not "faster for free".** It can return uncommitted data, **miss rows or read rows twice** (during page splits), and even throw errors. Use RCSI if the real problem is readers blocking writers.
🧠 Higher isolation = more correctness, less concurrency (more blocking/deadlocks). Row versioning trades **tempdb/version-store space** (SQL Server) or **dead tuples + VACUUM** (PostgreSQL) for non-blocking reads.

### 30.5 Locking (pessimistic concurrency)

| Lock | Purpose |
|------|---------|
| **Shared (S)** | Held while reading; compatible with other S |
| **Exclusive (X)** | For writes; incompatible with everything |
| **Update (U)** | "I intend to update": prevents the classic S+S → X conversion deadlock |
| **Intent (IS, IX, SIX)** | Signals at table/page level that finer locks exist below (cheap compatibility checks) |
| **Schema (Sch-S / Sch-M)** | Protects metadata; DDL takes Sch-M, blocks all |
| **Key-range** | Used by SERIALIZABLE to prevent phantoms |

- **Granularity:** row → page → table. **Lock escalation:** about **5,000 locks** held by one statement on one object → promoted to a table lock (fewer locks to track, but more blocking).
- **Blocking:** session B waits for a lock session A holds. Normal and temporary. Find the head blocker (`sys.dm_exec_requests`, `sp_WhoIsActive`).
- **Deadlock:** A holds X on row 1 and waits for row 2; B holds row 2 and waits for row 1. Nobody can proceed, so the engine picks a **victim** (cheapest to roll back) and kills it (SQL Server error **1205**; PG `40P01`). Prevent: access tables/rows in the **same order**, keep transactions short, index so fewer rows get locked, use RCSI/snapshot, and **retry** on 1205 in the application.
- **Optimistic concurrency:** don't lock while the user thinks; check a **version** at write time.
```sql
UPDATE employees SET salary = 95000, ...
WHERE emp_id = 2 AND row_ver = @row_ver_I_read;     -- row_ver is ROWVERSION; if @@ROWCOUNT = 0 someone else changed it
```
- **MVCC** (PostgreSQL, Oracle, MySQL InnoDB, SQL Server with RCSI/SNAPSHOT): each write creates a new row version; each reader sees a consistent snapshot; **readers don't block writers and writers don't block readers** (writers still block writers).
- `SELECT ... FOR UPDATE` (PG/MySQL/Oracle) / `WITH (UPDLOCK, ROWLOCK)` (T-SQL) = explicit pessimistic row lock: used for job-queue patterns (`FOR UPDATE SKIP LOCKED`, `READPAST`).

---

## 31. Transaction Log, Recovery and Backups

🧠 The data files are updated lazily in memory; **the log is what makes commits durable and recovery possible**.

- **Write-ahead logging:** a change is written to the **log** first (sequentially, fast), and only later flushed to the data file at a **checkpoint**. Commit returns once the log record is safely on disk.
- **Crash recovery** on restart: **redo** committed changes not yet in data files, **undo** uncommitted ones.
- Each log record has an **LSN** (log sequence number); backups and replication are LSN-based.

### 31.1 Recovery models (SQL Server)

| Model | Log behaviour | Point-in-time restore |
|-------|---------------|----------------------|
| **SIMPLE** | Log truncated automatically at checkpoints | No (only to the last full/differential backup) |
| **FULL** | Log kept until you take a **log backup** | Yes (to any moment, e.g., just before a bad `DELETE`) |
| **BULK_LOGGED** | Like FULL but bulk ops minimally logged | Limited |

⚠️ **Classic outage:** database in FULL recovery, nobody takes log backups → the log grows until the disk is full → database stops accepting writes. Another cause of log growth: a **long-open transaction** (nothing after it can be truncated).

### 31.2 Backup types and terms

| Type | Contains | Notes |
|------|----------|-------|
| **Full** | Whole database (data + enough log) | Baseline |
| **Differential** | Pages changed since last full | Faster restore than replaying many logs |
| **Transaction log** | Log since last log backup | Enables point-in-time recovery; typical every 5–15 min |
| Copy-only | Full/log that doesn't disturb the chain | For ad-hoc copies |

Restore order: **last full → latest differential → all subsequent log backups** (`RESTORE ... WITH NORECOVERY` for all but the last, then `RECOVERY`).
- **RPO** (Recovery Point Objective): how much data loss is acceptable (drives log backup frequency).
- **RTO** (Recovery Time Objective): how long downtime is acceptable (drives HA design).
- A backup you have **never restored** is a hope, not a backup. Test restores. Run `DBCC CHECKDB` regularly to detect corruption.

### 31.3 High availability (vocabulary)
Failover clustering (shared storage), **Always On Availability Groups** (log-based replication with readable secondaries), log shipping, replication (transactional/merge/snapshot), backups to cloud. 🔀 PostgreSQL: streaming (WAL) replication, logical replication, Patroni; MySQL: binlog replication, Group Replication; Oracle: Data Guard, RAC. **Replication is not a backup**: a bad `DELETE` replicates instantly.

---

## 32. Normalization and Denormalization

🧠 **Normalization** organizes data so each fact is stored **once**, eliminating redundancy and the **anomalies** that come with it:

- **Insert anomaly:** can't record a new department until it has an employee.
- **Update anomaly:** a customer's city is stored on 500 order rows; change it in 499 and the data contradicts itself.
- **Delete anomaly:** deleting the last order of a customer also erases the only record of the customer.

### 32.1 Normal forms (with the memory aid)

> *"Every non-key attribute must provide a fact about **the key** (1NF), **the whole key** (2NF), and **nothing but the key** (3NF), so help me Codd."*

| NF | Rule | Violation example → fix |
|----|------|-------------------------|
| **1NF** | Atomic (indivisible) values, no repeating groups or lists, each row identifiable | `phones = '98480,98481'` → separate `phone` rows in a child table |
| **2NF** | 1NF + **no partial dependency** (relevant only for composite keys): non-key columns depend on the **whole** key | `OrderItems(order_id, product_id, product_name, qty)`: `product_name` depends only on `product_id` → move to `Products` |
| **3NF** | 2NF + **no transitive dependency** (non-key → non-key) | `Orders(order_id, cust_id, cust_city)`: `cust_city` depends on `cust_id` → move to `Customers` |
| **BCNF** | Every determinant is a candidate key (a stricter 3NF) | `Teaching(student, subject, teacher)` where each teacher teaches one subject: `teacher → subject` but `teacher` isn't a key → split |
| **4NF** | No independent **multi-valued** dependencies in one table | `Person(name, skill, language)` stores every skill × language combination → split into two tables |
| **5NF** | No join dependencies that aren't implied by keys | Rare in practice |

Functional dependency `X → Y` means: if two rows have the same X they must have the same Y.
In practice, OLTP designs aim for **3NF/BCNF**.

### 32.2 Denormalization
Deliberately adding redundancy for **read speed**: pre-joined/flattened tables, duplicated columns, summary tables. Used in reporting and warehouses. Costs: more storage, update complexity, risk of inconsistency; mitigate with ETL/refresh jobs, triggers or constraints.
🎯 *"Normalize for correctness, denormalize for measured performance, and document why."*

---

## 33. Data Modeling and Warehousing

### 33.1 OLTP vs OLAP

| | OLTP | OLAP |
|--|------|------|
| Purpose | Run the business: orders, payments | Analyse the business: trends, KPIs |
| Design | Normalized (3NF) | Denormalized (star/snowflake) |
| Queries | Short, many, by key, touch few rows | Long, aggregate millions of rows |
| Data | Current | Historical, integrated from many sources |
| Indexes | Few, B-tree | Columnstore, partitions |
| Optimized for | Writes + point reads | Large scans + aggregation |

### 33.2 Dimensional modeling (Kimball)

| Term | Meaning |
|------|---------|
| **Fact table** | Numeric measurements of business events (sales amount, RMR, cancellations) + FKs to dimensions. Tall, narrow, billions of rows |
| **Dimension table** | Descriptive context (customer, product, date, plan). Wide, shorter, text attributes |
| **Grain** | What exactly **one row** of the fact means ("one subscription per month"). **Decide first**; mixing grains causes wrong totals |
| **Star schema** | Fact in the middle, denormalized dimensions around it (one join per dimension) |
| **Snowflake schema** | Dimensions normalized into sub-dimensions (less redundancy, more joins) |
| **Constellation** | Several facts sharing dimensions |
| **Conformed dimension** | One dimension shared across facts (Date, Customer) so metrics can be compared |
| **Degenerate dimension** | A key stored in the fact with no table of its own (invoice number) |
| **Junk dimension** | Combines unrelated low-cardinality flags into one small dimension |
| **Role-playing dimension** | One physical dimension used several times (Order Date, Ship Date) |
| **Date dimension** | Pre-built calendar table (day, week, month, fiscal period, holiday flag); always build one |
| **Surrogate keys** | Meaningless integer keys in dimensions: insulate the warehouse from source-system key changes and enable SCD2 |

**Fact types:** **Transactional** (one row per event), **Periodic snapshot** (one row per entity per period, e.g., monthly balance), **Accumulating snapshot** (one row per process instance, updated as milestones happen), **Factless fact** (records that an event occurred, with no numeric measure).
**Measure additivity:** **Additive** (sum across all dimensions: sales), **Semi-additive** (sum across some, not time: account balance, headcount; use average or last value over time), **Non-additive** (ratios, percentages: recompute from numerator/denominator, never sum).

### 33.3 Slowly Changing Dimensions (SCD)

How to handle a dimension attribute that changes over time (customer moves city, plan changes).

| Type | Behaviour | History |
|------|-----------|---------|
| 0 | Never changes (original value kept) | n/a |
| **1** | **Overwrite** | None |
| **2** | **New row** per change with `eff_start`, `eff_end`, `is_current` and a **new surrogate key** | **Full history** ✅ (facts keep pointing to the version that was true then) |
| 3 | Extra column for previous value | One step back |
| 4 | Current table + separate history table | Full |
| 6 | Hybrid of 1 + 2 + 3 | Full + current-value column on each row |

```sql
CREATE TABLE dim_customer (
  cust_sk   INT IDENTITY PRIMARY KEY,   -- surrogate key (what facts reference)
  cust_id   INT NOT NULL,               -- business/natural key
  city      VARCHAR(50),
  eff_start DATE NOT NULL, eff_end DATE NOT NULL DEFAULT '9999-12-31', is_current BIT NOT NULL
);

-- SCD2 load: (1) expire changed rows   (2) insert new versions (new + changed customers)
UPDATE d SET eff_end = CAST(GETDATE() AS DATE), is_current = 0
FROM dim_customer d JOIN stg_customer s ON d.cust_id = s.cust_id
WHERE d.is_current = 1 AND d.city <> s.city;      -- ⚠️ add NULL-safe comparison if city can be NULL

INSERT INTO dim_customer (cust_id, city, eff_start, eff_end, is_current)
SELECT s.cust_id, s.city, CAST(GETDATE() AS DATE), '9999-12-31', 1
FROM stg_customer s
LEFT JOIN dim_customer d ON d.cust_id = s.cust_id AND d.is_current = 1
WHERE d.cust_id IS NULL;     -- new customers, plus changed ones (their old row was just expired, so no current row remains)

-- Point-in-time join of a fact to the dimension version valid on the event date
SELECT f.*, d.city FROM fact_sales f
JOIN dim_customer d ON d.cust_id = f.cust_id AND f.sale_date >= d.eff_start AND f.sale_date < d.eff_end;
```
(Best practice: resolve the surrogate key **at load time** and store `cust_sk` in the fact, instead of a range join at query time.)

### 33.4 Warehouse process vocabulary
- **ETL** (transform before load, on a staging server) vs **ELT** (load raw into the warehouse/lakehouse, transform with SQL inside). ELT dominates in cloud warehouses.
- **Layers:** staging → cleaned/integrated → marts. **Medallion:** Bronze (raw), Silver (cleaned, conformed), Gold (business-ready aggregates).
- **Incremental load:** watermark on `modified_date`/id, **CDC** (change data capture from the log), or hash comparison.
- Other terms: late-arriving facts/dimensions, data lineage, idempotent loads (re-runnable without duplicates), data quality checks, partition switching for loads.

---

## 34. Temp Tables, Table Variables and Dynamic SQL

### 34.1 Comparison

| | `#temp` (local) | `##temp` (global) | `@table` variable | CTE |
|--|----------------|-------------------|-------------------|-----|
| Scope | Session (and child scopes) | All sessions until the last referencing one ends | Batch / procedure | One statement |
| Stored | **tempdb** | tempdb | **tempdb** (not "in memory"; a common myth) | Not stored |
| Statistics | ✅ | ✅ | ❌ (until 2019 deferred compilation: row count known, no column stats) | n/a |
| Indexes | ✅ any | ✅ | Inline PK/unique/indexes only | ❌ |
| Affected by `ROLLBACK` | Yes | Yes | **No** (keeps its rows) | n/a |
| Best for | Medium/large intermediate sets, reuse, indexing | Rare | Small row counts (< ~100), TVP parameters | Readability |

```sql
CREATE TABLE #t (id INT PRIMARY KEY, val VARCHAR(50));
SELECT * INTO #t2 FROM employees WHERE dept_id = 2;
DECLARE @tv TABLE (id INT, name VARCHAR(50));
DROP TABLE IF EXISTS #t;
```
🧠 `tempdb` is also used for sorts/hash spills, row versions, cursors, and sorting during index builds: it is a shared hot spot. Multiple data files (commonly 1 per core up to 8) reduce allocation contention. Temp tables created inside stored procedures are cached and reused, which is fast.

### 34.2 When to use what
Few rows, no need for stats → table variable. Many rows, joins, or reused several times → `#temp` (+ index). Logic readability only → CTE.

### 34.3 Dynamic SQL

```sql
-- ❌ SQL INJECTION: user input becomes code
SET @sql = 'SELECT * FROM employees WHERE emp_name = ''' + @name + '''';
-- @name = ' OR 1=1; DROP TABLE employees; --      → the DROP runs

-- ✅ parameterized: input is always data
DECLARE @sql NVARCHAR(MAX) = N'SELECT * FROM dbo.employees WHERE dept_id = @d AND salary > @s';
EXEC sp_executesql @sql, N'@d INT, @s DECIMAL(10,2)', @d = 2, @s = 60000;

-- Dynamic identifiers (table/column names can't be parameters): whitelist + QUOTENAME
DECLARE @cols NVARCHAR(MAX) = (SELECT STRING_AGG(QUOTENAME(status), ',') FROM (SELECT DISTINCT status FROM orders) x);
DECLARE @q NVARCHAR(MAX) = N'SELECT * FROM (SELECT customer_id,status,amount FROM orders) s
                             PIVOT (SUM(amount) FOR status IN (' + @cols + N')) p';
EXEC sp_executesql @q;
```
Dynamic SQL runs in its **own scope**: local variables/#temp tables created inside aren't visible outside (except via OUTPUT parameters or global temp). It also breaks ownership chaining (needs direct permissions). Use it for dynamic pivots, optional-filter search screens (instead of the `@p IS NULL OR col = @p` catch-all), dynamic sorting.

---

## 35. JSON and XML in SQL

### 35.1 JSON (SQL Server 2016+; no native type until 2025 previews, stored in `NVARCHAR(MAX)`)

```sql
SELECT ISJSON(col);
SELECT JSON_VALUE(j, '$.name'),          -- scalar (returns NVARCHAR(4000))
       JSON_QUERY(j, '$.address'),       -- object or array
       JSON_MODIFY(j, '$.age', 30)
FROM t;
SELECT * FROM OPENJSON(@json) WITH (id INT '$.id', name NVARCHAR(50) '$.name');      -- JSON → rows
SELECT emp_id, emp_name FROM employees FOR JSON PATH;                                 -- rows → JSON
```
- ⚡ To index a JSON property, expose it as a **persisted computed column** and index that.
- 🔀 PostgreSQL: `json` (text copy) vs **`jsonb`** (binary, indexable): `col->'k'` (json), `col->>'k'` (text), `@>` containment, `jsonb_each`, `jsonb_array_elements`, **GIN index**. 🔀 MySQL: `JSON` type, `->`, `->>`, `JSON_EXTRACT`, `JSON_TABLE`.
- Use JSON for genuinely flexible/sparse attributes or API payload staging; keep relational data relational (you lose FKs, constraints, stats, efficient joins).

### 35.2 XML
`.value()`, `.query()`, `.nodes()`, `.exist()`, `FOR XML PATH` (still the old trick for string aggregation before `STRING_AGG`).

---

## 36. Partitioning

Splitting a big table into **partitions** by a key (usually a date) while it remains **one logical table**.

```sql
-- SQL Server
CREATE PARTITION FUNCTION pf_year (DATE) AS RANGE RIGHT FOR VALUES ('2023-01-01','2024-01-01','2025-01-01');
CREATE PARTITION SCHEME ps_year AS PARTITION pf_year ALL TO ([PRIMARY]);
CREATE TABLE sales (id INT, sale_date DATE NOT NULL, amt DECIMAL(10,2)) ON ps_year(sale_date);

-- 🔀 PostgreSQL
CREATE TABLE sales (id INT, sale_date DATE, amt NUMERIC) PARTITION BY RANGE (sale_date);
CREATE TABLE sales_2024 PARTITION OF sales FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

| Type | Use |
|------|-----|
| Range | Dates, numeric ranges |
| List | Region, status |
| Hash | Even spread, no natural range |
| Composite | Range + hash |

**Real benefits (often misunderstood):**
1. **Partition elimination:** queries filtered on the partition key read only the relevant partitions.
2. **Maintenance and data lifecycle:** **switch** a partition in/out as a *metadata-only* operation (instantly archive or load a month); drop old partitions instead of a giant `DELETE`.
3. Rebuild/maintain indexes per partition; place partitions on different storage.

⚠️ Partitioning is **not an automatic speed-up** for OLTP: queries that don't filter on the partition key touch every partition and can be *slower*. The partition column must be part of unique keys/clustered index in SQL Server (a "partition-aligned" design).
**Partitioning vs sharding:** partitioning = within one database/server; **sharding** = spreading data across multiple servers.

---

## 37. Security (DCL)

```sql
-- T-SQL: login (server) → user (database) → role → permission
CREATE LOGIN ravi WITH PASSWORD = 'Str0ng!Pass#2024';
CREATE USER  ravi FOR LOGIN ravi;
CREATE ROLE  analyst;
ALTER ROLE   analyst ADD MEMBER ravi;

GRANT SELECT, INSERT ON dbo.employees TO analyst;
GRANT EXECUTE ON dbo.usp_GetEmployees TO analyst;
REVOKE INSERT ON dbo.employees FROM analyst;          -- removes a previous GRANT or DENY
DENY SELECT ON dbo.employees(salary) TO analyst;      -- DENY always overrides GRANT
```

| Concept | Meaning |
|---------|---------|
| **Authentication** vs **Authorization** | Who are you? (Windows/Entra ID/password) vs What may you do? (permissions) |
| Login vs user | Login = server-level identity; user = its mapping inside one database |
| **Least privilege** | Grant the minimum; use roles, not individual grants; never run apps as `sa`/`dbo` |
| **Ownership chaining** | If a procedure and the tables it touches share an owner, callers need only `EXECUTE`, not table rights (broken by dynamic SQL) |
| **Row-Level Security** | Predicate function + security policy filters rows per user (multi-tenant apps) |
| **Dynamic Data Masking** | Hides columns (e.g., `xxxx-1234`) from non-privileged users; **not** encryption |
| **TDE** | Encrypts data files/backups **at rest** (transparent to apps) |
| **Always Encrypted** | Client-side column encryption; server never sees plaintext |
| **TLS** | Encrypts data **in transit** |
| **Auditing** | Records who accessed/changed what |
| **SQL injection** | Prevented by **parameterized queries**, stored procedures, input validation, least privilege (§34.3) |

Treat PII (Aadhaar, PAN, phone, email) carefully: mask, encrypt, restrict, and log access. Never store passwords in plain text; store a salted **hash** produced by the application (bcrypt/argon2), not `HASHBYTES` alone.

---

## 38. Error Handling

```sql
SET XACT_ABORT ON;                 -- any runtime error rolls the whole transaction back automatically (recommended in procs)
BEGIN TRY
    BEGIN TRANSACTION;
        INSERT INTO orders VALUES (999, 1, GETDATE(), -1, 'X');
    COMMIT;
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0 ROLLBACK;
    SELECT ERROR_NUMBER() AS num, ERROR_SEVERITY() sev, ERROR_STATE() st,
           ERROR_MESSAGE() AS msg, ERROR_LINE() AS line, ERROR_PROCEDURE() AS proc_name;
    THROW;                         -- re-raise the original error (SQL Server 2012+; preferred over RAISERROR)
END CATCH;

THROW 50001, 'Custom error', 1;            -- user error numbers must be ≥ 50000
RAISERROR('Custom %s', 16, 1, 'text');     -- older; supports printf-style formatting
```
- `TRY/CATCH` does **not** catch: compile errors in the same batch, errors with severity ≥ 20 that kill the connection, or attention events (client disconnect/timeout).
- `XACT_STATE()`: `-1` = transaction is **doomed** (only `ROLLBACK` allowed), `1` = active, `0` = none. Check it in `CATCH` before committing.
- Constraint violations, deadlock victims (1205) and timeouts are the usual suspects. Make your app **retry** on deadlocks.
- 🔀 PostgreSQL: `BEGIN ... EXCEPTION WHEN unique_violation THEN ... END;` in PL/pgSQL (an exception block automatically rolls back to an implicit savepoint). MySQL: `DECLARE ... HANDLER`.
- Always return meaningful errors; don't swallow them in an empty `CATCH`.

---

## 39. Advanced Analytical Patterns

### 39.1 Remove duplicates (keep newest)
```sql
WITH x AS (SELECT *, ROW_NUMBER() OVER (PARTITION BY email ORDER BY hire_date DESC, emp_id DESC) rn FROM employees WHERE email IS NOT NULL)
DELETE FROM x WHERE rn > 1;
```

### 39.2 Running total, % of total, MoM and YoY
```sql
WITH m AS (
  SELECT DATEFROMPARTS(YEAR(order_date),MONTH(order_date),1) AS mth, SUM(amount) AS rev
  FROM orders GROUP BY DATEFROMPARTS(YEAR(order_date),MONTH(order_date),1)
)
SELECT mth, rev,
  SUM(rev) OVER (ORDER BY mth ROWS UNBOUNDED PRECEDING) AS running_rev,
  LAG(rev) OVER (ORDER BY mth) AS prev_mth,
  (rev - LAG(rev) OVER (ORDER BY mth)) * 100.0 / NULLIF(LAG(rev) OVER (ORDER BY mth), 0) AS mom_pct,
  LAG(rev, 12) OVER (ORDER BY mth) AS same_mth_last_year        -- ⚠️ only valid if there is exactly one row per month (no gaps); use a date spine (39.11)
FROM m;
```

### 39.3 Gaps and islands (consecutive sequences)
Trick: for consecutive values, `value − ROW_NUMBER()` is **constant** within an island.
```sql
WITH x AS (
  SELECT customer_id, order_date,
         DATEADD(DAY, -ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date), order_date) AS grp
  FROM (SELECT DISTINCT customer_id, order_date FROM orders) d
)
SELECT customer_id, MIN(order_date) AS island_start, MAX(order_date) AS island_end, COUNT(*) AS days
FROM x GROUP BY customer_id, grp;
```

### 39.4 Find gaps in a sequence
```sql
SELECT id + 1 AS gap_start, nxt - 1 AS gap_end
FROM (SELECT id, LEAD(id) OVER (ORDER BY id) nxt FROM t) x
WHERE nxt - id > 1;
```

### 39.5 First / latest row per group
```sql
SELECT * FROM (SELECT o.*, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC, order_id DESC) rn FROM orders o) t WHERE rn = 1;
```

### 39.6 Median
```sql
SELECT DISTINCT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER () AS median_salary FROM employees;   -- T-SQL (analytic form)
-- 🔀 PostgreSQL / Oracle: SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) FROM employees;
```
`PERCENTILE_CONT` interpolates between the two middle values; `PERCENTILE_DISC` returns an actual value from the data.

### 39.7 "Never" and "all" questions
```sql
SELECT c.* FROM customers c WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);   -- never ordered

-- Relational division: customers who bought ALL products
SELECT customer_id FROM purchases GROUP BY customer_id
HAVING COUNT(DISTINCT product_id) = (SELECT COUNT(*) FROM products);
```

### 39.8 Swap values, conditional update
```sql
UPDATE t SET gender = CASE gender WHEN 'M' THEN 'F' WHEN 'F' THEN 'M' END;     -- all new values computed from old values in one statement
```

### 39.9 Sessionization (gap > 30 min = new session)
```sql
WITH x AS (
  SELECT user_id, ts,
    CASE WHEN LAG(ts) OVER (PARTITION BY user_id ORDER BY ts) IS NULL
           OR DATEDIFF(MINUTE, LAG(ts) OVER (PARTITION BY user_id ORDER BY ts), ts) > 30 THEN 1 ELSE 0 END AS new_sess
  FROM events
)
SELECT *, SUM(new_sess) OVER (PARTITION BY user_id ORDER BY ts ROWS UNBOUNDED PRECEDING) AS session_id FROM x;
```

### 39.10 Consecutive-streak detection (e.g., 3 days in a row)
Use gaps-and-islands (39.3) then `HAVING COUNT(*) >= 3`; or `LAG(date, 2)` and compare `DATEDIFF(DAY, LAG(date,2) OVER (...), date) = 2`.

### 39.11 Date spine (show zero for missing days)
```sql
WITH d AS (SELECT CAST('2024-01-01' AS DATE) dt UNION ALL SELECT DATEADD(DAY,1,dt) FROM d WHERE dt < '2024-03-31')
SELECT d.dt, COALESCE(SUM(o.amount), 0) AS rev
FROM d LEFT JOIN orders o ON o.order_date = d.dt
GROUP BY d.dt OPTION (MAXRECURSION 0);
```
In production use a permanent **date dimension** table instead of generating dates on the fly.

### 39.12 Pareto / ABC analysis
```sql
SELECT customer_id, total,
  SUM(total) OVER (ORDER BY total DESC ROWS UNBOUNDED PRECEDING) * 100.0 / SUM(total) OVER () AS cum_pct
FROM (SELECT customer_id, SUM(amount) total FROM orders GROUP BY customer_id) t;
-- class A: cum_pct ≤ 80, B: ≤ 95, C: rest
```

### 39.13 Overlapping intervals
Two ranges `[s1,e1]` and `[s2,e2]` overlap when **`s1 <= e2 AND s2 <= e1`**: the most useful one-liner in booking/subscription problems.

---

## 40. Subscription and Churn Analytics Patterns

> Domain patterns for **RMR (Recurring Monthly Revenue), churn, saves, retention**.
> Assumed tables: `subscriptions(sub_id, customer_id, plan, rmr, start_date, end_date, status)` and `cancel_requests(req_id, sub_id, req_date, reason, outcome)`.
> **Convention used here:** `end_date` is the date the subscription **stopped** being active (`NULL` = still active). A subscription is active on date D when `start_date <= D AND (end_date IS NULL OR end_date > D)`. State this convention in every metric definition; most "wrong numbers" are boundary disagreements.

### 40.1 Active subscriptions and RMR at month end

```sql
SELECT m.month_end, COUNT(s.sub_id) AS active_subs, SUM(s.rmr) AS total_rmr
FROM dim_month m
JOIN subscriptions s ON s.start_date <= m.month_end AND (s.end_date IS NULL OR s.end_date > m.month_end)
GROUP BY m.month_end;
```
If a month has no active subs it disappears from the `INNER JOIN`; use `LEFT JOIN` from `dim_month` if you need zeros.

### 40.2 Monthly churn rate (logo churn and revenue churn)

```sql
-- Active at the start of month M = active on the last day of the previous month
-- Churned in M = of those, ended before the first day of the next month
SELECT m.month_start,
       COUNT(*)                                                                  AS active_at_start,
       SUM(CASE WHEN s.end_date < DATEADD(MONTH, 1, m.month_start) THEN 1 ELSE 0 END)     AS churned_subs,
       SUM(CASE WHEN s.end_date < DATEADD(MONTH, 1, m.month_start) THEN 1 ELSE 0 END) * 100.0 / NULLIF(COUNT(*), 0) AS logo_churn_pct,
       SUM(CASE WHEN s.end_date < DATEADD(MONTH, 1, m.month_start) THEN s.rmr ELSE 0 END) * 100.0 / NULLIF(SUM(s.rmr), 0) AS rmr_churn_pct
FROM dim_month m
JOIN subscriptions s ON s.start_date < m.month_start AND (s.end_date IS NULL OR s.end_date >= m.month_start)
GROUP BY m.month_start;
```
- The **denominator** is the cohort at the start of the month; customers who join *and* leave inside the same month are excluded from the rate (standard definition).
- **Logo churn** = count-based; **gross revenue churn** = lost RMR ÷ starting RMR; **net revenue churn** = (churned + contraction − expansion) ÷ starting RMR (can be negative = **net negative churn**).
- This works at subscription level. For **customer-level** churn (a customer leaves only when *all* subscriptions end), aggregate to customer first: `GROUP BY customer_id` with `MAX(CASE WHEN end_date IS NULL THEN 1 ELSE 0 END)` etc.

### 40.3 Save rate

```sql
SELECT DATEFROMPARTS(YEAR(req_date), MONTH(req_date), 1) AS mth,
       COUNT(*) AS cancel_requests,
       SUM(CASE WHEN outcome = 'Saved' THEN 1 ELSE 0 END) AS saves,
       SUM(CASE WHEN outcome = 'Saved' THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS save_rate_pct
FROM cancel_requests
GROUP BY DATEFROMPARTS(YEAR(req_date), MONTH(req_date), 1);
```
Decide how to treat **multiple requests per subscription** (count distinct subscriptions or all requests) and **requests still pending** (exclude from the denominator, or report separately). Document the definition.

### 40.4 Cohort retention

```sql
WITH c AS (   -- cohort = month of the customer's first subscription start
  SELECT customer_id, DATEFROMPARTS(YEAR(MIN(start_date)), MONTH(MIN(start_date)), 1) AS cohort_month
  FROM subscriptions GROUP BY customer_id
),
a AS (        -- activity-based retention: customers with an order in month N after joining
  SELECT c.cohort_month, DATEDIFF(MONTH, c.cohort_month, o.order_date) AS month_idx, COUNT(DISTINCT o.customer_id) AS active
  FROM c JOIN orders o ON o.customer_id = c.customer_id
  GROUP BY c.cohort_month, DATEDIFF(MONTH, c.cohort_month, o.order_date)
)
SELECT a.*, a.active * 100.0 / FIRST_VALUE(a.active) OVER (PARTITION BY a.cohort_month ORDER BY a.month_idx) AS retention_pct
FROM a;
```
For **subscription-based retention**, count customers whose subscription is *active at the end of month N* instead of using orders. Output is usually pivoted into a triangle (cohort rows × month columns).

### 40.5 Tenure, ARPU, LTV

```sql
SELECT customer_id,
       DATEDIFF(MONTH, MIN(start_date), COALESCE(MAX(end_date), CAST(GETDATE() AS DATE))) AS tenure_months,
       SUM(CASE WHEN end_date IS NULL THEN rmr ELSE 0 END) AS current_rmr
FROM subscriptions GROUP BY customer_id;
-- ARPU = total RMR ÷ active customers
-- Simple LTV ≈ ARPU ÷ monthly churn rate   (e.g., ₹500 ARPU, 2% churn → ₹25,000)  — assumes constant churn; mention the assumption
```

### 40.6 RMR movement (New / Expansion / Contraction / Churn)

```sql
WITH m AS (SELECT customer_id, month_start, SUM(rmr) AS rmr FROM fact_rmr_monthly GROUP BY customer_id, month_start),
x AS (
  SELECT customer_id, month_start, rmr,
         LAG(rmr, 1, 0) OVER (PARTITION BY customer_id ORDER BY month_start) AS prev_rmr
  FROM m          -- ⚠️ assumes each customer has a row for every month (incl. 0 after churn); otherwise build a customer × month spine first
)
SELECT month_start,
  SUM(CASE WHEN prev_rmr = 0 AND rmr > 0 THEN rmr END)                                   AS new_rmr,
  SUM(CASE WHEN prev_rmr > 0 AND rmr > prev_rmr THEN rmr - prev_rmr END)                 AS expansion,
  SUM(CASE WHEN prev_rmr > 0 AND rmr < prev_rmr AND rmr > 0 THEN rmr - prev_rmr END)     AS contraction,
  SUM(CASE WHEN prev_rmr > 0 AND rmr = 0 THEN -prev_rmr END)                             AS churned_rmr
FROM x GROUP BY month_start;
```
Check: `opening RMR + new + expansion + contraction + churn = closing RMR` (a reconciliation every revenue analyst should run).

### 40.7 Reactivation / win-back
A customer with a **gap** between one subscription's `end_date` and the next subscription's `start_date`: `LAG(end_date) OVER (PARTITION BY customer_id ORDER BY start_date)` and flag rows where `start_date > prev_end_date`. Decide the gap that separates "resubscribe" from "continuous" (e.g., > 30 days).

---

## 41. Dialect Differences Cheat Sheet

| Task | SQL Server | PostgreSQL | MySQL | Oracle |
|------|-----------|------------|-------|--------|
| Limit rows | `TOP n` / `OFFSET FETCH` | `LIMIT n OFFSET m` | `LIMIT n OFFSET m` | `FETCH FIRST n ROWS ONLY` (12c+), old: `ROWNUM` |
| Auto-number | `IDENTITY(1,1)` | `GENERATED ... AS IDENTITY` / `SERIAL` | `AUTO_INCREMENT` | `IDENTITY` (12c+) / sequence |
| String concat | `+`, `CONCAT` | `\|\|`, `CONCAT` | `CONCAT` (`\|\|` is OR by default!) | `\|\|`, `CONCAT` (2 args) |
| NULL replace | `ISNULL`, `COALESCE` | `COALESCE` | `IFNULL`, `COALESCE` | `NVL`, `COALESCE` |
| Current datetime | `GETDATE()`, `SYSDATETIME()` | `NOW()` | `NOW()` | `SYSDATE`, `SYSTIMESTAMP` |
| Recursive CTE | `WITH` | `WITH RECURSIVE` | `WITH RECURSIVE` (8.0+) | `WITH` / `CONNECT BY` |
| FULL OUTER JOIN | ✅ | ✅ | ❌ (emulate with UNION) | ✅ |
| Set difference | `EXCEPT` | `EXCEPT` | `EXCEPT` (8.0.31+) | `MINUS` |
| String aggregation | `STRING_AGG` | `STRING_AGG` | `GROUP_CONCAT` | `LISTAGG` |
| Upsert | `MERGE` | `ON CONFLICT` | `ON DUPLICATE KEY UPDATE` | `MERGE` |
| Case-insensitive search | collation (default CI) | `ILIKE`, `LOWER()`, `citext` | collation (default CI) | `UPPER()`/`NLS_SORT` |
| Boolean type | `BIT` | `BOOLEAN` | `BOOLEAN` (= `TINYINT(1)`) | `NUMBER(1)` (native in 23c) |
| `INT / INT` | integer division | integer division | **decimal** (`DIV` for integer) | decimal |
| Empty string `''` vs NULL | different | different | different | **same (`''` is NULL)** |
| NULL sort position (ASC) | first | last | first | last |
| Identifier quoting | `[name]` or `"name"` | `"name"` (and folds unquoted names to lower case) | `` `name` `` | `"name"` (folds to upper case) |
| Procedural language | T-SQL | PL/pgSQL | stored routines | PL/SQL |
| Default isolation | READ COMMITTED | READ COMMITTED | REPEATABLE READ | READ COMMITTED |
| Transactional DDL | partly (many DDLs roll back) | **yes** | **no** (implicit commit) | **no** (implicit commit) |
| Alias in `GROUP BY` | no | yes | yes | no |
| `SELECT` without `FROM` | ✅ | ✅ | ✅ | needs `FROM DUAL` |
| Update with join | `UPDATE ... FROM ... JOIN` | `UPDATE ... FROM` | `UPDATE a JOIN b ... SET` | `MERGE` or subquery |

---

## 42. Best Practices and Anti-Patterns

### 42.1 Best practices

- **Design:** model the grain and keys first; normalize to 3NF for OLTP; enforce integrity with PK/FK/CHECK/NOT NULL; use the **smallest correct data type**; `DECIMAL` for money; `DATE` for dates; index FK columns.
- **Naming:** consistent `snake_case`, clear constraint/index prefixes, schema-qualified object names.
- **Writing queries:** list columns (no `SELECT *`), explicit `JOIN ... ON`, table aliases with qualified columns, `EXISTS` over `NOT IN`, half-open date ranges, tie-breakers in `ORDER BY`, comments on the *why*.
- **Safety:** wrap multi-step DML in transactions; `SELECT` before `UPDATE/DELETE`; check `@@ROWCOUNT`; never run ad-hoc DML in production without a `WHERE` you have verified; take a backup or use a snapshot before risky changes.
- **Performance:** SARGable predicates, right indexes, current statistics, set-based logic, batch big changes, read the plan before guessing.
- **Security:** parameterize everything, least privilege, no `sa` for apps, protect PII.
- **Operations:** version-control DDL (migrations), test restores, monitor waits/blocking/log growth, review unused indexes.
- **Readability:** CTEs for steps, consistent formatting, one clause per line.

### 42.2 Anti-patterns

| Anti-pattern | Why it's bad |
|-------------|--------------|
| `SELECT *` | Extra I/O, breaks on schema change, blocks covering indexes |
| Functions on columns in `WHERE`/`JOIN` | Non-SARGable → scans |
| `NOT IN` with a nullable subquery | Returns zero rows when a NULL exists |
| Implicit conversions | Index not used, CPU wasted, sometimes errors |
| Cursors/loops for set-based tasks | Row-by-row = slow |
| `NOLOCK` everywhere | Dirty, missing and duplicate reads |
| Scalar UDFs in big queries | Executed per row, blocks parallelism |
| Comma-separated values in a column | Violates 1NF, unindexable |
| EAV (entity-attribute-value) tables for everything | No types, no constraints, painful queries |
| `ORDER BY` in views/subqueries | Not honoured / not allowed |
| Over-indexing | Slow writes, big storage |
| `FLOAT` for money | Rounding drift |
| Dates stored as strings | No validation, no range queries, no index use |
| String-concatenated dynamic SQL | SQL injection |
| Tables without a primary key | Duplicates, poor plans |
| `DISTINCT` to hide a bad join | Masks fan-out bugs |
| `VARCHAR(MAX)`/`NVARCHAR(4000)` for everything | Wide memory grants, can't be indexed, hides intent |
| Random GUID as clustered key | Page splits, fragmentation |
| Magic values for "unknown" (`-1`, `1900-01-01`) | Corrupt aggregates, break FKs |
| Using `BETWEEN` on timestamps | End-of-day rounding bugs |
| `SELECT COUNT(*)` to check existence | Use `EXISTS` (stops at first row) |
| Storing derived data without a reason | Goes stale; compute it or use a computed column |
| Mixing business logic in triggers | Hidden side effects |

---

## 43. Interview Q&A

> Each answer is deliberately short; add the "why" from the relevant section to stand out.

**1. WHERE vs HAVING?** WHERE filters rows before grouping (no aggregates); HAVING filters groups after aggregation. Put row filters in WHERE so less data is aggregated.

**2. DELETE vs TRUNCATE vs DROP?** (§2.1) DELETE: DML, row-by-row, WHERE, triggers. TRUNCATE: DDL, deallocates pages, resets identity, no WHERE, blocked by FKs. DROP: removes the object.

**3. RANK vs DENSE_RANK vs ROW_NUMBER?** Ties: RANK leaves gaps (1,1,3), DENSE_RANK doesn't (1,1,2), ROW_NUMBER is always unique (arbitrary among ties unless tie-broken).

**4. INNER vs OUTER join?** Inner returns only matches; outer also keeps unmatched rows of one or both sides with NULLs. Filter placement (ON vs WHERE) matters for outer joins.

**5. UNION vs UNION ALL?** UNION removes duplicates (sort/hash cost); UNION ALL keeps all and is faster.

**6. Clustered vs non-clustered index?** Clustered = the table itself sorted by key (one per table, leaf = data pages); non-clustered = separate structure pointing to rows (many allowed). Non-clustered indexes carry the clustering key, so keep it narrow.

**7. Primary key vs unique key?** PK: one per table, no NULLs, clustered by default; unique: many, NULL rules vary by engine.

**8. CTE vs temp table vs view vs subquery?** CTE = named, single-statement query (not stored); temp table = stored in tempdb with stats/indexes; view = saved query in the database; subquery = unnamed inline query.

**9. Stored procedure vs function?** (§29.3) Procedures can change data and use transactions; functions return values and are used inside queries.

**10. Explain normalization.** (§32) Remove redundancy and anomalies: 1NF atomic, 2NF whole key, 3NF nothing but the key; denormalize deliberately for reads.

**11. ACID?** Atomicity (all or nothing), Consistency (valid state to valid state), Isolation (concurrent txns don't interfere), Durability (commit survives crashes via write-ahead log).

**12. Isolation levels and the anomalies they prevent?** (§30.4) Read Uncommitted → Read Committed (no dirty reads) → Repeatable Read (no non-repeatable reads) → Serializable (no phantoms). RCSI/Snapshot use row versions.

**13. What is a deadlock and how do you handle it?** Cyclic lock wait; the engine kills a victim (error 1205). Prevent with consistent access order, short transactions, good indexes, RCSI; retry in the application.

**14. Table scan vs index seek?** Scan reads all pages; seek navigates the B-tree to the matching range (O(log n)).

**15. What is SARGable?** A predicate written so the engine can use an index seek: leave the indexed column bare and avoid functions/implicit conversions on it.

**16. Correlated vs non-correlated subquery?** Correlated references the outer query (logically evaluated per outer row, often unnested by the optimizer); non-correlated is independent and runs once.

**17. COUNT(\*) vs COUNT(col) vs COUNT(1)?** `COUNT(*)` = `COUNT(1)` = all rows; `COUNT(col)` skips NULLs; `COUNT(DISTINCT col)` counts distinct non-NULLs.

**18. How do you find duplicates?** `GROUP BY cols HAVING COUNT(*) > 1`, or `ROW_NUMBER() OVER (PARTITION BY cols ORDER BY ...)` and look at `rn > 1`.

**19. Second highest salary?** `DENSE_RANK() = 2`, or `MAX(salary) WHERE salary < (SELECT MAX(salary) ...)`; discuss ties and empty results.

**20. Delete duplicate rows?** CTE with `ROW_NUMBER()` and `DELETE ... WHERE rn > 1`; mind NULL keys.

**21. Surrogate key?** An artificial, meaningless key (identity/GUID) used for joins and stability; keep a unique constraint on the natural key.

**22. Star vs snowflake?** Star = denormalized dimensions, fewer joins, faster; snowflake = normalized dimensions, less redundancy, more joins.

**23. SCD types?** 1 overwrite, 2 history rows with validity dates and new surrogate keys, 3 extra column for previous value.

**24. What is a trigger?** Code that runs automatically on DML/DDL events; AFTER/INSTEAD OF; set-based `inserted/deleted`; use sparingly.

**25. View vs materialized view?** Virtual (always current, no storage) vs stored result (fast reads, refresh cost).

**26. How do you optimize a slow query?** Get the actual plan and `STATISTICS IO` → find scans, lookups, spills, estimate/actual mismatches → fix SARGability, indexes, statistics, data types, query shape → re-measure. Don't guess.

**27. Parameter sniffing?** The plan is built for the first parameter values and reused for others; skewed data makes it bad for some. Fix: RECOMPILE, OPTIMIZE FOR, split procs, Query Store, PSP optimization.

**28. Can TRUNCATE be rolled back?** SQL Server and PostgreSQL: yes inside a transaction. MySQL and Oracle: no (implicit commit).

**29. Self join vs cross join?** Self join: a table joined to itself via aliases (hierarchies, comparisons); cross join: Cartesian product m × n.

**30. `= NULL` vs `IS NULL`?** `= NULL` is always UNKNOWN (no rows); use `IS NULL`.

**31. ETL vs ELT?** ETL transforms before loading; ELT loads raw first and transforms inside the warehouse.

**32. Fact vs dimension?** Facts: measurements of events with FKs; dimensions: descriptive context.

**33. Covering index?** An index that contains all columns the query needs (key + INCLUDE) so no lookup is required.

**34. DATEDIFF semantics?** It counts datepart boundaries crossed, not elapsed whole units.

**35. MVCC?** Multi-version concurrency control: readers see snapshots, writers create new versions, so readers and writers don't block each other.

**36. Why not use `NOLOCK` everywhere?** Dirty reads, missing/duplicate rows, errors; use RCSI instead.

**37. Why does data type size matter?** (§4–§5) Rows per page, I/O, memory, index size (the clustering key is copied into every non-clustered index), memory grants, comparison speed.

**38. INT vs BIGINT?** 4 vs 8 bytes; INT tops out at ~2.1 billion; choose BIGINT when growth can exceed that, otherwise INT keeps the table and every index narrower.

**39. CHAR vs VARCHAR vs NVARCHAR?** Fixed width vs variable (+2 bytes) vs Unicode (2 bytes/char).

**40. DECIMAL vs FLOAT?** DECIMAL is exact (5–17 bytes in SQL Server) for money; FLOAT is binary approximate (4/8 bytes) and drifts.

**41. DATETIME vs DATETIME2?** DATETIME2: wider range, up to 100 ns, 6–8 bytes, ANSI-friendly; DATETIME: 8 bytes, 1753+, ~3.33 ms rounding.

**42. What is a transaction log used for?** Atomicity/durability/recovery; point-in-time restore and replication; grows if log backups are missing in FULL recovery or a transaction stays open.

**43. Window function vs GROUP BY?** GROUP BY collapses rows; window functions keep every row and add computed columns over a window.

**44. What causes row duplication after a join?** Joining on a non-unique key (fan-out), many-to-many; fix by aggregating to the join grain first.

**45. How do you design a table for 1 billion rows?** Narrow types, BIGINT identity clustered key, partition by date, columnstore for analytics, careful indexes, batch loads, archive via partition switching, monitor stats and log.

---

## 44. One-Page Cheat Sheet

```text
WRITE ORDER:     SELECT → FROM → JOIN → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT/TOP
EXECUTION ORDER: FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT(+windows) → DISTINCT → ORDER BY → TOP/LIMIT

JOINS:      INNER (A∩B) | LEFT (A + A∩B) | RIGHT | FULL | CROSS (m×n) | SELF | SEMI (EXISTS) | ANTI (NOT EXISTS)
SETS:       UNION (distinct) | UNION ALL | INTERSECT | EXCEPT/MINUS
AGGREGATES: COUNT SUM AVG MIN MAX STRING_AGG   (ignore NULLs; COUNT(*) counts rows)
WINDOW:     fn() OVER (PARTITION BY .. ORDER BY .. ROWS BETWEEN ..)   ROWS = physical, RANGE = peers
RANKING:    ROW_NUMBER (unique) | RANK (gaps) | DENSE_RANK (no gaps) | NTILE
OFFSET:     LAG / LEAD / FIRST_VALUE / LAST_VALUE (needs full frame)
NULLS:      IS NULL | COALESCE | NULLIF | NOT IN + NULL = trap | NULL never joins | CHECK passes on UNKNOWN
KEYS:       PK | FK (index it!) | UNIQUE | CHECK | DEFAULT      surrogate + natural unique
SIZES:      BIT 1b | TINYINT 1 | SMALLINT 2 | INT 4 | BIGINT 8 | DECIMAL 5/9/13/17 | FLOAT 4/8 | DATE 3 | DATETIME2 6-8
            DATETIME 8 | GUID 16 | CHAR(n) n | VARCHAR actual+2 | NVARCHAR 2×actual+2 | page 8 KB | extent 64 KB
INDEX:      Clustered (1/table, the data) | Non-clustered | Covering (INCLUDE) | Filtered | Columnstore
            equality cols first, range last; leftmost prefix; keep clustered key narrow/unique/static/increasing
SARGABLE:   bare column, no functions, match data types, half-open date ranges
NF:         1NF atomic | 2NF whole key | 3NF nothing but the key | BCNF every determinant is a key
ACID:       Atomicity Consistency Isolation Durability (WAL)
ISOLATION:  RU < RC < RR < SERIALIZABLE; SNAPSHOT/RCSI = row versioning (readers don't block writers)
SCD:        1 overwrite | 2 history rows + new surrogate | 3 previous column
STAR:       Fact (grain, measures, FKs) ↔ Dimensions (descriptive); define grain first
DELETE (DML, WHERE, logged) | TRUNCATE (DDL, fast, resets identity) | DROP (removes object)
GOTCHAS:    integer division | AVG(int) is int | SUM(int) overflows | DATEDIFF counts boundaries
            BETWEEN on datetimes | ON vs WHERE in outer joins | NOT IN + NULL | ROWS vs RANGE default frame
```

---

## 45. Where to Learn More, and a Study Roadmap

### 45.1 Official documentation (the final authority for sizes, limits and syntax)

- **SQL Server / T-SQL:** Microsoft Learn → "Transact-SQL reference", "Data types (Transact-SQL)", "SQL Server index architecture and design guide", "Pages and extents architecture guide", "Transaction locking and row versioning guide", "Query processing architecture guide". These guides are the best free deep dives available.
- **PostgreSQL:** postgresql.org/docs → "Data Types", "Performance Tips", "Concurrency Control", "Indexes", "Database Physical Storage".
- **MySQL:** dev.mysql.com/doc → "Data Types", "InnoDB Storage Engine", "Optimization".
- **Oracle:** docs.oracle.com → "SQL Language Reference", "Concepts".

### 45.2 Books worth owning

| Book | Why |
|------|-----|
| *T-SQL Fundamentals* (Itzik Ben-Gan) | The clearest explanation of logical query processing, joins, windows, and set-based thinking |
| *T-SQL Window Functions* (Itzik Ben-Gan) | The definitive window function book |
| *SQL Performance Explained* (Markus Winand), also free at **use-the-index-luke.com** | Indexing and SARGability across all databases |
| *SQL Server Execution Plans* (Grant Fritchey) | Reading plans properly |
| *The Data Warehouse Toolkit* (Ralph Kimball & Margy Ross) | Dimensional modelling bible: facts, dimensions, SCDs |
| *Designing Data-Intensive Applications* (Martin Kleppmann) | Transactions, isolation, replication, storage engines in depth |
| *SQL Antipatterns* (Bill Karwin) | Common design mistakes and fixes |
| *Database Internals* (Alex Petrov) | If you want B-trees, LSM, WAL and consensus explained properly |

### 45.3 Blogs, courses and tools

- **Brent Ozar** (brentozar.com): practical SQL Server performance, free `sp_Blitz` tools.
- **SQLskills** (Paul Randal, Kimberly Tripp): internals, waits, indexing strategy.
- **Erik Darling**: modern query tuning, parameter sniffing.
- **Pinal Dave (SQLAuthority)** for quick how-tos.
- **Use The Index, Luke!**: indexing for developers.
- **Mode SQL Tutorial**, **SQLBolt**, **PG Exercises** (pgexercises.com): interactive basics.
- **SQL Server Central**, **dba.stackexchange.com**, **Stack Overflow** for real-world problems.
- **Practice:** LeetCode (SQL 50 and Database section), HackerRank SQL, StrataScratch, DataLemur, SQLZoo.
- **Free practice databases:** AdventureWorks and WideWorldImporters (SQL Server), Northwind, Pagila / DVD Rental (PostgreSQL), Sakila (MySQL), Stack Overflow data dump (huge, great for tuning practice).
- **Tools:** SSMS / Azure Data Studio, DBeaver, pgAdmin, DataGrip; Docker images to run SQL Server/PostgreSQL locally in minutes.

### 45.4 Study roadmap

| Phase | Topics (sections) | Goal |
|-------|-------------------|------|
| **1. Foundations** | §1–3, §13–17 | Write correct SELECTs with filters, aggregates, NULL awareness |
| **2. Core querying** | §18–21 | Joins, set operators, subqueries, CTEs without fan-out bugs |
| **3. Analytics** | §22–25, §39 | Window functions, CASE, pivots, classic patterns (**highest interview value**) |
| **4. Types and storage** | §4–8 | Explain data type choices and their performance impact with numbers |
| **5. Design** | §9–12, §32–33 | Constraints, keys, normalization, dimensional modelling |
| **6. Performance** | §26–28, §34 | Indexes, statistics, plans, SARGability, tuning |
| **7. Concurrency and operations** | §29–31, §36–38 | Transactions, locking, recovery, partitioning, security |
| **8. Domain mastery** | §40 | Churn, RMR, retention, cohort analytics |

Daily habit: solve 2 to 3 problems, then **explain each solution aloud** (what the engine does, where NULLs/ties/duplicates could break it). Once a week, take one slow query, read its plan and make it faster.

---

⭐ *Keep adding what you learn: every real bug you hit is a gotcha worth a line in this file.*
