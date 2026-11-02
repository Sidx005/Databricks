# Databricks Data Engineering Concepts

**A Practical Guide to Data Ingestion, Lakeflow Pipelines, CDC, SCD, and Jobs**

---

## Table of Contents

1. [The Big Picture](#1-the-big-picture)
2. [Data Ingestion](#2-data-ingestion)
3. [Pipelines](#3-pipelines)
4. [Streaming Tables](#4-streaming-tables)
5. [Change Data Capture (CDC)](#5-change-data-capture-cdc)
6. [Slowly Changing Dimensions (SCD)](#6-slowly-changing-dimensions-scd)
7. [Lakeflow Jobs](#7-lakeflow-jobs)
8. [Flows in Lakeflow Pipelines](#8-flows-in-lakeflow-pipelines)
9. [Materialized Views](#9-materialized-views)
10. [Incremental Processing](#10-incremental-processing)
11. [DAGs](#11-dags)
12. [How Everything Works Together](#12-how-everything-works-together)
13. [Quick Revision](#13-quick-revision)
14. [Knowledge Check](#14-knowledge-check)

---

# 1. The Big Picture

Imagine an e-commerce company that receives thousands of orders every day.

The company receives data from multiple sources:

- Application databases.
- Customer registration systems.
- Payment systems.
- CSV and JSON files.
- External APIs.
- Event streaming platforms.

The company wants to:

1. Bring data from different sources into Databricks.
2. Clean and transform the raw data.
3. Handle newly inserted and updated records.
4. Maintain historical data where required.
5. Store processed data for analytics.
6. Generate business reports.
7. Automate the entire workflow.

This is where data ingestion, pipelines, streaming tables, CDC, SCD, and Jobs come into the picture.

## 1.1 Complete Data Engineering Workflow

```text
                    DATA SOURCES
          ┌──────────┬───────────┬──────────┐
          │ Database │ CSV Files │   APIs   │
          └──────────┴───────────┴──────────┘
                         |
                         v
                  DATA INGESTION
                         |
                         v
             LAKEFLOW DECLARATIVE PIPELINE
                         |
             ┌───────────┼───────────┐
             |           |           |
             v           v           v
          BRONZE      SILVER       GOLD
             |           |           |
          Raw Data   Clean Data  Business Data
             |           |           |
             └───────────┼───────────┘
                         |
                         v
               ANALYTICS & REPORTING
                         |
                         v
              Dashboards / SQL / BI Tools

       LAKEFLOW JOBS
       Orchestrate and schedule workloads
```

**Important distinction:**

- A **Pipeline** defines and processes data transformations.
- A **Job** coordinates when and how different workloads execute.

A Job can execute a Pipeline as one of its tasks, but a Pipeline does not necessarily need a Job to run.

---

# 2. Data Ingestion

## 2.1 What is Data Ingestion?

Data ingestion is the process of collecting data from different sources and bringing it into a data platform such as Databricks.

For example, suppose an e-commerce application has a database containing customer orders.

The source table looks like this:

| order_id | customer | amount |
|---|---|---:|
| 101 | John | 500 |
| 102 | Alice | 800 |
| 103 | Bob | 300 |

This data currently resides in the application's database.

Bringing this data into Databricks for further processing is called **Data Ingestion**.

## 2.2 Sources of Data

| Source Type | Examples |
|---|---|
| Databases | MySQL, PostgreSQL, SQL Server |
| Files | CSV, JSON, Parquet, XML |
| APIs | REST APIs |
| Cloud Storage | AWS S3, Azure Data Lake, Google Cloud Storage |
| Event Streams | Kafka, Event Hubs |

## 2.3 Types of Data Ingestion

### A. Batch Ingestion

Batch ingestion collects and processes data at specific intervals.

**Example:**

- Every day at midnight.
- Collect all orders created during the previous day.
- Load them into Databricks.

```text
Source Database
      |
      v
Collect records every day
      |
      v
Databricks
```

**Use cases:**

- Daily sales reports.
- Monthly financial reports.
- Periodic data synchronization.

### B. Streaming Ingestion

Streaming ingestion continuously processes incoming data as it arrives.

**Example:**

1. A customer places an order.
2. The application generates an order event.
3. Databricks receives the event.
4. The data becomes available for downstream processing.

```text
Customer places order
         |
         v
     Order Event
         |
         v
   Streaming Source
         |
         v
     Databricks
```

**Use cases:**

- Real-time order monitoring.
- Fraud detection.
- Live transaction monitoring.

## 2.4 Batch vs Streaming

| Batch | Streaming |
|---|---|
| Processes data periodically. | Processes incoming data continuously or in micro-batches. |
| Suitable for scheduled loads. | Suitable for near-real-time processing. |
| Example: Daily sales report. | Example: Live order monitoring. |

## 2.5 Reading Data in Spark

| Method | Purpose |
|---|---|
| `spark.read()` | Reads data in batch mode. |
| `spark.readStream()` | Reads data using streaming semantics. |

Example:

```python
# Batch read
df = spark.read.format("csv").load("/path/to/orders/")

# Streaming read
df = spark.readStream.format("csv").load("/path/to/orders/")
```

### What is Auto Loader?

Auto Loader is a Databricks feature used to incrementally discover and ingest new files from cloud storage.

For example, suppose new CSV files continuously arrive in a storage directory.

Instead of manually identifying and reading every new file, Auto Loader tracks discovered files and processes newly arriving files incrementally.

It commonly uses the `cloudFiles` format.

```python
df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "csv")
    .load("/path/to/orders/")
)
```

**Remember:**

- Batch ingestion processes data periodically.
- Streaming ingestion processes incoming data continuously or in micro-batches.
- Auto Loader is commonly used for incremental file ingestion.

---

# 3. Pipelines

## 3.1 What is a Pipeline?

A Pipeline is a sequence of data processing operations that transforms raw data into useful data.

Suppose you ingest order data into Databricks.

The raw data may contain:

- Missing customer names.
- Duplicate orders.
- Incorrect amounts.
- Invalid dates.

You need to clean and transform this data before using it for analytics.

A Pipeline defines these transformations and their dependencies.

## 3.2 Medallion Architecture

Databricks commonly organizes data using the Medallion Architecture.

It consists of three layers:

### Bronze Layer – Raw Data

Stores ingested data with minimal transformation.

Example:

Original order records received from the application.

```text
orders_bronze
```

### Silver Layer – Cleaned Data

Contains cleaned, validated, and transformed data.

Typical operations:

- Remove duplicates.
- Handle missing values.
- Validate records.
- Standardize formats.
- Apply business rules.

```text
orders_silver
```

### Gold Layer – Business-Ready Data

Contains data prepared for analytics and reporting.

Typical operations:

- Aggregations.
- Business metrics.
- Fact and dimension tables.
- Reporting datasets.

Example:

Daily revenue by product and region.

```text
daily_revenue_gold
```

### Medallion Architecture Overview

```text
             SOURCE DATA
                  |
                  v
          ┌───────────────┐
          │    BRONZE     │
          │   Raw Data    │
          └───────────────┘
                  |
                  v
          ┌───────────────┐
          │    SILVER     │
          │  Clean Data   │
          └───────────────┘
                  |
                  v
          ┌───────────────┐
          │     GOLD      │
          │ Business Data │
          └───────────────┘
                  |
                  v
          ANALYTICS & BI
```

## 3.3 What is Lakeflow Declarative Pipelines?

Lakeflow Declarative Pipelines is a Databricks framework for building and maintaining data pipelines.

It was previously known as **Delta Live Tables (DLT)**.

It allows you to define:

- Streaming tables.
- Materialized views.
- Data quality expectations.
- Dependencies between datasets.
- Incremental data processing.
- Change Data Capture workflows.

The key idea is that you define what the output dataset should look like, and Databricks manages pipeline execution and dependencies.

## 3.4 Example: Creating a Materialized View

```python
from pyspark import pipelines as dp

@dp.materialized_view
def clean_orders():
    return (
        spark.read.table("orders_raw")
        .filter("amount > 0")
    )
```

Here:

- `dp` is the newer Spark Declarative Pipelines Python API.
- `@dp.materialized_view` defines a materialized view.
- `spark.read.table()` reads the source table.
- `.filter()` applies a transformation.

Databricks identifies the dependency between the source and the output dataset.

## 3.5 Traditional ETL vs Declarative Pipelines

| Traditional ETL | Declarative Pipeline |
|---|---|
| You write execution logic. | You define the desired datasets. |
| Dependencies are often managed manually. | Dependencies are automatically identified. |
| More manual orchestration. | Built-in pipeline orchestration. |
| Quality checks are implemented separately. | Supports built-in data quality expectations. |

**Remember:** A declarative pipeline focuses on defining the desired result rather than manually controlling every execution step.

---

# 4. Streaming Tables

## 4.1 What is a Streaming Table?

A Streaming Table is a table designed for incremental data ingestion and processing using streaming semantics.

Imagine your e-commerce company receives:

- 10,000 orders on Day 1.
- 5,000 additional orders on Day 2.

You do not necessarily want to reload all 15,000 records every time.

You want to process newly arriving records incrementally.

That is where Streaming Tables are useful.

## 4.2 Example

### Day 1 – Initial Data

| order_id | customer | amount |
|---|---|---:|
| 101 | John | 500 |
| 102 | Alice | 800 |

The Streaming Table ingests these records.

### Day 2 – New Data

| order_id | customer | amount |
|---|---|---:|
| 103 | Bob | 300 |
| 104 | Sam | 900 |

During the next pipeline update, the Streaming Table processes newly discovered records.

### Conceptual Output

| order_id | customer | amount |
|---|---|---:|
| 101 | John | 500 |
| 102 | Alice | 800 |
| 103 | Bob | 300 |
| 104 | Sam | 900 |

## 4.3 Creating a Streaming Table

Using the newer `dp` API:

```python
from pyspark import pipelines as dp

@dp.table
def orders_raw():
    return (
        spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "csv")
        .schema(order_schema)
        .load("/path/to/orders/")
    )
```

Explanation:

| Code | Purpose |
|---|---|
| `@dp.table` | Defines a pipeline table. |
| `spark.readStream` | Creates a streaming DataFrame. |
| `cloudFiles` | Enables Auto Loader. |
| `cloudFiles.format` | Specifies the incoming file format. |
| `.schema()` | Defines the expected schema. |
| `.load()` | Specifies the input directory. |

The streaming DataFrame allows the pipeline to process incoming files incrementally.

## 4.4 Streaming Table vs Materialized View

| Streaming Table | Materialized View |
|---|---|
| Designed for incremental ingestion and streaming transformations. | Designed to maintain the results of a query. |
| Commonly used for ingestion and streaming transformations. | Commonly used for aggregations, joins and analytical results. |
| Uses streaming semantics. | Uses batch query semantics in pipeline definitions. |
| Processes new records incrementally, subject to streaming semantics. | Can incrementally refresh eligible query results or perform a full refresh. |

**Important:** A Streaming Table does not automatically mean real-time processing.

- A triggered pipeline processes data when an update is started.
- A continuous pipeline keeps processing as new data arrives.

---

# 5. Change Data Capture (CDC)

## 5.1 What is CDC?

CDC stands for **Change Data Capture**.

It is a technique used to identify and capture changes happening in a source database.

Instead of repeatedly copying an entire source table, CDC captures changes such as:

- INSERT
- UPDATE
- DELETE

## 5.2 Understand CDC with an Example

Suppose your source database contains this customer table.

### Initial Source Data

| customer_id | name | city |
|---|---|---|
| 1 | John | Mumbai |
| 2 | Alice | Pune |
| 3 | Bob | Delhi |

Now, three things happen:

1. A new customer is inserted.
2. Alice changes her city.
3. Bob deletes his account.

### Updated Source Data

| customer_id | name | city |
|---|---|---|
| 1 | John | Mumbai |
| 2 | Alice | Bangalore |
| 4 | Sam | Hyderabad |

If you only read the latest source table, you see the final state. You do not necessarily know what changed.

CDC captures the individual operations.

### CDC Feed

| Operation | customer_id | name | city |
|---|---|---|---|
| INSERT | 4 | Sam | Hyderabad |
| UPDATE | 2 | Alice | Bangalore |
| DELETE | 3 | Bob | Delhi |

A CDC feed generally contains:

| Field | Meaning |
|---|---|
| Operation | INSERT, UPDATE or DELETE |
| Primary Key | Identifies the affected record |
| Sequence Number | Helps determine the order of changes |
| Timestamp | Indicates when the change occurred |
| Record Data | Contains the relevant record values |

## 5.3 Why Do We Need CDC?

Imagine a customer table containing 10 million records.

Every hour, only 100 customers are updated.

Without CDC, you might repeatedly read all 10 million records to identify changes.

With CDC, you can process the 100 changed records instead.

This can reduce unnecessary processing.

## 5.4 Types of CDC Approaches

### A. Log-Based CDC

Captures changes from database transaction logs.

Examples:

- MySQL Binary Logs.
- PostgreSQL Write-Ahead Logs (WAL).
- SQL Server transaction logs.

The database records its changes in transaction logs, and CDC tools read those logs to identify changes.

### B. Trigger-Based CDC

Database triggers capture inserts, updates and deletes into a separate change table.

### C. Timestamp-Based Incremental Extraction

Uses a column such as `updated_at` to identify records modified since the previous extraction.

Example:

```sql
SELECT *
FROM customers
WHERE updated_at > '2026-11-01 00:00:00';
```

This is a common incremental extraction technique, but it is not necessarily a complete CDC feed because it may not capture deletes.

### D. Snapshot-Based CDC

Compares successive snapshots of a source table to identify:

- New records.
- Updated records.
- Deleted records.

## 5.5 CDC in Databricks

Lakeflow Declarative Pipelines provides:

1. `AUTO CDC`
2. `AUTO CDC FROM SNAPSHOT`

| Feature | Purpose |
|---|---|
| AUTO CDC | Processes an incoming change feed. |
| AUTO CDC FROM SNAPSHOT | Identifies changes by comparing successive snapshots. |

These APIs can automatically manage SCD Type 1 and SCD Type 2 processing.

## 5.6 Implementing CDC in Databricks

Example using the newer `dp` API:

```python
from pyspark import pipelines as dp

dp.create_streaming_table("customers_silver")

dp.create_auto_cdc_flow(
    target="customers_silver",
    source="customers_cdc",
    keys=["customer_id"],
    sequence_by="sequence_num",
    stored_as_scd_type=2
)
```

### Explanation

| Parameter | Meaning |
|---|---|
| `target` | Target table where changes are applied. |
| `source` | Incoming CDC records. |
| `keys` | Identifies each customer record. |
| `sequence_by` | Determines the order in which changes are applied. |
| `stored_as_scd_type` | Defines the SCD processing strategy. |

**Important:** This example assumes that the source contains a valid CDC feed with the required change metadata and sequence information.

---

# 6. Slowly Changing Dimensions (SCD)

## 6.1 What is SCD?

SCD stands for **Slowly Changing Dimension**.

It is a data warehousing technique used to manage changes in dimension data over time.

Let's say your company maintains a customer table.

### Initial Data

| customer_id | name | city |
|---|---|---|
| 101 | John | Mumbai |

After two months, John moves to Bangalore.

### Updated Data

| customer_id | name | city |
|---|---|---|
| 101 | John | Bangalore |

The question is:

Should we overwrite John's old city, or should we preserve his previous city as well?

Different SCD types solve this problem differently.

## 6.2 SCD Type 1 – Overwrite Old Data

In SCD Type 1, you directly update the existing record.

### Before

| customer_id | name | city |
|---|---|---|
| 101 | John | Mumbai |

### After

| customer_id | name | city |
|---|---|---|
| 101 | John | Bangalore |

The old value `Mumbai` is lost.

**Use when historical tracking is unnecessary.**

Examples:

- Correcting spelling mistakes.
- Fixing incorrect customer information.
- Updating records where only the latest value matters.

## 6.3 SCD Type 2 – Maintain Historical Records

In SCD Type 2, instead of overwriting the existing record, you create a new version of the record.

### Before

| customer_id | name | city | start_date | end_date | is_current |
|---|---|---|---|---|---|
| 101 | John | Mumbai | Jan 1 | NULL | TRUE |

After John moves to Bangalore:

### After

| customer_id | name | city | start_date | end_date | is_current |
|---|---|---|---|---|---|
| 101 | John | Mumbai | Jan 1 | Mar 1 | FALSE |
| 101 | John | Bangalore | Mar 1 | NULL | TRUE |

Now you have both records.

You can answer:

- Where did John live previously?
- When did he move?
- What is his current city?

**Use when historical tracking is required.**

## 6.4 SCD Type 3 – Maintain Limited History

SCD Type 3 stores the current value and a previous value in separate columns.

Example:

| customer_id | name | current_city | previous_city |
|---|---|---|---|
| 101 | John | Bangalore | Mumbai |

It does not maintain unlimited history like Type 2.

## 6.5 SCD Comparison

| Type | What Happens? | History |
|---|---|---|
| Type 1 | Overwrites existing record. | Not retained. |
| Type 2 | Creates a new version. | Historical versions are retained. |
| Type 3 | Stores previous value in another column. | Limited history. |

## 6.6 CDC vs SCD

These concepts are related, but they are not the same.

| CDC | SCD |
|---|---|
| Captures changes from source data. | Defines how changes are stored in the target. |
| Identifies inserts, updates and deletes. | Manages historical versions of records. |
| Focuses on detecting changes. | Focuses on maintaining data history. |
| Example: John changed his city. | Example: Keep both Mumbai and Bangalore records. |

**Easy way to remember:**

- CDC tells you **WHAT changed**.
- SCD tells you **HOW to maintain that change** in your data warehouse.

## 6.7 How CDC and SCD Work Together

Suppose John changes his city from Mumbai to Bangalore.

```text
             SOURCE DATABASE
              John: Mumbai
                    |
                    v
                 CDC FEED
           Detects UPDATE operation
                    |
                    v
            Mumbai → Bangalore
                    |
             ┌──────┴──────┐
             |             |
             v             v
          SCD TYPE 1    SCD TYPE 2
             |             |
             v             v
       Overwrite city   Preserve old row
       with Bangalore  and insert new row
             |             |
             v             v
        Bangalore     Mumbai + Bangalore
```

---

# 7. Lakeflow Jobs

## 7.1 What is a Job in Databricks?

A Job is used to automate, schedule and coordinate different tasks in Databricks.

Think about a company that needs to perform the following operations every day:

1. Ingest customer data.
2. Run a pipeline to clean customer data.
3. Execute a SQL query to calculate revenue.
4. Run a notebook to validate the results.
5. Send a notification when the workflow completes or fails.

You do not want someone manually executing all five operations.

You create a Job that executes them in the required order.

## 7.2 Understanding Job, Task and Trigger

These are three fundamental concepts.

### A. Job

The complete workflow.

Example:

```text
Daily_Customer_ETL_Job
```

### B. Task

An individual operation inside a Job.

Example:

| Task | Operation |
|---|---|
| Task 1 | Run ingestion notebook. |
| Task 2 | Run cleaning pipeline. |
| Task 3 | Execute SQL query. |
| Task 4 | Run validation notebook. |

### C. Trigger

Determines when the Job starts.

Examples:

- Every day at 2 AM.
- Every Monday.
- When new files arrive.
- When a monitored table is updated.
- Manually when a user clicks Run Now.

## 7.3 Visualize a Job

```text
       DAILY CUSTOMER ETL JOB
       Trigger: Every day at 2 AM
                    |
                    v
          ┌──────────────────┐
          │ Task 1           │
          │ Data Ingestion   │
          │ Notebook Task    │
          └──────────────────┘
                    |
                    v
          ┌──────────────────┐
          │ Task 2           │
          │ Data Cleaning    │
          │ Pipeline Task    │
          └──────────────────┘
                    |
                    v
          ┌──────────────────┐
          │ Task 3           │
          │ Revenue           │
          │ SQL Task          │
          └──────────────────┘
                    |
                    v
          ┌──────────────────┐
          │ Task 4           │
          │ Data Validation  │
          │ Notebook Task    │
          └──────────────────┘
                    |
                    v
                JOB DONE
```

## 7.4 Important Job Concepts

| Concept | Meaning |
|---|---|
| Task Dependency | One task waits for another task. |
| Parallel Execution | Independent tasks can execute simultaneously. |
| Sequential Execution | Tasks execute one after another. |
| Retry | Automatically tries a failed task again. |
| Repair Run | Reruns failed tasks and necessary downstream tasks in an existing Job run. |
| Timeout | Maximum permitted execution duration. |
| Concurrency | Controls how many runs can execute simultaneously. |
| Parameters | Passes input values to tasks. |
| Notifications | Sends alerts for failures or configured events. |
| Job DAG | Represents task execution dependencies. |

## 7.5 Types of Job Tasks

Databricks Jobs can execute different types of workloads.

| Task Type | Purpose |
|---|---|
| Notebook Task | Executes a Databricks notebook. |
| Python Script Task | Executes a Python script. |
| Python Wheel Task | Executes a packaged Python application. |
| SQL Task | Executes SQL queries. |
| Pipeline Task | Executes an existing Lakeflow Pipeline. |
| Run Job Task | Executes another Job. |
| If/Else Task | Adds conditional execution. |
| For Each Task | Iterates over a collection of inputs. |
| JAR Task | Executes a Java or Scala application. |
| Spark Submit Task | Submits a Spark application. |

**Important:** A Pipeline Task allows you to execute an existing Lakeflow Pipeline as part of a Job.

## 7.6 Job vs Pipeline

| Pipeline | Job |
|---|---|
| Builds and maintains datasets. | Orchestrates workloads. |
| Defines data transformations. | Defines task execution. |
| Manages dependencies between datasets. | Manages dependencies between tasks. |
| Supports data quality expectations. | Supports retries, scheduling and notifications. |
| Example: Bronze → Silver → Gold. | Example: Ingestion → Pipeline → Validation. |

**Remember:**

A Pipeline is responsible for processing data.

A Job is responsible for coordinating when and how that processing happens.

---

# 8. Flows in Lakeflow Pipelines

## 8.1 What is a Flow?

A Flow describes how data moves from a source into a target dataset within a pipeline.

For example:

```text
Source Customer Data
         |
         v
Customer Streaming Table
```

You can have multiple flows feeding data into pipeline datasets.

Two important types are:

## 8.2 Append Flow

An Append Flow adds incoming records to a target streaming table.

Example:

- Today's orders are received.
- New orders are added to the target.
- Existing records are not intentionally replaced just because a new record arrives.

Useful for append-oriented event and transaction data.

## 8.3 CDC Flow

A CDC Flow processes inserts, updates and deletes from a change feed and applies those changes to the target.

Example:

- Customer 101 is inserted.
- Customer 102 is updated.
- Customer 103 is deleted.

Databricks provides `AUTO CDC` for this use case.

**Remember:**

- A Flow describes a data movement and processing path.
- A Job describes orchestration of tasks.

---

# 9. Materialized Views

## 9.1 What is a Materialized View?

A Materialized View stores the result of a query and maintains it as the underlying source data changes.

Suppose your orders table contains:

| order_id | region | amount |
|---|---|---:|
| 1 | Mumbai | 500 |
| 2 | Mumbai | 300 |
| 3 | Pune | 700 |
| 4 | Pune | 200 |

You want to calculate revenue by region.

## 9.2 Creating a Materialized View

```sql
CREATE OR REFRESH MATERIALIZED VIEW revenue_by_region
AS
SELECT
    region,
    SUM(amount) AS total_revenue
FROM orders
GROUP BY region;
```

### Output

| region | total_revenue |
|---|---:|
| Mumbai | 800 |
| Pune | 900 |

The result is stored as a Materialized View.

When the source data changes and the view is refreshed, Databricks maintains the result.

## 9.3 Materialized View vs Normal View

| Normal View | Materialized View |
|---|---|
| Stores a query definition. | Stores precomputed query results. |
| Query is evaluated when accessed. | Results are maintained through refreshes. |
| Does not independently store the complete query result. | Stores the result data. |

## 9.4 When Should You Use Each?

| Dataset Type | Use Case |
|---|---|
| Streaming Table | Incremental ingestion and streaming transformations. |
| Materialized View | Aggregations, joins, transformations and analytical results. |
| Normal View | Reusable query logic without maintaining a separately stored result. |

---

# 10. Incremental Processing

## 10.1 What is Incremental Processing?

Incremental Processing means processing only newly arrived or changed data instead of repeatedly processing the entire dataset, when the processing mechanism supports it.

Imagine your source contains 10 million records.

Every day, only 5,000 new records arrive.

There are two ways to process them.

## 10.2 Full Processing

Read and process all 10 million records again.

```text
Day 1: 10 million records
Day 2: 10 million records
Day 3: 10 million records
```

This can result in unnecessary processing.

## 10.3 Incremental Processing

Process only newly arrived or changed records when supported.

```text
Day 1: Initial 10 million records
Day 2: 5,000 new records
Day 3: 3,000 new records
```

This reduces unnecessary computation.

## 10.4 Full Refresh vs Incremental Refresh

| Full Refresh | Incremental Refresh |
|---|---|
| Recomputes the dataset from the source. | Processes eligible changes. |
| Can be more expensive for large datasets. | Can be more efficient. |
| Useful for rebuilding data. | Useful for routine updates. |

### Important Distinction

- Streaming Tables use streaming ingestion semantics.
- Materialized Views may use incremental refresh when eligible.
- CDC processes source changes.
- These are related concepts, but they are not interchangeable.

---

# 11. DAGs

## 11.1 What is a DAG?

DAG stands for **Directed Acyclic Graph**.

It represents dependencies and execution order between operations.

You will encounter DAGs in both Pipelines and Jobs.

However, their purposes are different.

## 11.2 Pipeline DAG

A Pipeline DAG represents dependencies between datasets.

Example:

```text
      Source Table
           |
           v
      Bronze Table
           |
           v
      Silver Table
           |
           v
    Gold Materialized View
```

Databricks determines the order in which datasets need to be processed based on their dependencies.

## 11.3 Job DAG

A Job DAG represents dependencies between tasks.

Example:

```text
      Task 1: Ingestion
             |
             v
      Task 2: Cleaning
             |
             v
      Task 3: Validation
             |
             v
      Task 4: Reporting
```

You configure these dependencies in the Job.

## 11.4 Why is it Called Acyclic?

Because dependencies cannot form a circular loop.

For example:

```text
A → B → C → A
```

This is not a valid DAG because it creates a circular dependency.

## 11.5 Pipeline DAG vs Job DAG

| Pipeline DAG | Job DAG |
|---|---|
| Represents dataset dependencies. | Represents task dependencies. |
| Defines the order of data processing. | Defines the order of workload execution. |
| Managed by the Pipeline framework. | Configured through Job tasks and dependencies. |
| Example: Bronze → Silver → Gold. | Example: Ingestion → Pipeline → Reporting. |

---

# 12. How Everything Works Together

Let's put all these concepts together.

Imagine an e-commerce company wants to maintain customer information and generate daily revenue reports.

## Step 1: Data Ingestion

The company has a PostgreSQL database containing customer information.

Databricks ingests the data using a suitable ingestion mechanism.

This is called **Data Ingestion**.

## Step 2: Streaming Table

The incoming customer records are ingested into a Bronze Streaming Table.

```text
customers_bronze
```

This stores the incoming records.

## Step 3: CDC

Suppose a customer changes their address.

The source database generates a CDC event.

```text
Customer ID: 101

Old City: Mumbai
New City: Bangalore

Operation: UPDATE
```

CDC captures this change.

## Step 4: SCD

Now you decide how to maintain the updated customer record.

- **SCD Type 1:** Replace Mumbai with Bangalore.
- **SCD Type 2:** Preserve Mumbai as historical data and create a new Bangalore record.

## Step 5: Materialized Views

You create a Silver Materialized View containing cleaned customer records.

Then you create a Gold Materialized View containing aggregated sales information.

## Step 6: Pipeline

You define the datasets, transformations and dependencies inside a Lakeflow Declarative Pipeline.

Databricks manages the pipeline execution.

## Step 7: Job

You create a Lakeflow Job that:

1. Executes the ingestion process.
2. Runs the pipeline.
3. Performs additional validation.
4. Executes the reporting workload.
5. Sends notifications if something fails.

You configure it to run every day at 2 AM.

## 12.1 Complete Databricks Architecture

```text
                  SOURCE SYSTEMS
             PostgreSQL | APIs | CSV | Kafka
                          |
                          v
                     INGESTION
             Auto Loader | Lakeflow Connect
                          |
                          v
          ┌───────────────────────────────┐
          │ LAKEFLOW DECLARATIVE PIPELINE │
          └───────────────────────────────┘
                          |
                          v
                  BRONZE LAYER
                   Raw Data
                Streaming Tables
                          |
                          v
                  SILVER LAYER
              Cleaned and Validated Data
              Streaming Tables / Views
                 CDC and SCD Logic
                          |
                          v
                   GOLD LAYER
              Business-Ready Datasets
                Materialized Views
                          |
                          v
               ANALYTICS & REPORTING
              Databricks SQL / Dashboards
                          |
                          v
                    BUSINESS USERS


                   LAKEFLOW JOBS
          ┌───────────────────────────────┐
          │                               │
          │  Ingestion                    │
          │      |                        │
          │  Pipeline Execution           │
          │      |                        │
          │  Validation                   │
          │      |                        │
          │  Reporting                    │
          │      |                        │
          │  Notifications                │
          │                               │
          └───────────────────────────────┘
```

This is a typical architecture, not a mandatory sequence.

For example:

- CDC can be ingested directly into a Bronze Streaming Table.
- A Job is not required to run every Pipeline.
- Materialized Views and Streaming Tables serve different purposes.
- The actual architecture depends on the source, processing requirements and output.

---

# 13. Quick Revision

This is the summary table to keep for Databricks revision.

| Concept | Simple Meaning |
|---|---|
| Data Ingestion | Bringing data from sources into Databricks. |
| ETL | Extract, Transform and Load. |
| Pipeline | Defines and processes data transformations. |
| Lakeflow Declarative Pipelines | Databricks framework for building and maintaining data pipelines. |
| Streaming Table | Dataset designed for incremental ingestion and streaming processing. |
| Materialized View | Stores and maintains the results of a query. |
| Normal View | Stores query logic rather than a separately maintained result. |
| Auto Loader | Incrementally discovers and ingests files. |
| CDC | Captures inserts, updates and deletes from source data. |
| SCD | Defines how changes to dimension records are maintained over time. |
| SCD Type 1 | Overwrites old values. |
| SCD Type 2 | Maintains historical versions. |
| SCD Type 3 | Maintains limited previous-value history. |
| Flow | Defines a data movement and processing path in a Pipeline. |
| Incremental Processing | Processes eligible new or changed data. |
| Full Refresh | Recomputes a dataset from its source. |
| DAG | Represents dependencies and execution order. |
| Job | Automates and orchestrates workloads. |
| Task | Individual operation inside a Job. |
| Trigger | Determines when a Job starts. |
| Retry | Attempts a failed task again. |
| Repair Run | Reruns failed tasks in an existing Job run. |
| Medallion Architecture | Organizes data into Bronze, Silver and Gold layers. |

---

# 14. Knowledge Check

Try answering these questions to check whether you understand the concepts.

## Question 1

A company wants to bring data from PostgreSQL into Databricks. What is this process called?

- [ ] CDC
- [ ] Data Ingestion
- [ ] SCD
- [ ] Orchestration

## Question 2

You want to process newly arriving files incrementally. Which Databricks feature is commonly used?

- [ ] Auto Loader
- [ ] Materialized View
- [ ] SCD Type 2
- [ ] Job Repair

## Question 3

You want to preserve both the old and new addresses of a customer. Which SCD type do you use?

- [ ] SCD Type 1
- [ ] SCD Type 2
- [ ] SCD Type 3
- [ ] CDC Only

## Question 4

What identifies INSERT, UPDATE and DELETE changes from a source?

- [ ] DAG
- [ ] Materialized View
- [ ] CDC
- [ ] Trigger

## Question 5

What is responsible for orchestrating multiple notebooks and pipelines?

- [ ] Streaming Table
- [ ] Lakeflow Job
- [ ] Auto Loader
- [ ] SCD

## Question 6

What does a Pipeline DAG represent?

- [ ] Job schedules
- [ ] Dataset dependencies
- [ ] User permissions
- [ ] Cluster configuration

## Question 7

Which dataset type is designed for incremental ingestion using streaming semantics?

- [ ] Normal View
- [ ] Streaming Table
- [ ] Static SQL Query
- [ ] SCD Type 3

## Question 8

What is the difference between CDC and SCD?

- [ ] They are identical.
- [ ] CDC captures changes; SCD manages how changes are stored.
- [ ] SCD captures files; CDC schedules jobs.
- [ ] CDC is only for streaming.

---

## Answer Key

| Question | Correct Answer |
|---|---|
| 1 | Data Ingestion |
| 2 | Auto Loader |
| 3 | SCD Type 2 |
| 4 | CDC |
| 5 | Lakeflow Job |
| 6 | Dataset Dependencies |
| 7 | Streaming Table |
| 8 | CDC captures changes; SCD manages how changes are stored. |

---

# Recommended Next Steps

After understanding these concepts, the next step is to implement them practically.

Recommended learning sequence:

1. Build a Bronze → Silver → Gold Lakeflow Pipeline.
2. Ingest files using Auto Loader.
3. Create Streaming Tables and Materialized Views.
4. Implement CDC using `AUTO CDC`.
5. Implement SCD Type 1 and SCD Type 2.
6. Create a Lakeflow Job with multiple tasks.
7. Configure task dependencies, retries and schedules.
8. Understand pipeline and job DAGs.
9. Deploy the pipeline and Job using Databricks Asset Bundles.

---

# Official Documentation

- [Lakeflow Declarative Pipelines – AUTO CDC](https://docs.databricks.com/aws/en/ldp/cdc)
- [Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs)
- [Standalone Pipelines and Lakeflow Pipelines](https://docs.databricks.com/aws/en/ldp/concepts/standalone-pipelines)
- [Job Triggers and Scheduling](https://docs.databricks.com/aws/en/jobs/triggers)

---

**End of Notes**
