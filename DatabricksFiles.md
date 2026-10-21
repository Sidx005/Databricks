# Databricks Files

## 1. What are Databricks Files?

The Databricks Data Intelligence Platform helps data teams across an organization work together and turn data ideas into real solutions by using shared and secure data assets and tools.

Databricks workspace files allow developers to store and use supporting files during development, such as:

- `.txt` — Text files
- `.py` — Python files and modules
- `.json` — JSON data and mappings
- `.csv` — Tabular data
- `.sql` — SQL scripts

These files can support notebook development, reusable code, and small static datasets.

## 2. Reading a Text File

Python's built-in `open()` function allows us to read file contents.

```python
path = "/Workspace/Users/desaisiddharth1246@gmail.com/db_files_folder/file.txt"

with open(path, "r") as f:
    print(f.read())
```

- `"r"` — Opens the file in read mode.
- `f.read()` — Reads the entire file.
- `with` — Automatically closes the file after execution.

## 3. Writing to a File

Use `"w"` mode to create a file or overwrite its existing content.

```python
path = "/Workspace/Users/desaisiddharth1246@gmail.com/db_files_folder/file.txt"

with open(path, "w") as f:
    f.write("Hello Siddharth")
```

To write and then read the content in the same block, use `"w+"`.

```python
with open(path, "w+") as f:
    f.write("Hello Siddharth")
    f.seek(0)
    print(f.read())
```

**Output:**
```text
Hello Siddharth
```

`f.seek(0)` moves the file cursor to the beginning so the content can be read.

**Important:** `"w"` and `"w+"` overwrite the existing file contents.

## 4. Appending to a File

Use `"a"` mode to add content at the end of a file without overwriting the existing content.

```python
with open(path, "a") as f:
    f.write("\nThis is my file")
```

To append and then read the complete file, use `"a+"`.

```python
with open(path, "a+") as f:
    f.write("\nThis is my file")
    f.seek(0)
    print(f.read())
```

**Important:** `"a+"` preserves existing content and appends new content. Each execution adds the text again.

## 5. Python File Modes

| Mode | Purpose |
|---|---|
| `"r"` | Read an existing file |
| `"w"` | Write; create or overwrite |
| `"a"` | Append; create if missing |
| `"r+"` | Read and write without truncating |
| `"w+"` | Read and write; overwrite existing content |
| `"a+"` | Read and append |

## 6. Reading JSON Files Using Pandas

JSON files are useful for storing structured data, configuration values, and static mappings.

Example `mappings.json`:

```json
[
    {"id": 1, "name": "ABC"},
    {"id": 2, "name": "DDD"},
    {"id": 3, "name": "EEE"}
]
```

Read the JSON file using Pandas:

```python
import pandas as pd

df = pd.read_json(
    "/Workspace/Users/desaisiddharth1246@gmail.com/db_files_folder/mappings.json"
)

print(df.head())
```

- `pd.read_json()` reads JSON data into a Pandas DataFrame.
- `df.head()` displays the first five rows by default.
- Pandas is convenient for small, static datasets.

## 7. Converting a Pandas DataFrame to PySpark

Databricks data engineering often uses PySpark to process larger datasets.

You can convert a Pandas DataFrame into a PySpark DataFrame:

```python
df_pyspark = spark.createDataFrame(df)

display(df_pyspark)
```

**Workflow:**

`mappings.json → Pandas DataFrame → PySpark DataFrame`

Note that Databricks can also read JSON directly with Spark:

```python
df_pyspark = spark.read.json(
    "/path/to/mappings.json"
)

display(df_pyspark)
```

Use a path accessible to Spark for the direct-read approach.

## 8. Workspace Files vs. Data Storage

Workspace files are useful for development, reusable scripts, and small supporting datasets. They are not a replacement for production data storage.

| Workspace files | Volumes / cloud storage |
|---|---|
| Supporting code and development files | Data assets and datasets |
| Small static mappings and scripts | Larger datasets and pipeline inputs |
| Accessed using workspace file paths where supported | Accessed through supported storage paths and APIs |

**Security note:** Never store passwords, API keys, or storage access keys directly in Python files. Use Databricks Secrets or Azure Key Vault for sensitive credentials.

## Key Takeaways

1. Use `open()` to read, write, and append to text files.
2. Use `f.seek(0)` to move the cursor before reading.
3. Use Python files as modules to reuse code and variables.
4. Use Pandas to conveniently read small JSON or CSV files.
5. Use `spark.createDataFrame()` to convert a Pandas DataFrame into a PySpark DataFrame.
6. Use workspace files for development and appropriate data storage for production datasets.
