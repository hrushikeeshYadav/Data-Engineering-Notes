# PySpark Complete End-to-End Guide
## From Python Fundamentals to Production Data Engineering

**Purpose:** Learn PySpark, understand its libraries, build end-to-end data pipelines, optimize Spark jobs, and prepare for Data Engineer interviews.

**Technology stack:** Python, Apache Spark, PySpark, Spark SQL, Delta Lake, Azure Databricks, ADLS Gen2, Azure Data Factory, and Power BI.

---

# 1. What Is PySpark?

PySpark is the Python API for Apache Spark. It allows us to process and analyze large datasets using Python while Apache Spark distributes computation across multiple CPU cores or machines.

For example, suppose an organization stores 500 GB of customer transactions. Processing the entire dataset on one computer may be slow or impossible because of memory limitations.

PySpark can divide the work into partitions, process those partitions in parallel, and combine the results.

### Apache Spark vs. PySpark

- **Apache Spark:** The distributed data processing engine.
- **PySpark:** The Python interface used to interact with Spark.
- **Spark SQL:** Spark's structured data processing and SQL engine.
- **Spark DataFrame:** A distributed collection of data organized into named columns.
- **SparkSession:** The primary entry point for working with Spark DataFrames and SQL.

PySpark is not a replacement for Python. It is a framework that lets Python code use Spark's distributed processing capabilities.

### Why use PySpark?

1. Process large datasets across multiple machines.
2. Perform ETL and ELT operations.
3. Clean and transform raw data.
4. Join multiple large datasets.
5. Read and write CSV, JSON, Parquet, and Delta tables.
6. Build batch and streaming pipelines.
7. Implement Bronze, Silver, and Gold data layers.
8. Integrate with cloud storage and data warehouses.
9. Optimize data processing using partitioning, caching, and query optimization.
10. Support analytics, reporting, and machine learning workflows.

---

# 2. What Is Apache Spark Architecture?

Apache Spark uses a driver and executors to coordinate distributed processing.

## Main components

### 2.1 Driver

The driver coordinates the Spark application.

Responsibilities:
- Creates the SparkSession.
- Builds the execution plan.
- Schedules jobs, stages, and tasks.
- Coordinates executors.
- Collects task results and manages application execution.

### 2.2 Executor

Executors are processes that run on worker nodes.

Responsibilities:
- Execute tasks assigned by the driver.
- Process data partitions.
- Cache data in memory or on disk.
- Perform shuffle operations.
- Report task status to the driver.

### 2.3 Cluster Manager

The cluster manager allocates computing resources.

Examples:
- Spark Standalone.
- Hadoop YARN.
- Kubernetes.
- Cloud-managed compute services such as Azure Databricks.

Azure Databricks manages the underlying Spark infrastructure through its platform.

### 2.4 Worker Node

A worker node provides CPU, memory, and other resources used by executors.

### 2.5 Spark Application

A Spark application consists of a driver and its executors.

A single application can execute multiple jobs, and each job can contain multiple stages and tasks.

## How Spark processes data

```text
Python / PySpark Code
          |
          v
     SparkSession
          |
          v
     Spark Driver
          |
          v
    Logical Plan
          |
          v
    Optimized Plan
          |
          v
    Physical Plan
          |
          v
   Spark Scheduler
          |
          v
     Spark Stages
          |
          v
    Tasks / Partitions
          |
          v
      Executors
          |
          v
   Distributed Output
```

The Catalyst optimizer optimizes supported structured queries. Spark's execution engine then executes the selected physical plan.

---

# 3. Important Spark Terminology

| Term | Meaning |
|---|---|
| Application | A complete Spark application |
| Driver | Coordinates application execution |
| Executor | Runs tasks and processes partitions |
| Cluster | Collection of computing resources |
| Partition | A logical division of distributed data |
| Job | Computation triggered by an action |
| Stage | Group of tasks separated by shuffle boundaries |
| Task | Smallest scheduled unit of Spark computation |
| DAG | Directed Acyclic Graph of computation dependencies |
| Lazy evaluation | Transformations are planned before execution |
| Shuffle | Redistribution of data between partitions |
| Narrow transformation | Processing without a full redistribution of parent data |
| Wide transformation | Transformation that generally requires data redistribution |
| Catalyst | Optimizer for Spark SQL and DataFrame operations |
| Tungsten | Spark execution and memory-management improvements |
| AQE | Adaptive Query Execution; optimizes execution using runtime statistics |
| Lineage | Information describing how a dataset was derived |

### Narrow transformation examples

- `select()`
- `filter()`
- `withColumn()`
- `map()` on an RDD
- `coalesce()` in many common uses

### Wide transformation examples

- `groupBy().agg()`
- `distinct()`
- `repartition()`
- Many joins
- `orderBy()`

The exact execution behavior depends on the operation and the physical plan.

---

# 4. PySpark Installation and Environment Setup

For local learning, you can use Python and PySpark. For production-style cloud projects, Azure Databricks provides a managed Spark environment.

## Install PySpark locally

```bash
pip install pyspark
```

For common local workflows that also use pandas and Parquet:

```bash
pip install pandas pyarrow
```

Use a compatible Python and Java runtime for the selected Spark release. Check the official Spark documentation for version-specific requirements.

Verify the installation:

```python
import pyspark

print(pyspark.__version__)
```

## Create a SparkSession

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("PySparkLearning")
    .master("local[*]")
    .getOrCreate()
)

print(spark.version)
```

`local[*]` uses the available logical CPU cores on the local machine. It is intended for local development, not a production cluster configuration.

In Azure Databricks notebooks, a SparkSession is normally available as `spark`.

---

# 5. PySpark Libraries and Modules

Understanding the libraries is essential because PySpark is a framework containing multiple APIs and supporting integrations.

## 5.1 Core PySpark libraries

### `pyspark.sql`

Used for SparkSession, DataFrames, SQL, schemas, and structured data processing.

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql import types as T
from pyspark.sql.window import Window
```

### `pyspark.sql.functions`

Contains built-in functions for filtering, string manipulation, dates, aggregation, conditional logic, null handling, and more.

Examples:

- `F.col()`
- `F.lit()`
- `F.when()`
- `F.coalesce()`
- `F.sum()`
- `F.count()`
- `F.row_number()`
- `F.to_date()`
- `F.current_timestamp()`
- `F.regexp_replace()`
- `F.broadcast()`

### `pyspark.sql.types`

Used to define schemas and data types.

Common types:

- `StringType`
- `IntegerType`
- `LongType`
- `DoubleType`
- `DecimalType`
- `BooleanType`
- `DateType`
- `TimestampType`
- `ArrayType`
- `MapType`
- `StructType`
- `StructField`

### `pyspark.sql.window`

Used for window functions such as ranking, running totals, and previous-row comparisons.

### `pyspark.sql.streaming`

Provides Structured Streaming APIs for incremental data processing.

### `pyspark.pandas`

Provides a pandas-like API backed by Spark. It is useful when adapting pandas-style workflows, but its behavior and performance characteristics differ from native pandas.

### `pyspark.ml`

Provides DataFrame-based machine learning pipelines and algorithms.

### `pyspark.mllib`

The older RDD-based machine learning API. Prefer `pyspark.ml` for most new Spark ML development.

### `pyspark.RDD`

Provides a lower-level distributed collection API. DataFrames are generally preferred for structured ETL because Spark can optimize their execution.

## 5.2 Supporting Python libraries

| Library | Purpose |
|---|---|
| `pandas` | Local in-memory data analysis |
| `numpy` | Numerical computing |
| `pyarrow` | Columnar data interchange and Arrow integration |
| `requests` | Calling HTTP APIs from Python |
| `json` | JSON encoding and decoding |
| `datetime` | Python date and time operations |
| `logging` | Application logging |
| `os` and `pathlib` | Local filesystem and path handling |
| `unittest` or `pytest` | Testing |
| `delta-spark` | Delta Lake integration in compatible self-managed Spark environments |
| `azure-storage-blob` | Azure Blob Storage SDK operations |
| `azure-identity` | Azure identity and authentication |
| `scikit-learn` | Machine learning outside or alongside Spark |

**Important:** These libraries are not all part of PySpark itself. Some are Python standard-library modules, some are external packages, and some are available only with particular Spark distributions or cloud platforms.

In Azure Databricks, Delta Lake capabilities are typically integrated into the platform. Do not install packages blindly; confirm the runtime and dependency requirements first.

---

# 6. Create DataFrames

A DataFrame represents structured distributed data.

## Create a DataFrame from Python data

```python
data = [
    (1, "Ravi", 50000),
    (2, "Priya", 65000),
    (3, "Kiran", 45000)
]

columns = ["employee_id", "name", "salary"]

df = spark.createDataFrame(data, columns)

df.show()
df.printSchema()
df.columns
df.count()
```

## Define an explicit schema

Explicit schemas help prevent inconsistent type inference.

```python
from pyspark.sql import types as T

schema = T.StructType([
    T.StructField("employee_id", T.IntegerType(), False),
    T.StructField("name", T.StringType(), True),
    T.StructField("salary", T.DoubleType(), True)
])

df = spark.createDataFrame(data, schema)
```

The nullability setting describes the schema; it does not guarantee that every incoming source record satisfies the intended business rules.

## Common DataFrame inspection methods

```python
df.show(10, truncate=False)
df.printSchema()
df.columns
df.dtypes
df.count()
df.describe().show()
df.limit(5).show()
df.explain("formatted")
```

`count()` is an action and may require Spark to process the underlying data.

---

# 7. Read Data from Different Sources

## 7.1 Read CSV

```python
df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv("/path/to/customers.csv")
)
```

For production, prefer an explicit schema when practical.

```python
df = (
    spark.read
    .option("header", True)
    .schema(schema)
    .csv("/path/to/customers.csv")
)
```

## 7.2 Read JSON

```python
df = spark.read.json("/path/to/customers.json")
```

JSON can contain nested structures. Define an explicit schema when predictable types and validation are important.

## 7.3 Read Parquet

```python
df = spark.read.parquet("/path/to/customers/")
```

Parquet is a columnar format well suited to analytical workloads.

## 7.4 Read Delta tables

```python
df = spark.read.format("delta").load("/path/to/delta-table")
```

In Databricks, you can also read a registered table:

```python
df = spark.table("main.silver.customers")
```

## 7.5 Read a SQL table

The generic Spark JDBC reader can access supported relational databases.

```python
jdbc_df = (
    spark.read.format("jdbc")
    .option("url", "jdbc:sqlserver://server:1433;databaseName=SalesDB")
    .option("dbtable", "dbo.Customers")
    .option("user", "YOUR_USER")
    .option("password", "YOUR_SECRET")
    .option("driver", "com.microsoft.sqlserver.jdbc.SQLServerDriver")
    .load()
)
```

This is a template, not a ready-to-run connection. Configure the JDBC driver, networking, permissions, and secret management appropriately. Do not hardcode production credentials.

## 7.6 Read Excel

Spark does not provide a general native Excel reader in the same way it supports CSV and Parquet.

Common options include:
- An approved Spark Excel connector.
- Converting Excel files to CSV or Parquet before processing.
- Reading a small workbook with Python libraries and creating a DataFrame.

For large-scale production pipelines, converting the source into a scalable format is often preferable.

---

# 8. DataFrame Transformations

Transformations create new DataFrames or define computation. They are generally evaluated lazily.

## 8.1 Select columns

```python
df.select("employee_id", "name").show()
```

Using expressions:

```python
df.select(
    F.col("name"),
    (F.col("salary") * 12).alias("annual_salary")
).show()
```

## 8.2 Filter records

```python
high_salary = df.filter(F.col("salary") > 50000)

high_salary.show()
```

Equivalent SQL-like filtering:

```python
df.where("salary > 50000").show()
```

## 8.3 Add a column

```python
df = df.withColumn(
    "annual_salary",
    F.col("salary") * 12
)
```

## 8.4 Rename a column

```python
df = df.withColumnRenamed("name", "employee_name")
```

## 8.5 Drop a column

```python
df = df.drop("annual_salary")
```

## 8.6 Conditional logic

```python
df = df.withColumn(
    "salary_band",
    F.when(F.col("salary") >= 70000, "High")
     .when(F.col("salary") >= 50000, "Medium")
     .otherwise("Low")
)
```

## 8.7 Remove duplicates

```python
df = df.dropDuplicates()
```

Using a business key:

```python
df = df.dropDuplicates(["employee_id"])
```

`dropDuplicates()` does not define which record is the business-approved winner when duplicates have different values. For deterministic deduplication, use a window and an explicit ordering rule.

## 8.8 Sort data

```python
df.orderBy(F.col("salary").desc()).show()
```

## 8.9 Limit records

```python
df.limit(10).show()
```

## 8.10 Distinct values

```python
df.select("name").distinct().show()
```

## 8.11 Union DataFrames

```python
combined = df1.unionByName(df2)
```

When schemas differ, handle missing columns explicitly.

```python
combined = df1.unionByName(
    df2,
    allowMissingColumns=True
)
```

A union combines records; it does not automatically remove duplicates.

---

# 9. Data Types and Type Casting

Incorrect types can cause calculation errors and poor data quality.

```python
df = (
    df.withColumn("salary", F.col("salary").cast("double"))
      .withColumn("employee_id", F.col("employee_id").cast("int"))
)
```

Useful types include:

- String
- Integer
- Long
- Double
- Decimal
- Boolean
- Date
- Timestamp
- Array
- Map
- Struct

For financial amounts, consider an appropriate `DecimalType` rather than floating-point types when exact decimal representation is required.

```python
df = df.withColumn(
    "amount",
    F.col("amount").cast(T.DecimalType(18, 2))
)
```

Invalid conversions may produce nulls or errors depending on the data, Spark version, and ANSI configuration. Validate the results instead of assuming the conversion succeeded.

---

# 10. Handling Null Values

Null handling is one of the most important parts of data engineering.

## Identify null values

```python
df.filter(F.col("name").isNull()).show()
```

Count nulls in a column:

```python
df.select(
    F.sum(
        F.when(F.col("name").isNull(), 1).otherwise(0)
    ).alias("null_name_count")
).show()
```

## Remove rows with nulls

```python
df.dropna()
```

For selected columns:

```python
df.dropna(subset=["employee_id"])
```

## Replace null values

```python
df.fillna({"name": "Unknown"})
```

## Conditional replacement

```python
df = df.withColumn(
    "name",
    F.coalesce(F.col("name"), F.lit("Unknown"))
)
```

## Important distinction

- `isNull()` checks whether a value is null.
- `isNotNull()` checks whether a value is not null.
- `coalesce()` returns the first non-null expression.
- `fillna()` replaces null values.
- An empty string is not the same as null.
- A string containing `"NULL"` is not necessarily a database null.

Choose replacements according to business rules. Do not replace missing financial or customer values with zero unless that interpretation is valid.

---

# 11. String Functions

Import functions:

```python
from pyspark.sql import functions as F
```

Examples:

```python
df = df.withColumn("name", F.trim(F.col("name")))

df = df.withColumn("name", F.upper(F.col("name")))

df = df.withColumn("name", F.lower(F.col("name")))

df = df.withColumn(
    "name_length",
    F.length(F.col("name"))
)

df = df.withColumn(
    "clean_phone",
    F.regexp_replace(F.col("phone"), "[^0-9]", "")
)
```

Other useful functions:

- `substring()`
- `split()`
- `concat()`
- `concat_ws()`
- `regexp_extract()`
- `regexp_replace()`
- `instr()`
- `contains()`
- `startswith()`
- `endswith()`

Example: split a full name.

```python
df = df.withColumn(
    "name_parts",
    F.split(F.col("full_name"), " ")
)
```

Always test regular expressions against realistic source data.

---

# 12. Date and Timestamp Functions

Date operations are common in incremental loads, reporting, customer retention, and transaction analytics.

## Convert a string into a date

```python
df = df.withColumn(
    "order_date",
    F.to_date(F.col("order_date"), "yyyy-MM-dd")
)
```

## Extract year and month

```python
df = (
    df.withColumn("order_year", F.year("order_date"))
      .withColumn("order_month", F.month("order_date"))
)
```

## Date difference

```python
df = df.withColumn(
    "days_since_order",
    F.datediff(F.current_date(), F.col("order_date"))
)
```

## Add days

```python
df = df.withColumn(
    "next_date",
    F.date_add(F.col("order_date"), 7)
)
```

## First and last day of a month

```python
df = df.withColumn(
    "month_start",
    F.trunc("order_date", "month")
)

df = df.withColumn(
    "month_end",
    F.last_day("order_date")
)
```

## Other useful functions

- `current_date()`
- `current_timestamp()`
- `to_timestamp()`
- `date_format()`
- `months_between()`
- `add_months()`
- `datediff()`
- `unix_timestamp()`
- `from_unixtime()`

**Time-zone note:** Date and timestamp behavior depends on the source type, session time zone, and parsing configuration. Establish consistent time-zone rules in production.

---

# 13. Aggregations and Grouping

Aggregation summarizes data across records.

## Count customers

```python
df.groupBy("city").agg(
    F.count("*").alias("customer_count")
).show()
```

## Total sales by category

```python
sales_summary = df.groupBy("category").agg(
    F.sum("sales").alias("total_sales"),
    F.avg("sales").alias("average_sales"),
    F.count("*").alias("record_count"),
    F.max("sales").alias("maximum_sale"),
    F.min("sales").alias("minimum_sale")
)
```

## Filter grouped results

```python
sales_summary.filter(
    F.col("total_sales") > 100000
).show()
```

For SQL-style aggregate filtering, use `HAVING` when writing SQL.

```python
df.createOrReplaceTempView("sales")

spark.sql("""
    SELECT category, SUM(sales) AS total_sales
    FROM sales
    GROUP BY category
    HAVING SUM(sales) > 100000
""").show()
```

Common aggregate functions:

- `count()`
- `countDistinct()`
- `sum()`
- `avg()`
- `min()`
- `max()`
- `first()`
- `last()`
- `collect_list()`
- `collect_set()`

Use `collect_list()` and `collect_set()` carefully. Collecting large arrays into one group can consume significant memory.

---

# 14. Joins in PySpark

Joins combine records from two DataFrames using a matching condition.

## Inner join

Returns matching records from both sides.

```python
result = customers.join(
    orders,
    on="customer_id",
    how="inner"
)
```

## Left join

Keeps every left-side record, even when no right-side match exists.

```python
result = customers.join(
    orders,
    on="customer_id",
    how="left"
)
```

## Right join

```python
result = customers.join(
    orders,
    on="customer_id",
    how="right"
)
```

## Full outer join

```python
result = customers.join(
    orders,
    on="customer_id",
    how="full_outer"
)
```

## Left anti join

Returns left-side records that have no match on the right.

```python
missing = customers.join(
    orders,
    on="customer_id",
    how="left_anti"
)
```

Useful for identifying customers without orders.

## Left semi join

Returns left-side records that have a matching record on the right, without adding right-side columns.

```python
matched = customers.join(
    orders,
    on="customer_id",
    how="left_semi"
)
```

## Join using different column names

```python
result = customers.join(
    orders,
    customers["customer_id"] == orders["cust_id"],
    "left"
)
```

When both DataFrames contain columns with the same names, qualify or rename columns to avoid ambiguous references.

### Broadcast join

Broadcasting sends a sufficiently small DataFrame to executors to avoid shuffling the large side for that join.

```python
result = large_df.join(
    F.broadcast(small_df),
    on="product_id",
    how="left"
)
```

Only broadcast a DataFrame when its size is appropriate for the available executor memory.

### Common join problems

- Duplicate join keys.
- Unexpected many-to-many matches.
- Null join keys.
- Data skew.
- Unnecessary columns.
- Incompatible data types.
- Large shuffles.

Before joining, check key uniqueness and record counts to ensure the result matches business expectations.

---

# 15. Window Functions

Window functions calculate values across related rows without reducing every group to one record.

Import:

```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F
```

## Rank employees by salary

```python
window_spec = Window.orderBy(F.col("salary").desc())

df = df.withColumn(
    "salary_rank",
    F.rank().over(window_spec)
)
```

## Ranking functions

- `row_number()`: Assigns a unique sequence within the window.
- `rank()`: Gives equal values the same rank and leaves gaps after ties.
- `dense_rank()`: Gives equal values the same rank without gaps.

## Rank within a department

```python
window_spec = (
    Window.partitionBy("department")
    .orderBy(F.col("salary").desc(), F.col("employee_id"))
)

df = df.withColumn(
    "department_rank",
    F.row_number().over(window_spec)
)
```

Adding a tie-breaker creates deterministic ordering when the tie-breaker is itself unique.

## Running total

```python
window_spec = (
    Window.partitionBy("customer_id")
    .orderBy("order_date", "order_id")
    .rowsBetween(Window.unboundedPreceding, Window.currentRow)
)

df = df.withColumn(
    "running_sales",
    F.sum("sales").over(window_spec)
)
```

## Previous row value

```python
window_spec = (
    Window.partitionBy("customer_id")
    .orderBy("order_date", "order_id")
)

df = df.withColumn(
    "previous_order_date",
    F.lag("order_date", 1).over(window_spec)
)
```

## Next row value

```python
df = df.withColumn(
    "next_order_date",
    F.lead("order_date", 1).over(window_spec)
)
```

Window functions are useful for:
- Latest-record selection.
- Deduplication.
- Running totals.
- Ranking.
- Customer purchase analysis.
- Month-over-month comparisons.
- Previous and next event analysis.

---

# 16. Deduplication Using Window Functions

Suppose a customer table contains multiple records per customer. You need to keep the latest record based on `updated_at`.

```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F

window_spec = (
    Window.partitionBy("customer_id")
    .orderBy(
        F.col("updated_at").desc_nulls_last(),
        F.col("ingestion_id").desc()
    )
)

deduplicated = (
    df.withColumn(
        "row_num",
        F.row_number().over(window_spec)
    )
    .filter(F.col("row_num") == 1)
    .drop("row_num")
)
```

The `ingestion_id` tie-breaker should represent a meaningful ordering rule. If it does not exist, select an appropriate stable source sequence or other business-defined tie-breaker.

This method is more explicit than `dropDuplicates(["customer_id"])` because it defines which record should survive.

---

# 17. Pivot and Unpivot

## Pivot

Pivot converts values from rows into columns.

```python
pivot_df = (
    df.groupBy("region")
    .pivot("quarter", ["Q1", "Q2", "Q3", "Q4"])
    .agg(F.sum("sales"))
)
```

Providing the expected pivot values avoids discovering an unrestricted set of pivot columns at runtime.

## Unpivot

On supported Spark versions, `unpivot()` converts selected columns into rows.

```python
unpivot_df = pivot_df.unpivot(
    ids=["region"],
    values=["Q1", "Q2", "Q3", "Q4"],
    variableColumnName="quarter",
    valueColumnName="sales"
)
```

The columns supplied to `unpivot()` must have compatible data types. If necessary, cast them to a common type first.

For older Spark versions, an `array()` of `struct()` expressions followed by `explode()` can implement similar logic.

---

# 18. Explode Arrays and Nested JSON

JSON records frequently contain arrays and nested objects.

Example input:

```json
{
  "customer_id": 101,
  "name": "Ravi",
  "orders": [
    {"order_id": 1, "amount": 100},
    {"order_id": 2, "amount": 250}
  ]
}
```

Assume the JSON has been loaded into a DataFrame called `df`.

```python
orders_df = df.select(
    "customer_id",
    "name",
    F.explode("orders").alias("order")
)

flattened_df = orders_df.select(
    "customer_id",
    "name",
    F.col("order.order_id").alias("order_id"),
    F.col("order.amount").alias("amount")
)
```

Important functions:
- `explode()`
- `explode_outer()`
- `posexplode()`
- `from_json()`
- `to_json()`
- `get_json_object()`
- `schema_of_json()`

Use explicit schemas for predictable production JSON pipelines. `explode()` removes rows where the array is null or empty; `explode_outer()` preserves the parent row with null output in applicable cases.

---

# 19. Temporary Views and Spark SQL

PySpark supports both DataFrame transformations and SQL queries.

## Create a temporary view

```python
df.createOrReplaceTempView("employees")
```

## Query using SQL

```python
result = spark.sql("""
    SELECT
        department,
        COUNT(*) AS employee_count,
        AVG(salary) AS average_salary
    FROM employees
    GROUP BY department
""")

result.show()
```

A local temporary view is scoped to the SparkSession. It is not a permanent database table.

For a registered table, use an appropriate catalog and schema.

```python
df = spark.table("main.silver.customers")
```

Use DataFrame APIs or SQL according to readability and maintainability. Both can benefit from Spark SQL optimization.

---

# 20. Lazy Evaluation and Actions

Spark normally delays computation until an action requires a result.

## Transformations

Examples:
- `select()`
- `filter()`
- `withColumn()`
- `join()`
- `groupBy()`
- `orderBy()`

## Actions

Examples:
- `show()`
- `count()`
- `collect()`
- `take()`
- `first()`
- `write.save()`

Example:

```python
df2 = df.filter(F.col("salary") > 50000)
df3 = df2.select("name", "salary")
```

These transformations typically build a plan. When an action executes, Spark determines and runs the necessary computation.

### Why is lazy evaluation useful?

- Enables query optimization.
- Avoids executing unused transformations.
- Allows operations to be combined into an efficient execution plan.

### Important warning

Avoid collecting large datasets to the driver.

```python
# Risky for large datasets
rows = df.collect()
```

Use distributed operations for large-scale processing. `collect()` is appropriate only when the result is known to be small enough for driver memory.

---

# 21. RDD vs. DataFrame vs. Dataset

| Feature | RDD | DataFrame | Dataset |
|---|---|---|---|
| Abstraction | Low-level distributed collection | Structured distributed data | Typed distributed collection |
| Schema | No built-in tabular schema | Yes | Yes, with type information |
| Optimization | Less structured query optimization | Catalyst and physical execution optimization | Structured optimization |
| Python support | Yes | Yes | Typed Dataset API is primarily for Scala/Java |
| Typical use | Specialized low-level operations | Most structured ETL | Typed JVM applications |

For Python-based Data Engineering, prefer DataFrames and Spark SQL for most structured workloads.

Use RDDs when a genuine low-level use case justifies them.

---

# 22. Partitioning and Repartitioning

A partition is a logical unit of data processed by Spark tasks.

## Repartition

```python
df = df.repartition(20)
```

This changes the number of partitions and usually involves a shuffle.

Partition by a column:

```python
df = df.repartition(20, "customer_id")
```

## Coalesce

```python
df = df.coalesce(5)
```

`coalesce()` usually reduces the number of partitions with less data movement than a full repartition. It cannot generally increase the partition count.

## Why partitioning matters

Too few partitions:
- Poor parallelism.
- Some executors may remain underutilized.

Too many partitions:
- Scheduling overhead.
- Excessively small tasks or output files.

Bad partitioning:
- Data skew.
- Uneven task execution.
- Excessive shuffle.

Do not assume that the number of partitions should equal the number of CPU cores. Choose based on data size, cluster resources, task duration, and the execution plan.

---

# 23. Shuffle Operations

A shuffle redistributes data across partitions.

Common causes:
- `groupBy()`
- `distinct()`
- `repartition()`
- `orderBy()`
- Many joins
- Window operations requiring redistribution or sorting

### Why are shuffles expensive?

They can require:
- Network transfers.
- Serialization.
- Disk I/O.
- Sorting and merging.
- Additional memory.

### How to reduce unnecessary shuffles

1. Filter data before expensive joins when logically valid.
2. Select only necessary columns.
3. Avoid unnecessary `distinct()`.
4. Use broadcast joins for suitable small tables.
5. Choose appropriate partitioning.
6. Avoid repeatedly repartitioning the same dataset.
7. Inspect Spark UI and query plans.
8. Address skewed join keys.

A shuffle is not always bad. Many analytical operations require one. The goal is to avoid unnecessary data movement.

---

# 24. Cache and Persist

Caching can prevent repeated computation when the same DataFrame is reused.

```python
df.cache()

df.count()
```

The action can materialize the cached data. Actual cache use depends on the storage level and execution.

Remove cached data when it is no longer needed:

```python
df.unpersist()
```

Use persistence when a dataset is expensive to compute and will be reused.

```python
from pyspark import StorageLevel

df.persist(StorageLevel.MEMORY_AND_DISK)
```

Do not cache every DataFrame. Caching consumes resources and can reduce performance when used unnecessarily.

---

# 25. Spark Query Optimization

Spark SQL uses the Catalyst optimizer and a physical execution engine to optimize supported queries.

## 25.1 Catalyst optimizer

Catalyst applies logical query optimizations and helps choose an efficient plan.

Examples of optimization techniques include:
- Predicate pushdown where supported.
- Column pruning.
- Constant folding.
- Simplification of expressions.
- Join optimization.

## 25.2 Physical execution plan

Inspect the execution plan:

```python
df.explain("formatted")
```

Look for:
- Table or file scans.
- Filters.
- Exchanges indicating shuffle boundaries.
- Sort operations.
- Join strategies.
- Aggregation operators.

## 25.3 Adaptive Query Execution

AQE can adapt certain execution decisions at runtime using observed statistics.

Depending on the Spark version and configuration, it can help with:
- Coalescing shuffle partitions.
- Converting certain sort-merge joins into broadcast joins.
- Handling some forms of skewed joins.

Check the runtime's configuration rather than assuming all features are enabled.

## 25.4 Practical optimization checklist

- Use Parquet or Delta for analytical storage.
- Avoid reading unnecessary columns.
- Apply valid filters early.
- Avoid Python UDFs when built-in functions can express the logic.
- Broadcast only suitably small datasets.
- Monitor shuffle size and spill.
- Handle skewed keys.
- Tune partition counts based on evidence.
- Avoid unnecessary caching.
- Avoid excessive small files.
- Use table statistics and supported data layout optimizations.
- Review Spark UI before changing cluster size.

---

# 26. Built-in Functions vs. UDFs

A UDF is a user-defined function that implements custom logic.

Prefer built-in functions because Spark can often optimize them more effectively.

Example using built-in functions:

```python
df = df.withColumn(
    "normalized_name",
    F.upper(F.trim(F.col("name")))
)
```

A Python UDF:

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

def normalize_name(value):
    if value is None:
        return None
    return value.strip().upper()

normalize_udf = udf(normalize_name, StringType())

df = df.withColumn(
    "normalized_name",
    normalize_udf("name")
)
```

Python UDFs can introduce serialization and execution overhead. They are useful when the required logic cannot reasonably be expressed using built-in Spark functions.

Pandas UDFs use Apache Arrow to support vectorized processing in applicable scenarios, but they still require careful memory and performance management.

---

# 27. Writing Data

## Write CSV

```python
df.write.mode("overwrite").option(
    "header", True
).csv("/output/customers_csv")
```

Spark writes a directory containing output part files, not necessarily one file with the exact name you expect.

## Write Parquet

```python
df.write.mode("overwrite").parquet("/output/customers_parquet")
```

## Write Delta

```python
df.write.format("delta").mode(
    "overwrite"
).save("/output/customers_delta")
```

## Append data

```python
df.write.format("delta").mode(
    "append"
).save("/output/customers_delta")
```

## Save as a registered table

```python
df.write.format("delta").mode(
    "overwrite"
).saveAsTable("main.silver.customers")
```

Use a catalog, schema, and storage location appropriate for the environment.

## Save with partition columns

```python
df.write.format("delta").mode(
    "append"
).partitionBy("order_year", "order_month").save(
    "/output/partitioned_orders"
)
```

Partitioning by high-cardinality columns, such as unique customer IDs, can create excessive directories and small files.

---

# 28. Delta Lake

Delta Lake adds a transaction log and table-management capabilities to Parquet-based data storage.

Common capabilities include:
- ACID transactions.
- Schema enforcement.
- Schema evolution under supported configurations.
- `MERGE` operations.
- Time travel.
- Version history.
- Batch and streaming integration.
- Table maintenance features.

The exact availability and syntax of advanced features depend on the Delta Lake version and platform.

## Create a Delta table

```python
df.write.format("delta").mode(
    "overwrite"
).save("/data/silver/customers")
```

## Read Delta data

```python
df = spark.read.format("delta").load(
    "/data/silver/customers"
)
```

## Merge updates and inserts

In a compatible Delta environment:

```python
from delta.tables import DeltaTable

target = DeltaTable.forPath(
    spark,
    "/data/silver/customers"
)

source_df = source_df.dropDuplicates(["customer_id"])

(
    target.alias("t")
    .merge(
        source_df.alias("s"),
        "t.customer_id = s.customer_id"
    )
    .whenMatchedUpdateAll()
    .whenNotMatchedInsertAll()
    .execute()
)
```

This is a template. Before using it, validate the source and target schemas, key uniqueness, deletion rules, and update policy. Multiple source rows matching the same target row can make a merge ambiguous or fail.

## Schema evolution

Schema evolution can be useful when legitimate new columns appear in source data. Enable it deliberately and according to the platform's supported settings.

Do not automatically accept every schema change in a critical production table.

---

# 29. Medallion Architecture

Medallion Architecture organizes data into layers according to data quality and business readiness.

## Bronze layer — Raw data

Purpose:
- Preserve ingested source data.
- Keep source information for traceability.
- Capture ingestion metadata.
- Support replay and investigation.

Typical operations:
- Ingestion.
- Basic structural validation.
- Source metadata creation.
- Initial storage.

## Silver layer — Cleaned data

Purpose:
- Produce reliable, standardized datasets.

Typical operations:
- Data type conversion.
- Null handling.
- Deduplication.
- Data quality validation.
- Standardized columns.
- Business key validation.
- Joining related datasets.

## Gold layer — Business-ready data

Purpose:
- Serve analytical and reporting use cases.

Typical operations:
- Aggregations.
- Business metrics.
- Dimensional models.
- Fact and dimension tables.
- KPI calculations.
- Reporting tables.

### Example architecture

```text
API / Database / CSV / JSON
             |
             v
       Ingestion Layer
      ADF / Auto Loader
             |
             v
     ADLS Gen2 / Landing
             |
             v
       BRONZE LAYER
        Raw Delta
             |
             v
        SILVER LAYER
    Cleaned and Validated
             |
             v
         GOLD LAYER
      Aggregated Metrics
             |
             v
    Power BI / SQL Analytics
```

Bronze, Silver, and Gold can be implemented using Delta tables, catalog schemas, storage directories, or a combination of these. They are logical data layers, not mandatory physical storage locations.

---

# 30. End-to-End PySpark Project

## Project: E-Commerce Data Engineering Pipeline

### Objective

Build a pipeline that processes customers, products, and orders and produces clean sales analytics.

### Source tables

**Customers**

| customer_id | customer_name | city |
|---|---|---|
| 101 | Ravi | Hyderabad |
| 102 | Priya | Chennai |
| 103 | Kiran | Bengaluru |

**Orders**

| order_id | customer_id | amount | status |
|---|---|---:|---|
| 1001 | 101 | 1500 | Completed |
| 1002 | 102 | 2500 | Completed |
| 1003 | 101 | 500 | Cancelled |
| 1004 | 103 | 3000 | Completed |

### Step 1: Create sample data

```python
customers = [
    (101, "Ravi", "Hyderabad"),
    (102, "Priya", "Chennai"),
    (103, "Kiran", "Bengaluru")
]

orders = [
    (1001, 101, 1500.0, "Completed"),
    (1002, 102, 2500.0, "Completed"),
    (1003, 101, 500.0, "Cancelled"),
    (1004, 103, 3000.0, "Completed")
]

customers_df = spark.createDataFrame(
    customers,
    ["customer_id", "customer_name", "city"]
)

orders_df = spark.createDataFrame(
    orders,
    ["order_id", "customer_id", "amount", "status"]
)
```

### Step 2: Create Bronze tables

```python
customers_df.write.format("delta").mode(
    "overwrite"
).save("/data/bronze/customers")

orders_df.write.format("delta").mode(
    "overwrite"
).save("/data/bronze/orders")
```

In a real project, the Bronze layer would usually ingest source files or tables, preserve source metadata, and use an appropriate incremental loading strategy.

### Step 3: Read Bronze data

```python
customers_bronze = spark.read.format(
    "delta"
).load("/data/bronze/customers")

orders_bronze = spark.read.format(
    "delta"
).load("/data/bronze/orders")
```

### Step 4: Clean customer data

```python
customers_silver = (
    customers_bronze
    .withColumn(
        "customer_name",
        F.trim(F.col("customer_name"))
    )
    .withColumn(
        "city",
        F.initcap(F.trim(F.col("city")))
    )
    .filter(F.col("customer_id").isNotNull())
    .filter(F.col("customer_name").isNotNull())
    .dropDuplicates(["customer_id"])
)
```

For production, replace the arbitrary duplicate-removal step with a deterministic latest-record rule if duplicate customer IDs can contain different versions.

### Step 5: Clean order data

```python
orders_silver = (
    orders_bronze
    .withColumn(
        "order_id",
        F.col("order_id").cast("long")
    )
    .withColumn(
        "customer_id",
        F.col("customer_id").cast("long")
    )
    .withColumn(
        "amount",
        F.col("amount").cast("decimal(18,2)")
    )
    .withColumn(
        "status",
        F.upper(F.trim(F.col("status")))
    )
    .filter(F.col("order_id").isNotNull())
    .filter(F.col("customer_id").isNotNull())
    .dropDuplicates(["order_id"])
)
```

Validate cast failures, duplicate keys, negative amounts, and invalid statuses before publishing the Silver dataset.

### Step 6: Write Silver tables

```python
customers_silver.write.format("delta").mode(
    "overwrite"
).save("/data/silver/customers")

orders_silver.write.format("delta").mode(
    "overwrite"
).save("/data/silver/orders")
```

These overwrite operations are suitable for this small demonstration, not automatically appropriate for production incremental pipelines.

### Step 7: Join customers and orders

```python
customer_orders = (
    orders_silver.alias("o")
    .join(
        customers_silver.alias("c"),
        on="customer_id",
        how="left"
    )
    .select(
        "customer_id",
        "customer_name",
        "city",
        "order_id",
        "amount",
        "status"
    )
)
```

### Step 8: Calculate business metrics

```python
gold_sales = (
    customer_orders
    .filter(F.col("status") == "COMPLETED")
    .groupBy("customer_id", "customer_name", "city")
    .agg(
        F.countDistinct("order_id").alias("total_orders"),
        F.sum("amount").alias("total_sales"),
        F.avg("amount").alias("average_order_value")
    )
)
```

Expected results:

| Customer | Completed orders | Total sales |
|---|---:|---:|
| Ravi | 1 | 1500 |
| Priya | 1 | 2500 |
| Kiran | 1 | 3000 |

### Step 9: Write the Gold layer

```python
gold_sales.write.format("delta").mode(
    "overwrite"
).save("/data/gold/customer_sales")
```

### Step 10: Validate the output

```python
gold_sales.show(truncate=False)

gold_sales.printSchema()

assert gold_sales.filter(
    F.col("total_sales") < 0
).count() == 0
```

This is a basic demonstration. A production pipeline also needs schema contracts, quality thresholds, monitoring, incremental processing, access control, and recovery procedures.

---

# 31. Incremental Loading

Incremental loading processes new or changed records rather than reprocessing the entire dataset every time.

Common approaches:

1. Source watermark or modification timestamp.
2. Database change tracking or change data capture.
3. Auto Loader for incrementally discovered files.
4. Structured Streaming.
5. Delta Change Data Feed where supported and enabled.
6. Source-specific sequence numbers or event offsets.

## Example using a watermark

```python
last_watermark = "2026-10-09 00:00:00"

incremental_df = source_df.filter(
    F.col("updated_at") > F.to_timestamp(
        F.lit(last_watermark)
    )
)
```

This illustrates the filter only. A production watermark design must address:
- Late-arriving records.
- Records updated with the same timestamp.
- Timestamp precision.
- Time zones.
- Failed pipeline retries.
- Atomic persistence of data and watermark state.

For reliable pipelines, use a source-supported change sequence or a carefully designed composite watermark where possible.

## Idempotency

An idempotent pipeline can be rerun without producing unintended duplicate or inconsistent results.

Common practices:
- Stable business keys.
- Deterministic deduplication.
- Delta `MERGE` for upserts.
- Checkpointing where applicable.
- Transaction-aware state management.
- Controlled retries.
- Atomic or recoverable watermark updates.

Appending the same source batch repeatedly without deduplication or checkpointing is not automatically idempotent.

---

# 32. Structured Streaming

Structured Streaming processes continuously arriving data using Spark's structured APIs.

Typical sources include:
- Kafka.
- File streams.
- Cloud object storage.
- Other supported streaming sources.

Availability and connector details depend on the Spark platform and runtime.

## Example: Read streaming CSV files

```python
stream_df = (
    spark.readStream
    .schema(schema)
    .option("header", True)
    .csv("/data/stream_input")
)
```

## Write a streaming output

```python
query = (
    stream_df.writeStream
    .format("delta")
    .outputMode("append")
    .option("checkpointLocation", "/data/checkpoints/orders")
    .start("/data/stream_output")
)
```

## Important streaming concepts

- Trigger interval.
- Output mode.
- Checkpointing.
- Event time.
- Processing time.
- Watermarks.
- Late-arriving events.
- Stateful aggregations.
- Exactly-once processing guarantees and sink semantics.

A checkpoint records progress and state for recovery. Use a durable checkpoint location and avoid reusing the same checkpoint for unrelated streaming queries.

Do not assume that every source and sink combination provides identical end-to-end delivery guarantees.

---

# 33. Auto Loader in Azure Databricks

Auto Loader incrementally discovers and processes files from supported cloud storage locations.

It is commonly used to ingest files from ADLS Gen2 into a Bronze Delta table.

Example:

```python
source_path = "abfss://container@storageaccount.dfs.core.windows.net/landing/orders/"
checkpoint_path = "abfss://container@storageaccount.dfs.core.windows.net/checkpoints/orders/"
schema_path = "abfss://container@storageaccount.dfs.core.windows.net/schemas/orders/"

orders_stream = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", schema_path)
    .load(source_path)
)

(
    orders_stream.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .outputMode("append")
    .start("/data/bronze/orders")
)
```

This is an illustrative Databricks pattern. Replace all paths with valid locations and configure authentication, schema evolution, rescue behavior, and checkpoint permissions.

Auto Loader is a Databricks capability, not a generic feature available in every self-managed Apache Spark installation.

---

# 34. Azure Databricks Integration

Azure Databricks combines managed Spark compute with collaborative notebooks, workflows, and lakehouse capabilities.

## Typical end-to-end architecture

```text
Source APIs / SQL / CSV / JSON
              |
              v
     Azure Data Factory
        Ingestion
              |
              v
         ADLS Gen2
         Raw Files
              |
              v
      Azure Databricks
           PySpark
              |
              v
        Bronze Delta
              |
              v
        Silver Delta
              |
              v
         Gold Delta
              |
              v
    SQL Analytics / Power BI
```

Not every project needs all these services. The architecture depends on the data source, transformation requirements, serving layer, and operational needs.

## Important Azure Databricks concepts

- Workspace.
- Notebook.
- Compute.
- Databricks Runtime.
- SparkSession.
- Jobs and Workflows.
- Unity Catalog.
- Catalogs and schemas.
- Managed and external tables.
- Volumes.
- External locations.
- Storage credentials.
- Delta Lake.
- Auto Loader.
- Lakeflow Jobs.
- Monitoring and access control.

### Unity Catalog

Unity Catalog provides centralized governance for supported data assets and access control.

A fully qualified table name commonly follows:

```text
catalog.schema.table
```

Example:

```python
df = spark.table("main.silver.customers")
```

### ADLS Gen2

Azure Data Lake Storage Gen2 is cloud storage designed for data lake workloads.

Use appropriate identity-based authentication and permissions. In governed Databricks environments, Unity Catalog external locations and storage credentials are often preferable to unmanaged credentials embedded in notebooks.

### Azure Data Factory

Azure Data Factory can orchestrate ingestion and data movement between supported sources and destinations.

It can invoke Databricks jobs and coordinate dependencies, retries, and schedules. PySpark performs the distributed transformations inside the Spark environment.

### Power BI

Power BI can consume curated Gold datasets through an appropriate connector or serving layer.

Use business-friendly schemas, documented measures, stable keys, and validated aggregations to support reliable reports.

---

# 35. Error Handling and Data Quality

A production pipeline must not merely run; it must detect invalid data and report failures clearly.

## Validate required columns

```python
required_columns = ["customer_id", "customer_name"]

missing_columns = [
    column
    for column in required_columns
    if column not in df.columns
]

if missing_columns:
    raise ValueError(
        f"Missing required columns: {missing_columns}"
    )
```

## Identify invalid records

```python
invalid_df = df.filter(
    F.col("customer_id").isNull()
    | F.col("customer_name").isNull()
)
```

## Separate valid and invalid records

```python
valid_df = df.filter(
    F.col("customer_id").isNotNull()
    & F.col("customer_name").isNotNull()
)
```

In production, persist invalid records to a quarantine location with ingestion identifiers, failure reasons, and enough source information for investigation.

## Use logging

```python
import logging

logger = logging.getLogger("customer_pipeline")
logger.setLevel(logging.INFO)

logger.info("Starting customer pipeline")
```

Configure the appropriate logging handlers for your execution environment. In Databricks, use platform logs and job monitoring as part of the operational design.

## Use exception handling appropriately

```python
try:
    valid_df.write.format("delta").mode(
        "append"
    ).save("/data/silver/customers")

except Exception:
    logger.exception("Failed to write Silver customers")
    raise
```

Re-raising the exception ensures that the orchestration system can recognize the task as failed.

Avoid catching all exceptions and continuing silently.

---

# 36. Unit Testing PySpark

Testing helps verify business logic before deploying pipelines.

A basic example using Python assertions:

```python
actual_count = valid_df.count()

assert actual_count >= 0
```

For meaningful tests, verify specific expected outcomes.

```python
actual = (
    spark.createDataFrame(
        [(1, 100), (1, 200), (2, 50)],
        ["customer_id", "amount"]
    )
    .groupBy("customer_id")
    .agg(F.sum("amount").alias("total_amount"))
)
```

Test:
- Expected record counts.
- Required columns.
- Schema compatibility.
- Duplicate keys.
- Null handling.
- Join cardinality.
- Aggregate totals.
- Invalid source records.
- Incremental reruns.

Use `pytest` or `unittest` and compare actual results with known expected results.

For distributed pipelines, make tests small and deterministic. Include edge cases such as empty inputs, nulls, duplicate keys, and malformed data.

---

# 37. Common PySpark Errors

## Error: Column is ambiguous

Cause: Multiple joined DataFrames contain a column with the same name.

Solution:

```python
a = df1.alias("a")
b = df2.alias("b")

result = a.join(
    b,
    F.col("a.customer_id") == F.col("b.customer_id"),
    "inner"
).select(
    F.col("a.customer_id").alias("customer_id"),
    F.col("a.name"),
    F.col("b.order_id")
)
```

## Error: Cannot resolve column

Possible causes:
- Incorrect column name.
- Column dropped earlier.
- Typographical error.
- Incorrect nested-field path.
- Case-sensitivity or schema mismatch.

Inspect:

```python
df.printSchema()
print(df.columns)
```

## Error: Out of memory

Possible causes:
- Collecting too much data to the driver.
- Oversized broadcast data.
- Data skew.
- Large cached datasets.
- Excessive per-task memory requirements.

Solutions depend on the cause. Inspect driver and executor logs, task metrics, and the Spark UI before increasing resources.

## Error: Too many small files

Possible causes:
- Excessive output partitions.
- Frequent tiny streaming batches.
- High-cardinality storage partitioning.

Possible remedies:
- Tune output partitioning.
- Avoid high-cardinality directory partitioning.
- Use supported compaction or table optimization capabilities.
- Review write frequency and file sizes.

## Error: Shuffle fetch failure

Possible causes:
- Executor failures.
- Lost shuffle files.
- Network issues.
- Resource pressure.

Inspect executor logs and infrastructure health. Retrying may help transient failures, but recurring failures require investigation.

## Error: Delta merge conflict

Possible causes:
- Multiple source rows match one target row.
- Concurrent writes.
- Inconsistent schemas.
- Invalid merge conditions.

Deduplicate the source deterministically, verify the merge key, and review transaction conflicts.

## Error: Schema mismatch

Possible causes:
- Source column types changed.
- Missing required columns.
- Incorrect data types.
- Unexpected schema evolution.

Compare source and target schemas and apply an explicit schema contract.

---

# 38. PySpark Performance Tuning Checklist

Before deploying a pipeline, check the following:

1. Read only the required columns.
2. Apply valid filters early.
3. Use explicit schemas where practical.
4. Use Parquet or Delta for analytical datasets.
5. Avoid unnecessary `collect()`.
6. Avoid unnecessary `toPandas()`.
7. Prefer Spark built-in functions over Python UDFs.
8. Check for expensive shuffles.
9. Use broadcast joins only when appropriate.
10. Handle skewed join keys.
11. Tune partition counts based on measurements.
12. Avoid excessive caching.
13. Avoid excessive small files.
14. Review the physical query plan.
15. Inspect Spark UI stage and task metrics.
16. Validate source and target record counts.
17. Monitor failed tasks, spill, and executor memory.
18. Use supported AQE capabilities.
19. Make retries safe.
20. Test with realistic data volumes before production.

---

# 39. Batch Processing vs. Streaming

| Feature | Batch | Streaming |
|---|---|---|
| Processing | Bounded datasets | Continuously arriving or incrementally processed data |
| Trigger | Scheduled or manually started | Continuous or micro-batch execution |
| Latency | Usually minutes or longer | Seconds or longer, depending on design |
| State | Often managed per job | May require persistent state and checkpoints |
| Typical example | Daily sales report | Live event processing |
| PySpark API | `spark.read` | `spark.readStream` |

Structured Streaming supports incremental processing with different trigger configurations. It does not mean every application has true record-by-record, sub-second execution.

Choose batch or streaming based on business latency requirements, source characteristics, complexity, and cost.

---

# 40. Important PySpark Interview Questions

### Fundamentals

**1. What is PySpark?**

PySpark is the Python API for Apache Spark, used to process structured and unstructured data in distributed computing environments.

**2. What is SparkSession?**

SparkSession is the primary entry point for Spark DataFrame and SQL operations.

**3. What is a DataFrame?**

A distributed dataset organized into named columns with a schema.

**4. What is lazy evaluation?**

Spark delays executing most transformations until an action requires their results.

**5. What is a partition?**

A logical portion of distributed data processed by Spark tasks.

**6. What is a shuffle?**

Redistribution of data between partitions, often involving network transfers and disk I/O.

**7. What is the difference between transformations and actions?**

Transformations define computations; actions trigger execution or return results.

### Data processing

**8. What is the difference between `repartition()` and `coalesce()`?**

`repartition()` reshuffles data to change partition distribution. `coalesce()` typically reduces partitions with less data movement.

**9. What is the difference between `rank()`, `dense_rank()`, and `row_number()`?**

`rank()` leaves gaps after ties, `dense_rank()` does not, and `row_number()` assigns unique sequence numbers.

**10. What is a broadcast join?**

A join strategy that distributes a sufficiently small dataset to executors to reduce data shuffling on the other side.

**11. What is the difference between `cache()` and `persist()`?**

`cache()` uses the default caching storage level; `persist()` lets you select a supported storage level.

**12. What is a window function?**

A function that calculates results over a related group of rows without necessarily reducing them to one row per group.

**13. How do you remove duplicates while keeping the latest record?**

Use `row_number()` over a window partitioned by the business key and ordered by the timestamp and a deterministic tie-breaker.

### Architecture and optimization

**14. What is the Catalyst optimizer?**

Spark SQL's optimizer, which applies logical optimizations and helps produce an efficient execution plan.

**15. What is AQE?**

Adaptive Query Execution adjusts certain execution decisions at runtime based on observed statistics.

**16. What causes data skew?**

Uneven distribution of data, often caused by a small number of heavily repeated keys.

**17. How do you troubleshoot a slow Spark job?**

Inspect the Spark UI, stage duration, shuffle metrics, task imbalance, spill, query plan, join strategies, and data sizes.

**18. Why avoid `collect()`?**

It transfers the complete result to driver memory and may cause driver memory exhaustion.

### Delta Lake and Data Engineering

**19. What is Delta Lake?**

A storage layer that adds transaction-log-based table management and capabilities such as ACID transactions to lakehouse data.

**20. What is Medallion Architecture?**

A layered data organization pattern using Bronze for raw data, Silver for cleaned data, and Gold for business-ready datasets.

**21. What is an incremental load?**

A load that processes new or changed data rather than repeatedly processing the entire dataset.

**22. What is an idempotent pipeline?**

A pipeline that can be rerun without creating unintended duplicate or inconsistent outcomes.

**23. What is Auto Loader?**

A Databricks capability for incrementally discovering and ingesting files from supported cloud storage locations.

**24. What is checkpointing?**

Persisting progress and state so supported streaming applications can recover after interruption.

**25. What is the difference between a DataFrame and an RDD?**

DataFrames provide a schema and benefit from structured query optimization; RDDs offer a lower-level distributed collection API.

---

# 41. Recommended Learning Order

Follow this order to build practical confidence.

1. Python fundamentals.
2. SparkSession and DataFrames.
3. Reading and writing files.
4. DataFrame transformations.
5. Built-in functions and data types.
6. Null handling and data quality.
7. Joins and aggregations.
8. Window functions and deduplication.
9. Spark SQL.
10. Lazy evaluation and execution plans.
11. Partitions and shuffles.
12. Performance optimization.
13. Delta Lake.
14. Medallion Architecture.
15. Incremental loading and `MERGE`.
16. Structured Streaming.
17. Auto Loader.
18. Azure Databricks and Unity Catalog.
19. ADLS Gen2 and Azure Data Factory integration.
20. Testing, logging, monitoring, and deployment.

Build a small project after each major section instead of memorizing all the APIs without practice.

---

# 42. Official Documentation

Use official documentation to verify syntax and version-specific behavior.

- Apache Spark: https://spark.apache.org/docs/latest/
- PySpark API Reference: https://spark.apache.org/docs/latest/api/python/
- Spark SQL Guide: https://spark.apache.org/docs/latest/sql-programming-guide.html
- Spark Structured Streaming: https://spark.apache.org/docs/latest/streaming/index.html
- Spark Tuning Guide: https://spark.apache.org/docs/latest/tuning.html
- Delta Lake: https://docs.delta.io/
- Azure Databricks documentation: https://learn.microsoft.com/azure/databricks/
- Azure Data Factory: https://learn.microsoft.com/azure/data-factory/
- Azure Data Lake Storage: https://learn.microsoft.com/azure/storage/blobs/data-lake-storage-introduction

Some examples in this guide are templates. Confirm API compatibility with your installed Spark version or Databricks Runtime before running them in production.

---

# 43. Final Revision Checklist

Before considering yourself comfortable with PySpark, make sure you can explain and implement:

- [ ] Spark architecture: driver, executors, jobs, stages, and tasks.
- [ ] PySpark libraries and their purposes.
- [ ] DataFrame creation and schema definition.
- [ ] CSV, JSON, Parquet, and Delta ingestion.
- [ ] Transformations, actions, and lazy evaluation.
- [ ] Null handling and data type conversion.
- [ ] String and date functions.
- [ ] Joins, grouping, and aggregations.
- [ ] Window functions and deterministic deduplication.
- [ ] Pivot, unpivot, and nested JSON processing.
- [ ] RDD vs. DataFrame.
- [ ] Partitioning, shuffles, caching, and broadcast joins.
- [ ] Catalyst, AQE, and query-plan analysis.
- [ ] Delta Lake tables and merge operations.
- [ ] Bronze, Silver, and Gold layers.
- [ ] Incremental loads and idempotency.
- [ ] Structured Streaming and checkpointing.
- [ ] Auto Loader and cloud storage integration.
- [ ] Azure Databricks, Unity Catalog, and ADLS Gen2.
- [ ] Data quality, error handling, testing, and monitoring.
- [ ] End-to-end project implementation and interview explanations.

## Final advice

Treat this document as your master reference. Practice the examples in a Databricks notebook or a compatible local Spark environment, and add your own tested examples as you learn.

For a professional GitHub repository, keep this file as `PYSPARK_END_TO_END_NOTES.md` and add separate folders for projects, notebooks, sample datasets, and troubleshooting notes. This makes the material easier to maintain as you progress from PySpark fundamentals to production-level Azure Data Engineering.
