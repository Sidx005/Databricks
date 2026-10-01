In SQL, both `UNION` and `UNION ALL` combine the results of two or more `SELECT` queries. The main difference is how they handle duplicate rows.

## 1. Difference between UNION and UNION ALL

| Feature                | `UNION`                                      | `UNION ALL`                              |
| ---------------------- | -------------------------------------------- | ---------------------------------------- |
| Combines query results | Yes                                          | Yes                                      |
| Removes duplicate rows | Yes                                          | No                                       |
| Performance            | Usually slower because it removes duplicates | Usually faster because it keeps all rows |
| Best used when         | You want unique results                      | You want to preserve all results         |

## 2. Example using weather data

Suppose we have two queries returning these results.

Query A

| event |
| ----- |
| Rain  |
| Sunny |

Query B

| event |
| ----- |
| Sunny |
| Snow  |

### Using UNION

SQL

```
SELECT event FROM weather_a
UNION
SELECT event FROM weather_b;
```

Output:

| event |
| ----- |
| Rain  |
| Sunny |
| Snow  |

Notice that `Sunny` appears only once because `UNION` removes duplicate rows.

### Using UNION ALL

SQL

```
SELECT event FROM weather_a
UNION ALL
SELECT event FROM weather_b;
```

Output:

| event |
| ----- |
| Rain  |
| Sunny |
| Sunny |
| Snow  |

Here, both occurrences of `Sunny` are preserved.

## 3. How this applies to our Databricks project

We used `UNION ALL` in our validation query:

SQL

```
SELECT 'Bronze' AS layer, COUNT(*) AS row_count
FROM retail_lab_learning_catalog.bronze.weather_raw

UNION ALL

SELECT 'Silver' AS layer, COUNT(*) AS row_count
FROM retail_lab_learning_catalog.silver.weather_clean

UNION ALL

SELECT 'Gold' AS layer, COUNT(*) AS row_count
FROM retail_lab_learning_catalog.gold.weather_summary;
```

The output looks like this:

| layer  | row_count |
| ------ | --------- |
| Bronze | 9         |
| Silver | 9         |
| Gold   | 5         |

Why did we use `UNION ALL`?

Each `SELECT` produces a separate summary row. We want to preserve every row from every query, without unnecessary duplicate removal.

Even if two layers had the same name and row count, `UNION ALL` would preserve both rows. `UNION` could remove one if the entire resulting rows were identical.

## 4. One important rule

Both queries must return the same number of columns, and corresponding columns must have compatible data types.

For example, this works:

SQL

```
SELECT 'Bronze' AS layer, COUNT(*) AS row_count
FROM bronze_table

UNION ALL

SELECT 'Silver' AS layer, COUNT(*) AS row_count
FROM silver_table;
```

But combining a query returning two columns with one returning three columns will cause an error.

### Easy way to remember

`UNION ALL` = Combine everything

Keeps duplicates. Usually more efficient.

`UNION` = Combine unique rows

Removes duplicate rows from the combined result.

Interview tip: If asked which is faster, the typical answer is `UNION ALL`, because it doesn't need to perform duplicate elimination.
