# Databricks & Unity Catalog — Study Notes

Beginner notes · Hands-on learning · October 2026

These notes cover everything we've learned so far in your `retail_lab_learning_catalog` project: catalogs, schemas, tables, Delta Lake, the Bronze–Silver–Gold architecture, permissions and grants, managed versus external tables, and Unity Catalog Volumes.

# 1. Databricks fundamentals

## What is Databricks?

Databricks is a data and AI platform built around technologies such as Apache Spark. It allows you to ingest, store, transform, analyze, and govern large amounts of data.

For example, a retail company might:

1. Ingest raw sales data.

2. Clean duplicates and missing values.

3. Calculate revenue and sales metrics.

4. Create aggregated reports for business users.

Databricks supports these tasks through SQL, Python, Spark, Delta Lake, and other tools.

## What is Unity Catalog?

Unity Catalog is Databricks' centralized governance layer for organizing and controlling access to data and other resources.

It helps manage:

* Data organization and discovery.

* Permissions and access control.

* Ownership.

* Tables, schemas, catalogs, and Volumes.

* Auditing and governance features.

Think of Unity Catalog as the system that organizes your data assets and determines who can access them.

# 2. Catalogs, schemas, tables, and Volumes

Unity Catalog uses a three-level namespace for tables and Volumes.

Catalog

`retail_lab_learning_catalog`

Top-level container

Schema

`bronze` · `silver` · `gold`

Groups related data objects

Tables

Structured data

Volumes

Files

## A. Catalog

A catalog is the top-level container for organizing data assets in Unity Catalog.

Your catalog:

`retail_lab_learning_catalog`

It contains the schemas used in your learning project.

## B. Schema

A schema is a logical container inside a catalog. It groups related tables, Volumes, views, and other objects.

Your schemas:

|
Schema

|

Purpose in your project

|
| --- | --- |
|

`bronze`

|

Raw or initially ingested data

|
|

`silver`

|

Cleaned and validated data

|
|

`gold`

|

Aggregated, analysis-ready data

|

## C. Fully qualified table name

A fully qualified name identifies a table using its catalog, schema, and table name.

Format:

SQL

```
catalog_name.schema_name.table_name
```

Example from your project:

SQL

```
SELECT *
FROM retail_lab_learning_catalog.bronze.sales_raw;
```

This tells Databricks exactly which table you want to query.

Remember: catalog → schema → table.

The same three-level naming pattern applies to Volumes.


# 3. Tables and Delta Lake

## What is a table?

A table stores structured data in rows and columns.

For example, your Bronze table `sales_raw` contains columns such as:

|
order_id

|

product

|

region

|

quantity

|

unit_price

|
| --- | --- | --- | --- | --- |
|

O1001

|

Laptop

|

South

|

1

|

75000.00

|
|

O1002

|

Phone

|

East

|

2

|

12500.00

|
|

O1003

|

Headphones

|

West

|

2

|

3250.00

|

Tables are useful when you want to query, filter, aggregate, join, or transform structured data.

## What is Delta Lake?

Delta Lake is a storage framework that adds reliability and management features to data stored in data lakes. Databricks uses Delta Lake extensively for its tables.

Delta Lake is built on Parquet files and a transaction log that tracks changes to the table.

Key concepts:

* ACID transactions: helps keep table operations consistent.

* Schema enforcement: helps prevent incompatible data from being written.

* Schema evolution: allows supported schema changes.

* Time travel: allows querying earlier table versions when the required history is available.

* MERGE, UPDATE, DELETE: supports common data modification operations.

* Transaction log: records table changes and metadata.

* Parquet: stores the underlying columnar data.

### Delta Lake vs. Delta table

These terms are related but not identical.

|
Term

|

Meaning

|
| --- | --- |
|

Delta Lake

|

The storage framework and its capabilities

|
|

Delta table

|

A particular table that uses the Delta Lake format

|
|

Parquet

|

The columnar file format commonly used for the table's data

|
|

`_delta_log`

|

The transaction log directory for a Delta table

|

A Delta table is not simply a single Parquet file. Its data files and transaction log work together to represent the table's current state.

## How to identify a Delta table

You ran:

SQL

```
DESCRIBE TABLE EXTENDED
retail_lab_learning_catalog.gold.sales_by_region;
```

Your output included:

```
Type: MANAGED
Provider: delta
Is_managed_location: true
```

The `Provider: delta` field confirms that the table uses Delta format.

Other Delta properties you observed included:

* `delta.enableDeletionVectors`

* `delta.enableRowTracking`

* `delta.parquet.compression.codec=zstd`

These are table configuration properties. You do not need to memorize every property yet.

# 4. The Bronze → Silver → Gold architecture

This is a common data engineering pattern called the Medallion Architecture.

## Bronze — Raw data

Table: `sales_raw`

Contains initially ingested sales records, including duplicates, missing values, and invalid quantities.

Clean and validate

## Silver — Cleaned data

Table: `sales_clean`

Duplicates removed, NULL regions replaced, invalid quantities filtered, and total amounts calculated.

Aggregate and summarize

## Gold — Business-ready data

Table: `sales_by_region`

Contains order counts, units sold, and revenue grouped by region.

## Your actual project tables

|
Layer

|

Fully qualified table

|

Rows

|
| --- | --- | --- |
|

Bronze

|

`retail_lab_learning_catalog.bronze.sales_raw`

|

10

|
|

Silver

|

`retail_lab_learning_catalog.silver.sales_clean`

|

8

|
|

Gold

|

`retail_lab_learning_catalog.gold.sales_by_region`

|

5

|

### Bronze: raw data

The raw table had 10 rows. It included a duplicate order, a NULL region, and a negative quantity.

### Silver: cleaned data

You used SQL to remove duplicate orders, replace NULL regions with `Unknown`, filter invalid quantities, and calculate `total_amount`.

SQL

```
CREATE OR REPLACE TABLE
retail_lab_learning_catalog.silver.sales_clean AS
WITH deduplicated AS (
    SELECT *,
        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY order_date, customer_id
        ) AS row_num
    FROM retail_lab_learning_catalog.bronze.sales_raw
)
SELECT
    order_id,
    order_date,
    customer_id,
    product,
    category,
    COALESCE(region, 'Unknown') AS region,
    quantity,
    unit_price,
    quantity * unit_price AS total_amount
FROM deduplicated
WHERE row_num = 1
  AND quantity > 0;
```

Important functions:

* `ROW_NUMBER()` assigns a sequence number to rows within each order ID group.

* `PARTITION BY order_id` groups rows by order ID for deduplication.

* `COALESCE(region, 'Unknown')` substitutes a value when `region` is NULL.

* `CREATE OR REPLACE TABLE ... AS SELECT` creates or replaces a table using query results.

### Gold: aggregated data

You grouped the cleaned sales by region:

SQL

```
CREATE OR REPLACE TABLE
retail_lab_learning_catalog.gold.sales_by_region AS
SELECT
    region,
    COUNT(*) AS order_count,
    SUM(quantity) AS units_sold,
    SUM(total_amount) AS revenue
FROM retail_lab_learning_catalog.silver.sales_clean
GROUP BY region;
```

This produced five regional summary rows.

Why this architecture matters: Bronze preserves the raw input, Silver provides trusted data, and Gold prepares data for reporting, dashboards, and business analysis.

# 5. Managed tables vs. external tables

Unity Catalog distinguishes between the management of a table and the storage location of its data.

|
Feature

|

Managed table

|

External table

|
| --- | --- | --- |
|

Storage management

|

Databricks manages the table's storage lifecycle

|

Data storage is managed separately

|
|

Data location

|

Managed storage location

|

Specified external storage location

|
|

Dropping the table

|

Generally removes the table data as well

|

Generally removes registration metadata, not the files

|
|

Typical use

|

Tables managed within Databricks

|

Data that already exists in external cloud storage

|

Your tables `sales_raw`, `sales_clean`, and `sales_by_region` were created as managed Delta tables.

You verified this using:

SQL

```
DESCRIBE TABLE EXTENDED
retail_lab_learning_catalog.gold.sales_by_region;
```

Your output included:

```
Type: MANAGED
Is_managed_location: true
```

The blank `Location` field in your displayed output does not mean that the table has no storage location.

Important: An external table is not the same thing as a Volume. External tables register structured tabular data, whereas Volumes provide governed access to files.


# 6. Unity Catalog permissions and GRANTS

Unity Catalog controls which users and groups can discover, use, read, or modify data objects.

Permissions can be granted at different levels, including catalogs, schemas, tables, and Volumes.

## Common privileges

|
Privilege

|

General purpose

|
| --- | --- |
|

`BROWSE`

|

Discover catalog metadata without granting access to the underlying table data

|
|

`USE CATALOG`

|

Use a catalog

|
|

`USE SCHEMA`

|

Use a schema

|
|

`SELECT`

|

Read data from a table or view

|
|

`MODIFY`

|

Modify data in supported tables

|
|

`CREATE TABLE`

|

Create tables within a schema

|
|

`READ VOLUME`

|

Read files from a Volume

|
|

`WRITE VOLUME`

|

Write files to a Volume

|
|

`MANAGE`

|

Manage permissions and other supported operations, subject to the object and privilege rules

|

The exact privileges required depend on the operation, the object type, ownership, inherited permissions, and other applicable access rules.

## A. Understanding your `BROWSE` grant

You ran:

SQL

```
SHOW GRANTS ON CATALOG
retail_lab_learning_catalog;
```

Your output included:

|
Principal

|

Privilege

|

Object

|
| --- | --- | --- |
|

`account users`

|

`BROWSE`

|

`retail_lab_learning_catalog`

|

This means all account users have the listed catalog-discovery privilege.

BROWSE is not SELECT. Discovering a catalog or seeing metadata does not by itself authorize reading the table's data.

## B. Understanding `SHOW GRANTS`

You used:

SQL

```
SHOW GRANTS ON TABLE
retail_lab_learning_catalog.gold.sales_by_region;
```

This command displays the grants recorded for the table.

You initially saw no rows for that table. Later, after running a grant, you saw an explicit `SELECT` grant.

## C. Your first GRANT statement

You ran:

SQL

```
GRANT SELECT
ON TABLE retail_lab_learning_catalog.gold.sales_by_region
TO `desaisiddharth1246@gmail.com`;
```

The output then showed:

|
Principal

|

Privilege

|

Object

|
| --- | --- | --- |
|

`desaisiddharth1246@gmail.com`

|

`SELECT`

|

`retail_lab_learning_catalog.gold.sales_by_region`

|

### What does this mean?

* `GRANT` assigns a privilege.

* `SELECT` permits reading the table's data, subject to applicable access rules.

* `ON TABLE` specifies the object receiving the grant.

* `TO` specifies the recipient.

The backticks around your email allow it to be written as a delimited SQL identifier.

## D. GRANT vs. REVOKE

|
Command

|

Purpose

|
| --- | --- |
|

`GRANT`

|

Assigns a privilege

|
|

`REVOKE`

|

Removes a previously assigned privilege

|
|

`SHOW GRANTS`

|

Inspects recorded grants

|

For example, the corresponding revoke syntax is:

SQL

```
REVOKE SELECT
ON TABLE retail_lab_learning_catalog.gold.sales_by_region
FROM `desaisiddharth1246@gmail.com`;
```

Do not run the revoke just for practice unless you intend to remove that explicit grant. Since you own the table, removing this grant does not necessarily remove your effective access.

### Remember these distinctions

* Owner: has ownership-related control over an object.

* Principal: the user or group receiving a grant.

* Privilege: an allowed action.

* Object: the catalog, schema, table, Volume, or other resource being protected.

* Effective access: the access actually available after applicable grants, inheritance, ownership, and other rules are considered.

# 7. Unity Catalog Volumes

## What is a Volume?

A Volume is a Unity Catalog object that provides governed access to files. It is useful for non-tabular data and files that you want to organize without registering each file as a table.

Examples include:

* CSV files

* JSON files

* Images

* PDFs

* Text files

* Other supported data and application files

### Tables vs. Volumes

|
Tables

|

Volumes

|
| --- | --- |
|

Organize structured, queryable data

|

Organize files

|
|

Usually queried using SQL or Spark DataFrames

|

Accessed using file paths and supported file APIs

|
|

Example: `sales_raw`

|

Example: `raw_files`

|

A CSV can eventually be loaded into a table, but the CSV file itself can first reside in a Volume.

## Your Volume

You created this managed Volume:

SQL

```
CREATE VOLUME IF NOT EXISTS
retail_lab_learning_catalog.bronze.raw_files;
```

You then verified it with:

SQL

```
DESCRIBE VOLUME
retail_lab_learning_catalog.bronze.raw_files;
```

The Volume is located within your existing catalog and Bronze schema.

Its standard file path is:

```
/Volumes/retail_lab_learning_catalog/bronze/raw_files/
```

You uploaded this file:

```
/Volumes/retail_lab_learning_catalog/bronze/raw_files/weather_data.csv
```

## Reading files from a Volume

Once your file is available, you can read it from a Python notebook attached to compatible Databricks compute.

### List files

Python

Run

```
display(
    dbutils.fs.ls(
        "/Volumes/retail_lab_learning_catalog/bronze/raw_files/"
    )
)
```

This lists the files inside the Volume.

### Read the CSV using Spark

Python

Run

```
file_path = (
    "/Volumes/retail_lab_learning_catalog/"
    "bronze/raw_files/weather_data.csv"
)

df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv(file_path)
)

display(df)
```

### Inspect the DataFrame

Python

Run

```
df.printSchema()
```

Shows column names and data types.

Python

Run

```
df.show(5, truncate=False)
```

Shows the first five rows.

Python

Run

```
print("Total rows:", df.count())
```

Counts the rows in the DataFrame.

You have uploaded `weather_data.csv`; reading it into Spark is the next step in your hands-on work.

## Managed vs. external Volumes

* Managed Volume: Databricks manages the storage lifecycle.

* External Volume: Files remain in a separately managed external storage location registered with Unity Catalog.

A Volume does not automatically transform files into tables. You must read and process the files separately if you want to turn their contents into structured Delta tables.


# 8. SQL commands to remember

These are the most useful commands from your hands-on learning so far.

|
Task

|

Command

|
| --- | --- |
|

Create a schema

|

`CREATE SCHEMA IF NOT EXISTS catalog.schema;`

|
|

List schemas

|

`SHOW SCHEMAS IN catalog;`

|
|

List tables

|

`SHOW TABLES IN catalog.schema;`

|
|

Create a table from a query

|

`CREATE TABLE ... AS SELECT ...;`

|
|

Inspect table metadata

|

`DESCRIBE TABLE EXTENDED catalog.schema.table;`

|
|

Inspect Volume metadata

|

`DESCRIBE VOLUME catalog.schema.volume;`

|
|

Inspect table grants

|

`SHOW GRANTS ON TABLE catalog.schema.table;`

|
|

Inspect schema grants

|

`SHOW GRANTS ON SCHEMA catalog.schema;`

|
|

Inspect catalog grants

|

`SHOW GRANTS ON CATALOG catalog;`

|
|

Grant table read access

|

`GRANT SELECT ON TABLE ... TO ...;`

|
|

Remove a grant

|

`REVOKE SELECT ON TABLE ... FROM ...;`

|

Replace the placeholder names with your actual catalog, schema, table, or Volume names.

# 9. How all the concepts fit together

Unity Catalog

Organization, governance and access control

Catalog → Schema

Organizes your data objects

Delta tables

Structured data

Volumes

Files

Bronze → Silver → Gold

Ingest → Clean → Aggregate

The key distinction is that Unity Catalog governs your assets, Delta Lake provides reliable table storage, Volumes provide governed file access, and the Bronze–Silver–Gold pattern organizes your data processing workflow.

## 10. Your learning progress

Catalogs and schemas

Covered

Creating and querying tables

Covered

Bronze–Silver–Gold workflow

Covered

Delta table metadata

Covered

Managed vs. external tables

Covered

GRANT and SHOW GRANTS

Covered

Creating a Volume

Covered

Uploading a CSV into a Volume

Covered

Reading the CSV with Spark

Next

Writing the processed data into a Delta table

Next

Recommended next step: Continue with `weather_data.csv`. Read it into a Spark DataFrame, inspect its schema and contents, and then create a Bronze Delta table from it. That will connect your Volume knowledge to a real ingestion workflow.


<img width="490" height="547" alt="image" src="https://github.com/user-attachments/assets/ce330d49-3034-4fea-9a64-df26f4495912" />
