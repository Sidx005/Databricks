## Important PySpark DataFrame Commands

### 1. Create & Read

```python
# Existing table
df = spark.table("workspace.default.movies")

# CSV
df = spark.read.csv("path/file.csv", header=True, inferSchema=True)

# Parquet
df = spark.read.parquet("path/file.parquet")

# JSON
df = spark.read.json("path/file.json")

# From Python list
df = spark.createDataFrame(
    [(1, "A"), (2, "B")],
    ["id", "name"]
)
```

### 2. Inspect Data

```python
df.show(5)
display(df)
df.printSchema()
df.columns
df.count()
df.describe().show()
df.summary().show()
```

### 3. Select & Filter

```python
from pyspark.sql.functions import col

# Select columns
df.select("title", "imdb_rating")

# Filter rows
df.filter(col("imdb_rating") > 8)

# Multiple conditions
df.filter(
    (col("release_year") >= 2000) &
    (col("release_year") <= 2010)
)

# Between
df.filter(col("release_year").between(2000, 2010))

# Distinct
df.select("industry").distinct()
```

### 4. Add & Modify Columns

```python
from pyspark.sql.functions import lit, when

# Add column
df = df.withColumn("profit", col("revenue") - col("budget"))

# Rename
df = df.withColumnRenamed("revenue", "total_revenue")

# Drop column
df = df.drop("budget")

# Add constant
df = df.withColumn("source", lit("Movies"))

# Conditional column
df = df.withColumn(
    "category",
    when(col("imdb_rating") >= 8, "Excellent")
    .otherwise("Average")
)
```

### 5. Aggregations

```python
from pyspark.sql.functions import sum, avg, count, max, min

# Group by
df.groupBy("industry").count().show()

# Average rating by industry
df.groupBy("industry").agg(
    avg("imdb_rating").alias("avg_rating")
).show()

# Total revenue
df.agg(sum("revenue")).show()
```

### 6. Sorting & Duplicates

```python
# Sort descending
df.orderBy(col("imdb_rating").desc())

# Remove duplicates
df.dropDuplicates()

# Remove duplicates based on column
df.dropDuplicates(["title"])

# Limit rows
df.limit(10)
```

### 7. Joins

```python
# Inner join
df1.join(df2, df1.id == df2.id, "inner")

# Left join
df1.join(df2, "id", "left")
```

### 8. Null Handling

```python
# Remove null rows
df.dropna()

# Fill null values
df.fillna({"imdb_rating": 0})

# Filter nulls
df.filter(col("title").isNull())
df.filter(col("title").isNotNull())
```

### 9. Write Data

```python
# Write Parquet
df.write.mode("overwrite").parquet("path/output")

# Write CSV
df.write.mode("overwrite").option("header", True).csv("path/output")

# Write Delta table
df.write.mode("overwrite").saveAsTable("workspace.default.movies_processed")
```

### 10. Spark SQL

```python
# Create temporary view
df.createOrReplaceTempView("movies")

# Query using SQL
result = spark.sql("""
    SELECT industry, AVG(imdb_rating) AS avg_rating
    FROM movies
    GROUP BY industry
""")

display(result)
```

**Remember:** DataFrame transformations are lazy. Commands like `select()`, `filter()`, and `withColumn()` build a plan; actions like `show()`, `count()`, and `write()` execute it.
