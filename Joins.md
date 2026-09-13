## 1. Running SQL using Python

```python
df_marvel = spark.sql("""
    SELECT *
    FROM workspace.default.movies
    WHERE studio = 'Marvel Studios'
""")

df_marvel.show()
```

### Explanation

* `spark.sql()` executes a SQL query using Spark.
* `workspace.default.movies` is a fully qualified table name:

```text
catalog.schema.table
workspace.default.movies
```

* `WHERE studio = 'Marvel Studios'` filters only movies produced by Marvel Studios.
* The result is stored in a DataFrame named `df_marvel`.
* `.show()` displays the result.

Equivalent Databricks SQL:

```sql
%sql
SELECT *
FROM workspace.default.movies
WHERE studio = 'Marvel Studios';
```

The Python and `%sql` versions perform the same operation.

---

# 2. Creating a DataFrame manually

```python
data = [
    ("2017-01-01", 32.0, 6.0, "Rain"),
    ("2017-01-04", None, 9.0, "Sunny"),
    ("2017-01-05", 28.0, None, "Snow"),
    ("2017-01-06", None, 7.0, None),
    ("2017-01-07", 32.0, None, "Rain"),
    ("2017-01-08", None, None, "Sunny"),
    ("2017-01-09", None, None, None),
    ("2017-01-10", 34.1, 8.1, "Cloudy"),
    ("2017-01-11", 40.0, 12.0, "Sunny"),
]
```

This is a Python list containing tuples.

Each tuple represents one row:

```text
day, temperature, windspeed, event
```

`None` represents a SQL `NULL` value.

For example:

```python
("2017-01-04", None, 9.0, "Sunny")
```

means:

```text
day         = 2017-01-04
temperature = NULL
windspeed   = 9.0
event       = Sunny
```

---

## 3. Defining the schema

```python
schema = """
    day string,
    temperature double,
    windspeed double,
    event string
"""
```

The schema defines the column names and data types.

| Column        | Type     | Meaning                       |
| ------------- | -------- | ----------------------------- |
| `day`         | `string` | Date stored initially as text |
| `temperature` | `double` | Decimal temperature           |
| `windspeed`   | `double` | Decimal wind speed            |
| `event`       | `string` | Weather event                 |

---

## 4. Creating the DataFrame

```python
df = spark.createDataFrame(
    data=data,
    schema=schema
)
```

This converts the Python data into a Spark DataFrame.

```python
df.show()
```

Displays the rows in tabular form.

Important:

```python
None
```

becomes:

```text
NULL
```

in Spark.

---

# 5. Importing Spark functions

```python
from pyspark.sql import functions as F, types as T
```

This imports:

* `F`: Spark SQL functions such as `col`, `to_date`, `sum`, `avg`, `when`
* `T`: Spark data types such as `StringType`, `IntegerType`, `StructType`

Example:

```python
F.col("temperature")
F.to_date("day", "yyyy-MM-dd")
T.StringType()
```

The aliases are only shortcuts. You could also write:

```python
from pyspark.sql import functions
functions.col("temperature")
```

---

# 6. Converting a string into a date

```python
df = df.withColumn(
    "day",
    F.to_date("day", "yyyy-MM-dd")
)
```

Initially, `day` is a string:

```text
"2017-01-01"
```

After conversion, it becomes a Spark `date`:

```text
2017-01-01
```

### Important parts

```python
withColumn("day", ...)
```

means:

* Replace the existing `day` column if it exists.
* Otherwise, create a new column named `day`.

```python
F.to_date("day", "yyyy-MM-dd")
```

converts the string using the specified format.

You can verify the result:

```python
df.printSchema()
```

Expected schema:

```text
day: date
temperature: double
windspeed: double
event: string
```

Spark DataFrames are immutable, so this creates a new DataFrame assigned back to `df`.

---

# 7. Creating a temporary SQL view

```python
df.createOrReplaceTempView("weather")
```

This registers the DataFrame as a temporary SQL view named `weather`.

You can now query it using SQL:

```python
spark.sql("""
    SELECT day, temperature, windspeed, event
    FROM weather
    WHERE temperature IS NOT NULL
    ORDER BY day
""").show()
```

### Explanation

```sql
SELECT day, temperature, windspeed, event
```

Selects the specified columns.

```sql
FROM weather
```

Reads from the temporary view.

```sql
WHERE temperature IS NOT NULL
```

Keeps only rows where temperature has a value.

This removes rows such as:

```text
2017-01-04, NULL, 9.0, Sunny
```

It does not remove rows just because other columns contain nulls.

```sql
ORDER BY day
```

Sorts the result by date in ascending order.

### Temporary view characteristics

* It exists only during the current Spark session.
* It is not a permanent table.
* It can be queried using `spark.sql()`.
* It is useful for combining PySpark and SQL.

---

# 8. Creating customer and order data

## Customer data

```python
rows_customers = [
    (1, "Asha", "IN", True),
    (2, "Bob", "US", False),
    (3, "Chen", "CN", True),
    (4, "Diana", "US", None),
    (None, "Ghost", "UK", False),
]
```

Columns:

```text
customer_id, name, country, vip
```

The last row has a null customer ID:

```python
(None, "Ghost", "UK", False)
```

This is used to demonstrate null join behavior.

## Order data

```python
rows_orders = [
    (101, 1, 120.0, "IN"),
    (102, 1, 80.0, "IN"),
    (103, 2, 50.0, "US"),
    (104, 5, 30.0, "DE"),
    (105, 3, 200.0, "CN"),
    (106, None, 15.0, "UK"),
    (107, 3, 40.0, "CN"),
    (108, 2, 75.0, "US"),
]
```

Columns:

```text
order_id, customer_id, amount, country
```

Some orders intentionally do not have matching customers:

```text
customer_id = 5
customer_id = NULL
```

---

# 9. Defining a structured schema

```python
schema_customers = T.StructType([
    T.StructField("customer_id", T.IntegerType(), True),
    T.StructField("name", T.StringType(), True),
    T.StructField("country", T.StringType(), True),
    T.StructField("vip", T.BooleanType(), True)
])
```

## `StructType`

Represents the complete schema of a DataFrame.

## `StructField`

Represents one column.

For example:

```python
T.StructField(
    "customer_id",
    T.IntegerType(),
    True
)
```

means:

* Column name: `customer_id`
* Data type: integer
* Nullable: `True`

The third argument means the column is allowed to contain nulls.

The orders schema is similar:

```python
schema_orders = T.StructType([
    T.StructField("order_id", T.IntegerType(), True),
    T.StructField("customer_id", T.IntegerType(), True),
    T.StructField("amount", T.DoubleType(), True),
    T.StructField("country", T.StringType(), True)
])
```

---

# 10. Creating the customer and order DataFrames

```python
df_customers = spark.createDataFrame(
    rows_customers,
    schema_customers
)

df_orders = spark.createDataFrame(
    rows_orders,
    schema_orders
)
```

Display them:

```python
display(df_customers)
display(df_orders)
```

`display()` is Databricks-specific and provides a richer table and visualization interface.

`.show()` is the standard Spark method:

```python
df_customers.show()
df_orders.show()
```

---

# 11. Creating DataFrame aliases

```python
o = df_orders.alias("o")
c = df_customers.alias("c")
```

The aliases are:

```text
o = orders
c = customers
```

Aliases are useful when both DataFrames contain columns with the same name, such as:

```text
customer_id
country
```

You can refer to a specific column using:

```python
F.col("o.country")
F.col("c.country")
```

Without aliases, Spark may report an ambiguous column error.

---

# 12. Inner join

```python
df_inner = o.join(
    c,
    on="customer_id",
    how="inner"
)
```

An inner join returns only rows that have matching values in both DataFrames.

The join condition is:

```text
orders.customer_id = customers.customer_id
```

Matching customer IDs are:

```text
1, 2, 3
```

Therefore:

* Orders for customer `1` match Asha.
* Orders for customer `2` match Bob.
* Orders for customer `3` match Chen.
* Order with customer `5` has no match.
* Order with null customer ID has no match.
* Customer `4` has no order and is not included.
* Customer with null ID is not included.

### Result conceptually

| order_id | customer_id | amount | order country | name | customer country |
| -------: | ----------: | -----: | ------------- | ---- | ---------------- |
|      101 |           1 |  120.0 | IN            | Asha | IN               |
|      102 |           1 |   80.0 | IN            | Asha | IN               |
|      103 |           2 |   50.0 | US            | Bob  | US               |
|      105 |           3 |  200.0 | CN            | Chen | CN               |
|      107 |           3 |   40.0 | CN            | Chen | CN               |
|      108 |           2 |   75.0 | US            | Bob  | US               |

---

# 13. Disambiguating duplicate columns

Both DataFrames contain:

```text
country
```

So this join:

```python
o.join(c, on="customer_id", how="inner")
```

can produce two country columns.

To control the output, use `select()`:

```python
df_inner_clean = (
    o.join(c, on="customer_id", how="inner")
     .select(
         "order_id",
         "customer_id",
         "amount",
         F.col("o.country").alias("ship_country"),
         "name",
         F.col("c.country").alias("cust_country"),
         "vip"
     )
)
```

### Explanation

```python
F.col("o.country").alias("ship_country")
```

selects the country from the orders DataFrame and renames it to `ship_country`.

```python
F.col("c.country").alias("cust_country")
```

selects the country from the customers DataFrame and renames it to `cust_country`.

The output becomes clearer:

| Column         | Meaning                           |
| -------------- | --------------------------------- |
| `ship_country` | Country recorded on the order     |
| `cust_country` | Country recorded for the customer |

This avoids confusion between two columns both named `country`.

---

# 14. Left join

```python
o.join(
    c,
    on="customer_id",
    how="left"
)
```

A left join returns:

* Every row from the left DataFrame, `o`
* Matching rows from the right DataFrame, `c`
* `NULL` for right-side columns when no match exists

Since `o` is the left DataFrame, every order remains.

The unmatched orders are:

```text
order_id 104 → customer_id 5
order_id 106 → customer_id NULL
```

Their customer columns will be null.

Use a left join when you want to preserve all orders, even if customer information is missing.

---

# 15. Full outer join

```python
o.join(
    c,
    on="customer_id",
    how="full"
)
```

A full join returns:

* All matching rows
* Unmatched orders
* Unmatched customers

Unmatched orders:

```text
customer_id 5
customer_id NULL from orders
```

Unmatched customers:

```text
customer_id 4
customer_id NULL from customers
```

Therefore, the result contains every row from both DataFrames.

Other accepted names include:

```python
how="full_outer"
how="outer"
```

---

# 16. Left semi join

```python
o.join(
    c,
    on="customer_id",
    how="left_semi"
)
```

A left semi join returns only the rows from the left DataFrame that have a match in the right DataFrame.

It returns order columns only.

It does not include customer columns.

For this data, it returns orders for customer IDs:

```text
1, 2, 3
```

Conceptually:

| order_id | customer_id | amount | country |
| -------: | ----------: | -----: | ------- |
|      101 |           1 |  120.0 | IN      |
|      102 |           1 |   80.0 | IN      |
|      103 |           2 |   50.0 | US      |
|      105 |           3 |  200.0 | CN      |
|      107 |           3 |   40.0 | CN      |
|      108 |           2 |   75.0 | US      |

### Use case

Use a left semi join when you only need to check whether a matching record exists.

Example:

```text
Find orders belonging to registered customers.
```

---

# 17. Left anti join

```python
o.join(
    c,
    on="customer_id",
    how="left_anti"
)
```

A left anti join returns rows from the left DataFrame that do not have a match in the right DataFrame.

Here it returns orphan orders:

```text
order_id 104 → customer_id 5
order_id 106 → customer_id NULL
```

Conceptually:

| order_id | customer_id | amount | country |
| -------: | ----------: | -----: | ------- |
|      104 |           5 |   30.0 | DE      |
|      106 |        NULL |   15.0 | UK      |

### Use case

Left anti joins are useful for finding:

* Orphan records
* Invalid foreign keys
* Records missing from another dataset
* Data quality issues

Example:

```text
Find orders whose customer does not exist in the customer master table.
```

---

# 18. Multi-column join

```python
df_multi = o.join(
    c,
    on=["customer_id", "country"],
    how="inner"
)
```

This joins using two conditions:

```text
orders.customer_id = customers.customer_id
AND
orders.country = customers.country
```

Both columns must match.

For example:

```text
customer_id = 1
country = IN
```

matches Asha.

But if the customer ID matches and the country differs, the row does not match.

### Equivalent explicit condition

```python
df_multi = o.join(
    c,
    on=(
        (F.col("o.customer_id") == F.col("c.customer_id")) &
        (F.col("o.country") == F.col("c.country"))
    ),
    how="inner"
)
```

When using `&`, each condition must be enclosed in parentheses.

Do not use Python `and` for Spark column conditions.

Correct:

```python
(F.col("o.customer_id") == F.col("c.customer_id")) &
(F.col("o.country") == F.col("c.country"))
```

Incorrect:

```python
F.col("o.customer_id") == F.col("c.customer_id") and ...
```

---

# 19. Important join summary

| Join type   | Result                                         |
| ----------- | ---------------------------------------------- |
| `inner`     | Only matching rows from both DataFrames        |
| `left`      | All left rows plus matching right rows         |
| `right`     | All right rows plus matching left rows         |
| `full`      | All rows from both DataFrames                  |
| `left_semi` | Left rows that have a match; left columns only |
| `left_anti` | Left rows that do not have a match             |
| `cross`     | Cartesian product of both DataFrames           |

---

# 20. Important null behavior

In normal Spark SQL joins:

```text
NULL = NULL
```

is not considered true.

Therefore, these two rows do not match:

```text
orders.customer_id = NULL
customers.customer_id = NULL
```

If you specifically want nulls to match nulls, use the null-safe equality operator:

```python
df_null_safe = o.join(
    c,
    on=F.col("o.customer_id").eqNullSafe(
        F.col("c.customer_id")
    ),
    how="inner"
)
```

This treats:

```text
NULL <=> NULL
```

as true.

Use this carefully because it can match unrelated records that both have missing keys.
