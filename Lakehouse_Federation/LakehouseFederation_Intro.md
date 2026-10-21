# Databricks Lakehouse Federation — Notes

## 1. What is Lakehouse Federation?

Lakehouse Federation allows you to query data stored in external databases directly from Databricks without first ingesting the data into your Databricks lakehouse.

Examples of external data sources include:

- MySQL
- PostgreSQL
- SQL Server
- Azure SQL Database

### Example

Suppose your customer data is stored in a PostgreSQL database.

Normally, you might ingest that data into Databricks Bronze, then query it. With Lakehouse Federation, you can query the external database directly through Unity Catalog.

External Database

PostgreSQL / MySQL

Databricks

Unity Catalog

Federated query

The source data remains in the external database. Query results are returned to Databricks.

## 2. How does Lakehouse Federation work?

The process has four main steps:

1. Create a connection: Configure Databricks to connect to the external database using its connection details and credentials.
2. Create a foreign catalog: Register the external database's metadata in Unity Catalog.
3. Query the data: Access the external schemas and tables through the foreign catalog using SQL.
4. Receive results: Databricks executes supported query operations at the source where possible and returns the results. The source data is not first copied into the lakehouse.

## 3. What is a Foreign Catalog?

A foreign catalog is a Unity Catalog object that represents an external database's metadata, allowing you to discover and query its schemas and tables from Databricks.

For example:

```
Unity Catalog
└── foreign_postgres
    ├── public
    │   ├── customers
    │   └── orders
    └── inventory
        └── products
```

Here:

- `foreign_postgres` is the foreign catalog.
- `public` and `inventory` are schemas from the external database.
- `customers`, `orders`, and `products` are external tables.

The tables' underlying data remains in the source database.

## 4. Lakehouse Federation vs. ETL

| Feature               | Lakehouse Federation                               | ETL / ELT ingestion                                               |
| --------------------- | -------------------------------------------------- | ----------------------------------------------------------------- |
| Data location         | Remains in the source system                       | Copied into target storage                                        |
| Initial data movement | No full initial ingestion required                 | Data must be transferred                                          |
| Querying              | Query external data through Databricks             | Query ingested data                                               |
| Best for              | Exploration, ad hoc queries, selected integrations | Repeated processing and production pipelines                      |
| Source dependency     | Source must be accessible when querying            | Ingested data can be queried independently of source availability |

## 5. When should you use Lakehouse Federation?

Use it when:

- You want to explore data without building an ingestion pipeline first.
- You need ad hoc queries for a proof of concept (POC).
- You need to access data across multiple databases.
- You want to reduce unnecessary data copying and duplication.
- You need governed access to external data through Unity Catalog.

Consider ingestion instead when:

- You need repeated, large-scale transformations.
- You want to optimize analytics over large datasets.
- You need historical snapshots or independently available data.
- The external database cannot handle the query workload.
- Your production architecture benefits from Delta tables and a managed medallion pipeline.

Federation is not automatically the best choice for every production workload. The right approach depends on performance, source-system capacity, freshness, governance, and cost.

## 6. Important concepts to remember

- Connection: Stores the configuration required to connect to an external data source.
- Foreign catalog: Exposes external database metadata through Unity Catalog.
- Federated query: Queries data in an external system without first ingesting the underlying dataset.
- Data movement: Query results still travel to Databricks, but the full source dataset is not necessarily copied.
- Read-only access: Common for federated analytics, but the effective permissions depend on the connection configuration, source credentials, and supported operations.

## Interview-ready answer

Lakehouse Federation is a Databricks capability that allows users to query supported external databases through Unity Catalog without first ingesting their data into the Databricks lakehouse. It uses connections and foreign catalogs to expose external schemas and tables. It is useful for ad hoc analysis, proof of concepts, and cross-system access, while ingestion into Delta tables may be more suitable for large-scale, repeated production workloads.
