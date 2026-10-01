Here’s a cleaned-up, readable Markdown version of your notes, with consistent headings, tables, code blocks, and formatting. You can save this as `databricks_unity_catalog_notes.md`.

Databricks & Unity Catalog — Study Notes

# Databricks & Unity Catalog — Study Notes

**Beginner Notes · Hands-on Learning · October 2026**

These notes cover the concepts learned so far in the `retail_lab_learning_catalog` project: Databricks, Unity Catalog, catalogs, schemas, tables, Delta Lake, the Bronze–Silver–Gold architecture, permissions, managed vs. external tables, and Unity Catalog Volumes.

---

# 1. Databricks Fundamentals

## What is Databricks?

Databricks is a data and AI platform built around technologies such as Apache Spark. It allows you to ingest, store, transform, analyze, and govern large amounts of data.

**Example:** A retail company might:

1. Ingest raw sales data.

2. Clean duplicates and missing values.

3. Calculate revenue and sales metrics.

4. Create aggregated reports for business users.

Databricks supports these tasks through SQL, Python, Spark, Delta Lake, and other tools.

## What is Unity Catalog?

Unity Catalog is Databricks' centralized governance layer for organizing and controlling access to data and other resources.

It helps manage:

* Data organization and discovery

* Permissions and access control

* Ownership

* Tables, schemas, catalogs, and Volumes

* Auditing and governance features

**Remember:** Unity Catalog organizes your data assets and determines who can access them.

---

# 2. Catalogs, Schemas, Tables, and Volumes

Unity Catalog uses a three-level namespace for tables and Volumes.

```
Catalog
└── retail_lab_learning_catalog
    ├── Schema: bronze
    │   ├── Tables
    │   └── Volumes
    ├── Schema: silver
    │   └── Tables
    └── Schema: gold
        └── Tables
```

## A. Catalog

A catalog is the top-level container for organizing data assets in Unity Catalog.

Your catalog:

```
retail_lab_learning_catalog
```

It contains the schemas used in your learning project.

## B. Schema

A schema is a logical container inside a catalog. It groups related tables, Volumes, views, and other objects.

| Schema   | Purpose                         |
| -------- | ------------------------------- |
| `bronze` | Raw or initially ingested data  |
| `silver` | Cleaned and validated data      |
| `gold`   | Aggregated, analysis-ready data |

## C. Fully Qualified Table Name

A fully qualified name identifies a table using its catalog, schema, and table name.

**Format:**

```
catalog_name.schema_name.table_name
```

**Example:**

```
SELECT *
FROM retail_lab_learning_catalog.bronze.sales_raw;
```

This tells Databricks exactly which table you want to query.

**Remember:** Catalog → Schema → Table.

The same three-level naming pattern applies to Volumes.

---

# 3. Tables and Delta Lake

## What is a Table?

A table stores structured data in rows and columns.

For example, your Bronze table `sales_raw` contains columns such as:

| order_id | product    | region | quantity | unit_price |
| -------- | ---------- | ------ | -------- | ---------- |
| O1001    | Laptop     | South  | 1        | 75000.00   |
| O1002    | Phone      | East   | 2        | 12500.00   |
| O1003    | Headphones | West   | 2        | 3250.00    |

Tables are useful when you want to query, filter, aggregate, join, or transform structured data.

## What is Delta Lake?

Delta Lake is a storage framework that adds reliability and management features to data stored in data lakes. Databricks uses Delta Lake extensively for its tables.

Delta Lake uses Parquet data files and a transaction log that tracks changes to the table.

### Key Concepts

* **ACID transactions:** Help keep table operations consistent.

* **Schema enforcement:** Helps prevent incompatible data from being written.

* **Schema evolution:** Allows supported schema changes.

* **Time travel:** Allows querying earlier table versions when the required history is available.

* **MERGE, UPDATE, DELETE:** Support common data modification operations.

* **Transaction log:** Records table changes and metadata.

* **Parquet:** Stores the underlying columnar data.

## Delta Lake vs. Delta Table

| Term         | Meaning                                               |
| ------------ | ----------------------------------------------------- |
| Delta Lake   | The storage framework and its capabilities            |
| Delta table  | A particular table that uses the Delta Lake format    |
| Parquet      | The columnar file format commonly used for table data |
| `_delta_log` | The transaction log directory for a Delta table       |

A Delta table is not simply a single Parquet file. Its data files and transaction log work together to represent the table's current state.

## How to Identify a Delta Table

You ran:

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

---

# 4. The Bronze → Silver → Gold Architecture

This is a common data engineering pattern called the **Medallion Architecture**.

## Bronze — Raw Data

**Table:** `sales_raw`

Contains initially ingested sales records, including duplicates, missing values, and invalid quantities.

## Silver — Cleaned Data

**Table:** `sales_clean`

Duplicates are removed, NULL regions are replaced, invalid quantities are filtered, and total amounts are calculated.

## Gold — Business-Ready Data

**Table:** `sales_by_region`

Contains order counts, units sold, and revenue grouped by region.

## Your Project Tables

| Layer  | Fully Qualified Table                              | Rows |
| ------ | -------------------------------------------------- | ---- |
| Bronze | `retail_lab_learning_catalog.bronze.sales_raw`     | 10   |
| Silver | `retail_lab_learning_catalog.silver.sales_clean`   | 8    |
| Gold   | `retail_lab_learning_catalog.gold.sales_by_region` | 5    |

### Bronze: Raw Data

The raw table had 10 rows. It included a duplicate order, a NULL region, and a negative quantity.

### Silver: Cleaned Data

You used SQL to remove duplicate orders, replace NULL regions with `Unknown`, filter invalid quantities, and calculate `total_amount`.

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

### Important SQL Functions

* `ROW_NUMBER()` assigns a sequence number to rows within each order ID group.

* `PARTITION BY order_id` groups rows by order ID for deduplication.

* `COALESCE(region, 'Unknown')` substitutes a value when `region` is NULL.

* `CREATE OR REPLACE TABLE ... AS SELECT` creates or replaces a table using query results.

**Note:** The deduplication logic assumes the chosen ordering identifies which duplicate record to retain. If duplicate records can differ, use a reliable business rule to select the correct record.

### Gold: Aggregated Data

You grouped the cleaned sales by region:

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

### Why This Architecture Matters

* **Bronze:** Preserves the raw input.

* **Silver:** Provides cleaned and validated data.

* **Gold:** Prepares data for reporting, dashboards, and business analysis.

---

# 5. Managed Tables vs. External Tables

Unity Catalog distinguishes between the management of a table and the storage location of its data.

| Feature            | Managed Table                                    | External Table                                         |
| ------------------ | ------------------------------------------------ | ------------------------------------------------------ |
| Storage management | Databricks manages the table's storage lifecycle | Data storage is managed separately                     |
| Data location      | Managed storage location                         | Specified external storage location                    |
| Dropping the table | Generally removes the table data as well         | Generally removes registration metadata, not the files |
| Typical use        | Tables managed within Databricks                 | Data that already exists in external cloud storage     |

Your tables `sales_raw`, `sales_clean`, and `sales_by_region` were created as managed Delta tables.

You verified this using:

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

**Important:** An external table is not the same thing as a Volume. External tables register structured tabular data, whereas Volumes provide governed access to files.

---

# 6. Unity Catalog Permissions and GRANTS

Unity Catalog controls which users and groups can discover, use, read, or modify data objects.

Permissions can be granted at different levels, including catalogs, schemas, tables, and Volumes.

## Common Privileges

| Privilege      | General Purpose                                                                |
| -------------- | ------------------------------------------------------------------------------ |
| `BROWSE`       | Discover catalog metadata without granting access to underlying table data     |
| `USE CATALOG`  | Use a catalog                                                                  |
| `USE SCHEMA`   | Use a schema                                                                   |
| `SELECT`       | Read data from a table or view                                                 |
| `MODIFY`       | Modify data in supported tables                                                |
| `CREATE TABLE` | Create tables within a schema                                                  |
| `READ VOLUME`  | Read files from a Volume                                                       |
| `WRITE VOLUME` | Write files to a Volume                                                        |
| `MANAGE`       | Manage permissions and other supported operations, subject to applicable rules |

The exact privileges required depend on the operation, the object type, ownership, inherited permissions, and other applicable access rules.

## A. Understanding Your BROWSE Grant

You ran:

```
SHOW GRANTS ON CATALOG
retail_lab_learning_catalog;
```

Your output included:

| Principal       | Privilege | Object                        |
| --------------- | --------- | ----------------------------- |
| `account users` | `BROWSE`  | `retail_lab_learning_catalog` |

This means all account users have the listed catalog-discovery privilege.

**BROWSE is not SELECT.** Discovering a catalog or seeing metadata does not by itself authorize reading the table's data.

## B. Understanding SHOW GRANTS

You used:

```
SHOW GRANTS ON TABLE
retail_lab_learning_catalog.gold.sales_by_region;
```

This command displays the grants recorded for the table.

You initially saw no rows for that table. Later, after running a grant, you saw an explicit `SELECT` grant.

## C. Your First GRANT Statement

You ran:

```
GRANT SELECT
ON TABLE retail_lab_learning_catalog.gold.sales_by_region
TO `desaisiddharth1246@gmail.com`;
```

The output then showed:

| Principal                      | Privilege | Object                                             |
| ------------------------------ | --------- | -------------------------------------------------- |
| `desaisiddharth1246@gmail.com` | `SELECT`  | `retail_lab_learning_catalog.gold.sales_by_region` |

### What Does This Mean?

* `GRANT` assigns a privilege.

* `SELECT` permits reading the table's data, subject to applicable access rules.

* `ON TABLE` specifies the object receiving the grant.

* `TO` specifies the recipient.

The backticks around your email allow it to be written as a delimited SQL identifier.

## D. GRANT vs. REVOKE

| Command       | Purpose                                 |
| ------------- | --------------------------------------- |
| `GRANT`       | Assigns a privilege                     |
| `REVOKE`      | Removes a previously assigned privilege |
| `SHOW GRANTS` | Inspects recorded grants                |

For example, the corresponding revoke syntax is:

```
REVOKE SELECT
ON TABLE retail_lab_learning_catalog.gold.sales_by_region
FROM `desaisiddharth1246@gmail.com`;
```

Do not run the revoke just for practice unless you intend to remove that explicit grant. Since you own the table, removing this grant does not necessarily remove your effective access.

### Remember These Distinctions

* **Owner:** Has ownership-related control over an object.

* **Principal:** The user or group receiving a grant.

* **Privilege:** An allowed action.

* **Object:** The catalog, schema, table, Volume, or other resource being protected.

* **Effective access:** The access actually available after applicable grants, inheritance, ownership, and other rules are considered.

---

# 7. Unity Catalog Volumes

## What Is a Volume?

A Volume is a Unity Catalog object that provides governed access to files. It is useful for non-tabular data and files that you want to organize without registering each file as a table.

Examples include:

* CSV files

* JSON files

* Images

* PDFs

* Text files

* Other supported data and application files

## Tables vs. Volumes

| Tables                                        | Volumes                                           |
| --------------------------------------------- | ------------------------------------------------- |
| Organize structured, queryable data           | Organize files                                    |
| Usually queried using SQL or Spark DataFrames | Accessed using file paths and supported file APIs |
| Example: `sales_raw`                          | Example: `raw_files`                              |

A CSV can eventually be loaded into a table, but the CSV file itself can first reside in a Volume.

## Your Volume

You created this managed Volume:

```
CREATE VOLUME IF NOT EXISTS
retail_lab_learning_catalog.bronze.raw_files;
```

You then verified it with:

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

## Reading Files from a Volume

Once your file is available, you can read it from a Python notebook attached to compatible Databricks compute.

### A. List Files

```
display(
    dbutils.fs.ls(
        "/Volumes/retail_lab_learning_catalog/bronze/raw_files/"
    )
)
```

This lists the files inside the Volume.

### B. Read the CSV Using Spark

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

### C. Inspect the DataFrame

**Show the schema:**

```
df.printSchema()
```

Shows column names and data types.

**Display the first five rows:**

```
df.show(5, truncate=False)
```

**Count the rows:**

```
print("Total rows:", df.count())
```

You have uploaded `weather_data.csv`; reading it into Spark is the next step in your hands-on work.

## Managed vs. External Volumes

* **Managed Volume:** Databricks manages the storage lifecycle.

* **External Volume:** Files remain in a separately managed external storage location registered with Unity Catalog.

A Volume does not automatically transform files into tables. You must read and process the files separately if you want to turn their contents into structured Delta tables.

---

# 8. SQL Commands to Remember

These are the most useful commands from your hands-on learning so far.

| Task                        | Command                                         |
| --------------------------- | ----------------------------------------------- |
| Create a schema             | `CREATE SCHEMA IF NOT EXISTS catalog.schema;`   |
| List schemas                | `SHOW SCHEMAS IN catalog;`                      |
| List tables                 | `SHOW TABLES IN catalog.schema;`                |
| Create a table from a query | `CREATE TABLE ... AS SELECT ...;`               |
| Inspect table metadata      | `DESCRIBE TABLE EXTENDED catalog.schema.table;` |
| Inspect Volume metadata     | `DESCRIBE VOLUME catalog.schema.volume;`        |
| Inspect table grants        | `SHOW GRANTS ON TABLE catalog.schema.table;`    |
| Inspect schema grants       | `SHOW GRANTS ON SCHEMA catalog.schema;`         |
| Inspect catalog grants      | `SHOW GRANTS ON CATALOG catalog;`               |
| Grant table read access     | `GRANT SELECT ON TABLE ... TO ...;`             |
| Remove a grant              | `REVOKE SELECT ON TABLE ... FROM ...;`          |

Replace the placeholder names with your actual catalog, schema, table, or Volume names.

---

# 9. How All the Concepts Fit Together

```
Unity Catalog
│
├── Organization and governance
│
└── Catalog: retail_lab_learning_catalog
    │
    ├── Bronze Schema
    │   ├── sales_raw (Delta table)
    │   └── raw_files (Volume)
    │       └── weather_data.csv
    │
    ├── Silver Schema
    │   └── sales_clean (Delta table)
    │
    └── Gold Schema
        └── sales_by_region (Delta table)
```

The key distinction is:

* **Unity Catalog:** Governs your data assets.

* **Delta Lake:** Provides reliable table storage.

* **Volumes:** Provide governed file access.

* **Bronze → Silver → Gold:** Organizes your data processing workflow.

The typical flow is:

**Ingest → Clean → Aggregate → Analyze**

---

# 10. Your Learning Progress

* Catalogs and schemas

* Creating and querying tables

* Bronze–Silver–Gold workflow

* Delta table metadata

* Managed vs. external tables

* GRANT and SHOW GRANTS

* Creating a Volume

* Uploading a CSV into a Volume

* Reading the CSV with Spark

* Writing the processed data into a Delta table

## Recommended Next Step

Continue with `weather_data.csv`.

1. Read it into a Spark DataFrame.

2. Inspect its schema and contents.

3. Check for missing values and duplicates.

4. Create a Bronze Delta table from the DataFrame.

5. Apply cleaning transformations to create a Silver table.

This connects your Volume knowledge to a real data ingestion workflow and builds on the Unity Catalog concepts covered in these notes.

---

**Quick Revision**

| Concept     | Remember                            |
| ----------- | ----------------------------------- |
| Catalog     | Top-level container                 |
| Schema      | Groups related objects              |
| Table       | Structured data in rows and columns |
| Delta Lake  | Reliable table storage framework    |
| Volume      | Governed file access                |
| Bronze      | Raw data                            |
| Silver      | Cleaned data                        |
| Gold        | Aggregated, analysis-ready data     |
| GRANT       | Assigns privileges                  |
| REVOKE      | Removes privileges                  |
| SHOW GRANTS | Inspects recorded privileges        |


<img width="490" height="547" alt="image" src="https://github.com/user-attachments/assets/ce330d49-3034-4fea-9a64-df26f4495912" />
