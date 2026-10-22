# Databricks Auto Loader — Schema Evolution and Rescue Data

## 1. What happens when a new file has a different schema?

Suppose Auto Loader is processing order files from a Volume. The third file, `orders_batch_3_evolved`, contains additional columns such as `discount` and `payment_method`.

- Original files contain the existing columns.
- The new file introduces additional columns.
- Auto Loader detects the schema change and updates the schema information in the schema location.
- The streaming query may encounter a schema mismatch when processing the new file.

## 2. How do we resolve the issue?

Auto Loader maintains schema information in the configured schema location. Depending on the schema evolution mode, it can detect new columns, update the schema, and either continue or stop for a restart.

When using the `addNewColumns` mode, Auto Loader updates the schema and stops the stream with an exception. Restarting the query allows it to load the updated schema and continue processing.

## 3. What is rescued data?

When Auto Loader encounters data that cannot be mapped to the expected schema, it can store the unmatched data in a special column called `_rescued_data`.

For example, the new file contains:

| order_id | amount | discount | payment_method |
| -------- | ------ | -------- | -------------- |
| 101      | 500    | 50       | UPI            |

If `discount` and `payment_method` are not included in the schema used to parse the record, those fields can be captured in `_rescued_data` instead of being discarded.

The rescued data is typically stored as a string containing JSON-formatted data.

## 4. How do we extract rescued columns?

You cannot directly access `_rescued_data.discount` if `_rescued_data` is a string. First, parse it into a structured type using `from_json()`.

```
from pyspark.sql.functions import col, from_jsonfrom pyspark.sql.types import StructType, StructField, StringTyperescue_schema = StructType([    StructField("discount", StringType(), True),    StructField("payment_method", StringType(), True)])df_parsed = df.withColumn(    "rescued_parsed",    from_json(col("_rescued_data"), rescue_schema))display(    df_parsed.select(        "*",        col("rescued_parsed.discount").alias("discount"),        col("rescued_parsed.payment_method").alias("payment_method")    ))
```

How it works:

1. Define the schema of the rescued fields.
2. Use `from_json()` to parse the string into a struct.
3. Access individual fields using dot notation, such as `rescued_parsed.discount`.

The example assumes the rescued JSON contains these fields and that the DataFrame has an `_rescued_data` column.

## 5. Important distinction

| Concept          | Purpose                                                                    |
| ---------------- | -------------------------------------------------------------------------- |
| Schema location  | Stores Auto Loader's inferred schema and schema evolution information.     |
| Schema evolution | Handles changes to the source schema, such as newly added columns.         |
| `_rescued_data`  | Captures fields or values that could not be mapped to the expected schema. |
| `from_json()`    | Converts JSON-formatted strings into structured data that can be queried.  |

Remember: Schema evolution and rescued data solve different problems. Schema evolution helps Auto Loader adapt to new columns, while rescued data preserves information that does not fit the expected schema.
