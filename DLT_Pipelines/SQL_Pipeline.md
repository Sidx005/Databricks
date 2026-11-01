This DLT pipeline (Databricks Lakeflow Declarative Pipelines) creates two materialized views:

1. `sample_users_nov_1_2334` – extracts selected user data from a source table.
2. `sample_aggregation_nov_1_2334` – performs data aggregation and data quality analysis on the extracted users.

The overall flow is:

Source Table

`samples.wanderbricks.users`

Select required columns

Materialized View 1

`sample_users_nov_1_2334`

Selected user records

Group by user type and calculate metrics

Materialized View 2

`sample_aggregation_nov_1_2334`

User counts and valid email counts

## 1. First materialized view – Data extraction

```
CREATE OR REFRESH MATERIALIZED VIEW sample_users_nov_1_2334
AS
SELECT
  user_id,
  email,
  name,
  user_type
FROM samples.wanderbricks.users;
```

### What does it do?

It reads data from the existing source table:

`samples.wanderbricks.users`

And selects only four columns:

| Column      | Purpose                     |
| ----------- | --------------------------- |
| `user_id`   | Unique identifier of a user |
| `email`     | User's email address        |
| `name`      | User's name                 |
| `user_type` | Category or type of user    |

The result is stored as a materialized view named:

`sample_users_nov_1_2334`

Why create this view?

- Select only the required columns.
- Create an intermediate dataset for downstream transformations.
- Avoid repeatedly writing the source-table selection logic.
- Provide a clean input for the aggregation step.

### Understanding the syntax

- `CREATE`: Creates the materialized view if it doesn't exist.
- `OR REFRESH`: Refreshes the existing materialized view when the pipeline runs.
- `MATERIALIZED VIEW`: Stores the result of the query as a managed dataset.
- `AS SELECT`: Defines the SQL query used to produce the data.

## 2. Second materialized view – Aggregation and data quality

```
CREATE OR REFRESH MATERIALIZED VIEW sample_aggregation_nov_1_2334
AS
SELECT
  user_type,
  COUNT(user_id) AS total_count,
  COUNT_IF(
    email RLIKE '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$'
  ) AS count_valid_emails
FROM sample_users_nov_1_2334
GROUP BY ALL;
```

This view reads from the first materialized view:

`sample_users_nov_1_2334`

It groups users by their `user_type` and calculates two metrics.

### A. `COUNT(user_id) AS total_count`

Counts the number of non-NULL `user_id` values for each user type.

Example:

| user_type | total_count |
| --------- | ----------- |
| Customer  | 100         |
| Employee  | 50          |
| Admin     | 10          |

### B. `COUNT_IF()` – Valid email count

```
COUNT_IF(
  email RLIKE '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$'
) AS count_valid_emails
```

This counts records for which the email satisfies the specified regular expression.

Let's break it down:

| Expression          | Meaning                                                   |
| ------------------- | --------------------------------------------------------- |
| `COUNT_IF()`        | Counts rows where a condition is TRUE                     |
| `RLIKE`             | Checks whether a string matches a regular expression      |
| `^`                 | Start of the string                                       |
| `[a-zA-Z0-9._%+-]+` | One or more permitted characters before `@`               |
| `@`                 | Requires an at-sign                                       |
| `[a-zA-Z0-9.-]+`    | Domain characters                                         |
| `\\.`               | Matches a literal dot                                     |
| `[a-zA-Z]{2,}`      | Requires at least two alphabetic characters after the dot |
| `$`                 | End of the string                                         |

For example:

| Email               | Result  |
| ------------------- | ------- |
| john\@gmail.com     | Valid   |
| alice\@yahoo.in     | Valid   |
| user123\@company.co | Valid   |
| john\@gmail         | Invalid |
| john.com            | Invalid |
| @gmail.com          | Invalid |

Important: This is a basic email-format validation rule. It does not verify whether the email address actually exists or can receive emails.

## 3. Understanding `GROUP BY ALL`

```
GROUP BY ALL;
```

This is an important SQL concept.

Normally, you would write:

```
GROUP BY user_type;
```

But `GROUP BY ALL` automatically groups by all non-aggregated columns in the `SELECT` statement.

In your query:

```
SELECT
  user_type,
  COUNT(user_id),
  COUNT_IF(...)
```

- `user_type` is a normal column.
- `COUNT(user_id)` is an aggregate.
- `COUNT_IF(...)` is an aggregate.

Therefore, `GROUP BY ALL` automatically groups the records by `user_type`.

For example, suppose the input data looks like this:

| user_id | email              | user_type |
| ------- | ------------------ | --------- |
| 1       | john\@gmail.com    | Customer  |
| 2       | invalid-email      | Customer  |
| 3       | alice\@yahoo.in    | Customer  |
| 4       | admin\@company.com | Admin     |
| 5       | wrong-email        | Admin     |

The aggregation produces:

| user_type | total_count | count_valid_emails |
| --------- | ----------- | ------------------ |
| Customer  | 3           | 2                  |
| Admin     | 2           | 1                  |

So instead of getting individual user records, you get aggregated metrics for each user category.

## 4. How DLT executes this pipeline

One important concept: DLT does not necessarily execute SQL statements in the order in which they appear in your file.

Your aggregation view is written first, and the users view is written second.

However, Databricks identifies the dependency:

- `sample_aggregation_nov_1_2334` depends on `sample_users_nov_1_2334`.
- `sample_users_nov_1_2334` depends on `samples.wanderbricks.users`.

Lakeflow Declarative Pipelines automatically identifies these dependencies and builds a directed acyclic graph (DAG).

Pipeline execution graph

Source

Existing table

`samples.wanderbricks.users`

Transformation 1

Materialized View

`sample_users_nov_1_2334`

Transformation 2

Materialized View

`sample_aggregation_nov_1_2334`

When you run the pipeline:

1. Databricks identifies the source and transformations.
2. It determines the dependency order.
3. It processes the source data.
4. It creates or refreshes the first materialized view.
5. It creates or refreshes the aggregation view.
6. It makes the resulting datasets available for querying.

This is the declarative approach: you define what datasets should look like, and Databricks determines how to build and maintain them.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.databricks.com\&sz=32)

Databricks on AWS+2



## 5. Does it process the entire source table every time?

Not necessarily.

Materialized views support two refresh approaches:

| Refresh type        | Explanation                                                                     |
| ------------------- | ------------------------------------------------------------------------------- |
| Incremental refresh | Processes upstream changes where supported, rather than recomputing everything. |
| Full refresh        | Recomputes the result using all available source data.                          |

Databricks automatically determines whether incremental refresh is possible based on the query plan and source capabilities.

For example:

- A new user is inserted into the source table.
- The first materialized view gets updated.
- The aggregation view reflects the new user's category and updated counts.

The pipeline may process only the relevant changes rather than the entire dataset. However, incremental processing is not guaranteed for every query.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.databricks.com\&sz=32)

Databricks on Google Cloud+2



## 6. Important concepts you should know

| Concept             | Meaning in your pipeline                                          |
| ------------------- | ----------------------------------------------------------------- |
| Source table        | Existing dataset from which data is read.                         |
| Materialized view   | Stores precomputed query results.                                 |
| Transformation      | SQL operation that changes or aggregates data.                    |
| Dependency          | One dataset depends on another dataset.                           |
| DAG                 | Represents the dependency and execution order.                    |
| Incremental refresh | Updates results using eligible upstream changes.                  |
| Full refresh        | Recomputes results from the available source data.                |
| Aggregation         | Summarizes multiple records into grouped metrics.                 |
| Data quality metric | Measures something about data quality, such as valid email count. |

### A few important observations about your SQL

1\. `COUNT(user_id)` vs `COUNT(*)`

- `COUNT(user_id)` counts only non-NULL user IDs.
- `COUNT(*)` counts all rows.

If you want the total number of user records, even when `user_id` is NULL, use `COUNT(*)`.

2\. `COUNT_IF()` does not reject bad records.

Your query only counts valid email addresses. It does not remove invalid email records from the dataset.

For example, if there are 100 users and only 85 have valid email formats:

- `total_count` = 100 (assuming all user IDs are populated).
- `count_valid_emails` = 85.

This is a data quality measurement, not a data quality enforcement rule.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.databricks.com\&sz=32)

Databricks on AWS



3\. NULL email addresses

A NULL email does not satisfy the regular expression, so it is not included in `count_valid_emails`.

4\. The pipeline does not automatically schedule itself just because you define materialized views.

You need to run or configure the pipeline update/refresh mechanism. Scheduling and triggering are separate configuration decisions.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.databricks.com\&sz=32)

Databricks on AWS+1



## 7. Viva questions based on this pipeline

Q1. Why did you create two materialized views instead of one?

To separate data extraction from aggregation. The first view prepares the required user dataset, while the second performs aggregation and calculates data quality metrics.

Q2. Why use a materialized view instead of a normal view?

A materialized view stores precomputed query results, which can improve query performance and avoid recomputing the entire transformation whenever someone queries it.

Q3. What happens if a new user is inserted into the source table?

On a subsequent pipeline update, the materialized views are refreshed to reflect the changes. Databricks may use incremental refresh if supported.

Q4. Does DLT execute statements sequentially?

No. It identifies dependencies between datasets and determines the execution order using a DAG.

Q5. What is the difference between a streaming table and a materialized view?

A streaming table is designed for incremental ingestion of streaming or append-oriented data. A materialized view is designed for query results such as transformations, joins and aggregations over existing datasets.

Q6. Does `GROUP BY ALL` group by every column in the source table?

No. It groups by all non-aggregated expressions in the SELECT clause, which in this case is only `user_type`.

Q7. Does `COUNT_IF` validate the actual existence of an email address?

No. It checks whether the email matches the specified regular expression. It cannot confirm whether the email account exists.

Q8. What happens if the first materialized view fails?

The dependent aggregation view cannot successfully update using the failed upstream result.

## 8. Official Databricks documentation

These are useful references for understanding the concepts further:

1. [CREATE MATERIALIZED VIEW – Lakeflow Pipelines](https://docs.databricks.com/aws/en/ldp/developer/ldp-sql-ref-create-materialized-view?utm_source=chatgpt.com)
2. [Develop Lakeflow Pipelines using SQL](https://docs.databricks.com/aws/en/ldp/developer/sql-dev?utm_source=chatgpt.com)
3. [Incremental refresh for materialized views](https://docs.databricks.com/gcp/en/ldp/incremental-refresh?utm_source=chatgpt.com)
4. [COUNT_IF function](https://docs.databricks.com/aws/en/sql/language-manual/functions/count_if?utm_source=chatgpt.com)
5. [Create your first Lakeflow pipeline](https://docs.databricks.com/aws/en/ldp/tutorial-get-started?utm_source=chatgpt.com)

The main takeaway: Your pipeline follows a simple ETL pattern: it extracts user data from an existing table, creates an intermediate materialized view, and then transforms that data into aggregated data quality metrics using another materialized view. Databricks manages the dependencies and refresh process.
