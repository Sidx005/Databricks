# DLT Expectations (Data Quality Rules)

## 1. What are Expectations?

Expectations are data quality rules defined within a DLT pipeline to validate incoming data.

They ensure that the data meets certain conditions before it is written into the target table.

Examples:

- A primary key should never be NULL.
- Product price should never be negative.
- Age should be greater than or equal to 18.
- Email should follow a valid format.

Think of expectations as quality checks applied to incoming data.

## 2. How do Expectations work?

Suppose we receive the following product data:

| Product ID | Product | Price |
| ---------- | ------- | ----- |
| 1          | Shoes   | 500   |
| 2          | Comb    | 100   |
| 3          | Mobile  | -500  |
| NULL       | Laptop  | 50000 |

We define two expectations:

1. `product_id IS NOT NULL`
2. `price >= 0`

DLT evaluates every incoming record against these conditions.

- ID 1 and ID 2 pass both checks.
- ID 3 fails because its price is negative.
- The Laptop record fails because its product ID is NULL.

What happens to the failed records depends on the expectation action we choose.

## 3. Three Types of Expectations

DLT provides three main ways to handle records that violate expectations.

### A. Expect – Warn

Record is retained.

If a record violates the rule:

- DLT records the data quality violation in pipeline metrics.
- It does not discard the record.
- The record is still written into the target table.

```
@dlt.expect("valid_price", "price >= 0")@dlt.tabledef products():    return spark.readStream.table("products_bronze")
```

Suppose price = -500.

The record will still be inserted, but DLT reports the expectation violation.

Use when: You want to monitor data quality without interrupting the pipeline.

### B. Expect or Drop

Invalid record is dropped.

If a record violates the rule:

- DLT identifies the violation.
- The invalid record is dropped.
- Valid records continue to the target table.

```
@dlt.expect_or_drop("valid_price", "price >= 0")@dlt.tabledef products():    return spark.readStream.table("products_bronze")
```

If price = -500, that record will not reach the target table.

Use when: Invalid records should not enter downstream tables.

### C. Expect or Fail

Pipeline update fails.

If a record violates the expectation:

- DLT raises an error.
- The pipeline update fails.
- The invalid data is not accepted in that update.

```
@dlt.expect_or_fail("valid_price", "price >= 0")@dlt.tabledef products():    return spark.readStream.table("products_bronze")
```

If price = -500, the pipeline update fails.

Use when: Data quality is critical and invalid records must never be accepted.

## 4. Comparison of all Three

| Behaviour            | Expect | Expect or Drop | Expect or Fail   |
| -------------------- | ------ | -------------- | ---------------- |
| Checks data quality  | Yes    | Yes            | Yes              |
| Keeps invalid record | Yes    | No             | No               |
| Keeps valid records  | Yes    | Yes            | Update fails     |
| Pipeline continues   | Yes    | Yes            | No, update fails |
| Records violations   | Yes    | Yes            | Yes              |

## 5. Expectations Work at the Row Level

Suppose you have 100 records:

- 90 records satisfy the condition.
- 10 records violate the condition.

Depending on the expectation:

- `expect`: All 100 records continue, but 10 violations are reported.
- `expect_or_drop`: 10 records are dropped, and 90 continue.
- `expect_or_fail`: The pipeline update fails because of the violations.

## 6. Important Points to Remember

1. Expectations are defined using SQL Boolean expressions.
2. They can validate NULL values, ranges, uniqueness conditions, and business rules.
3. Multiple expectations can be defined on the same table.
4. Expectations help monitor and enforce data quality throughout the pipeline.
5. Expectation metrics can be viewed in the pipeline event log and data quality metrics.

Important: Expectations do not automatically fix bad data. They either report, filter, or fail based on your chosen policy.

## 7. Quick Revision

| API                   | Meaning                                           |
| --------------------- | ------------------------------------------------- |
| `@dlt.expect`         | Warn me, but keep the record.                     |
| `@dlt.expect_or_drop` | Drop the bad record and continue.                 |
| `@dlt.expect_or_fail` | Fail the pipeline update if a bad record appears. |

Note: These are the classic DLT decorator APIs. The newer Lakeflow Spark Declarative Pipelines API uses the corresponding `@dp.expect`, `@dp.expect_or_drop`, and `@dp.expect_or_fail` decorators.
