# 📋 COPY Options in Snowflake

---

## 🧭 Overview

The **`COPY INTO` command** in Snowflake supports multiple **options** to fine-tune the process of **loading** or **unloading** data.  
These options control behavior like error handling, file selection, validation, overwriting, compression, and more.

> 💡 COPY options apply to both **bulk load** (`COPY INTO <table>`) and **unload** (`COPY INTO @stage`) operations.

---

## ⚙️ Syntax Overview

```sql
COPY INTO <table_name>
FROM <stage_name | external_location>
[ FILES = ( '<file1>' [ , '<file2>' ... ] ) ]
[ PATTERN = '<regex_pattern>' ]
[ FILE_FORMAT = ( TYPE = '<file_type>' [ format_options ] ) ]
[ ON_ERROR = '<action>' ]
[ FORCE = TRUE | FALSE ]
[ PURGE = TRUE | FALSE ]
[ VALIDATION_MODE = '<mode>' ]
[ SIZE_LIMIT = <number> ]
[ MAX_FILE_SIZE = <number> ]
[ RETURN_FAILED_ONLY = TRUE | FALSE ];
````

---

## 🧩 Common COPY Options

| Option                 | Description                                     | Default            | Example                                                     |
| ---------------------- | ----------------------------------------------- | ------------------ | ----------------------------------------------------------- |
| **FILES**              | Load only specific files from the stage         | All files          | `FILES=('data1.csv','data2.csv')`                           |
| **PATTERN**            | Load files matching regex                       | All files          | `PATTERN='.*sales_2025.*.csv'`                              |
| **FILE_FORMAT**        | Specifies the file format type and options      | None (must be set) | `FILE_FORMAT=(TYPE='CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"')` |
| **ON_ERROR**           | Defines behavior on load errors                 | `ABORT_STATEMENT`  | `ON_ERROR='CONTINUE'`                                       |
| **FORCE**              | Reprocess files even if already loaded          | `FALSE`            | `FORCE=TRUE`                                                |
| **PURGE**              | Delete files from stage after successful load   | `FALSE`            | `PURGE=TRUE`                                                |
| **VALIDATION_MODE**    | Validate files without loading                  | None               | `VALIDATION_MODE=RETURN_ERRORS`                             |
| **SIZE_LIMIT**         | Limits total bytes loaded in one COPY execution | Unlimited          | `SIZE_LIMIT=5000000000`                                     |
| **MAX_FILE_SIZE**      | Defines max bytes per output file during unload | Unlimited          | `MAX_FILE_SIZE=104857600`                                   |
| **RETURN_FAILED_ONLY** | Show only failed file loads in results          | `FALSE`            | `RETURN_FAILED_ONLY=TRUE`                                   |

---

## 🧱 1. FILES Option

Specifies the exact files to be loaded from the stage.

```sql
COPY INTO SALES
FROM @SALES_STAGE
FILES = ('sales_2025_01.csv', 'sales_2025_02.csv')
FILE_FORMAT = (TYPE='CSV');
```

✅ **Use Case:** Useful when stage contains multiple files and you want to load specific ones.

---

## 🧱 2. PATTERN Option

Loads files that match a **regular expression pattern**.

```sql
COPY INTO SALES
FROM @SALES_STAGE
PATTERN = '.*2025.*.csv'
FILE_FORMAT = (TYPE='CSV');
```

✅ **Use Case:** Load all files from 2025 without listing them manually.

---

## 🧱 3. FILE_FORMAT Option

Defines the format of input or output files.

```sql
COPY INTO SALES
FROM @SALES_STAGE
FILE_FORMAT = (TYPE='CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);
```

You can also reference a predefined file format:

```sql
COPY INTO SALES
FROM @SALES_STAGE
FILE_FORMAT = (FORMAT_NAME = 'MY_CSV_FORMAT');
```

---

## 🧱 4. ON_ERROR Option

Specifies what to do when data loading errors occur.

| Value               | Description                              |
| ------------------- | ---------------------------------------- |
| `'ABORT_STATEMENT'` | Stop the load on first error (default)   |
| `'CONTINUE'`        | Skip problematic rows and continue       |
| `'SKIP_FILE'`       | Skip file when error occurs              |
| `'SKIP_FILE_<n>%'`  | Skip file if error percentage exceeds n% |
| `'SKIP_FILE_<n>'`   | Skip file if number of errors exceeds n  |

Example:

```sql
COPY INTO SALES FROM @STAGE ON_ERROR='CONTINUE';
```

✅ **Use Case:** Tolerate minor data quality issues during bulk loads.

---

## 🧱 5. FORCE Option

Forces Snowflake to **reload files even if already loaded**.

```sql
COPY INTO SALES FROM @STAGE FORCE=TRUE;
```

✅ **Use Case:** When reloading same files after a schema or format change.

---

## 🧱 6. PURGE Option

Deletes data files from the stage after successful load.

```sql
COPY INTO SALES FROM @STAGE PURGE=TRUE;
```

✅ **Use Case:** Prevents reprocessing and saves storage cost after successful ingestion.

---

## 🧱 7. VALIDATION_MODE Option

Validates data files without loading them into a table.

| Mode                | Description                          |
| ------------------- | ------------------------------------ |
| `RETURN_ERRORS`     | Displays first error per file        |
| `RETURN_ALL_ERRORS` | Displays all errors                  |
| `RETURN_n_ROWS`     | Returns n sample rows for inspection |

Example:

```sql
COPY INTO SALES
FROM @STAGE
FILE_FORMAT = (TYPE='CSV')
VALIDATION_MODE = RETURN_ERRORS;
```

✅ **Use Case:** Debug data issues before performing actual load.

---

## 🧱 8. SIZE_LIMIT Option

Restricts total data size processed in one command execution.

```sql
COPY INTO SALES
FROM @STAGE
SIZE_LIMIT = 5000000000; -- 5GB
```

✅ **Use Case:** Control resource usage and prevent oversized jobs.

---

## 🧱 9. MAX_FILE_SIZE Option

Controls maximum output file size when **unloading data** from Snowflake.

```sql
COPY INTO @MY_STAGE/sales_data/
FROM SALES
FILE_FORMAT = (TYPE='CSV')
MAX_FILE_SIZE = 104857600; -- 100MB
```

✅ **Use Case:** Split large exports into manageable file sizes.

---

## 🧱 10. RETURN_FAILED_ONLY Option

Displays only failed file records in COPY results.

```sql
COPY INTO SALES
FROM @STAGE
FILE_FORMAT = (TYPE='CSV')
RETURN_FAILED_ONLY=TRUE;
```

✅ **Use Case:** Focus on failed loads for error debugging.

---

## ⚡ Additional COPY Options for Unload Operations

| Option               | Description                     | Example                                       |
| -------------------- | ------------------------------- | --------------------------------------------- |
| **HEADER**           | Include column headers          | `HEADER=TRUE`                                 |
| **OVERWRITE**        | Replace existing files          | `OVERWRITE=TRUE`                              |
| **SINGLE**           | Combine output into one file    | `SINGLE=TRUE`                                 |
| **COMPRESSION**      | Specify compression type        | `FILE_FORMAT=(TYPE='CSV' COMPRESSION='GZIP')` |
| **INCLUDE_QUERY_ID** | Add query ID to output filename | `INCLUDE_QUERY_ID=TRUE`                       |

Example:

```sql
COPY INTO @MY_STAGE/export/
FROM SALES
FILE_FORMAT=(TYPE='CSV' COMPRESSION='GZIP')
HEADER=TRUE
SINGLE=TRUE
OVERWRITE=TRUE;
```

---

## 🧠 COPY Option Workflow Summary

```
[Stage Files]
     ↓
[SELECT Files via FILES or PATTERN]
     ↓
[Apply FILE_FORMAT]
     ↓
[Handle Errors → ON_ERROR]
     ↓
[Post Load Actions → PURGE, FORCE]
     ↓
[Validation / Monitoring]
```

---

## 🧰 Best Practices

| Practice                         | Description                     |
| -------------------------------- | ------------------------------- |
| Use **FILE_FORMAT references**   | Keeps data ingestion consistent |
| Enable **VALIDATION_MODE**       | Catch issues before real load   |
| Use **PURGE** for cleanup        | Save storage costs              |
| Use **FORCE** only when needed   | Prevent duplicate loading       |
| Monitor **COPY_HISTORY**         | Track performance and errors    |
| Combine with **Streams & Tasks** | Automate continuous loads       |

---

## 🧩 Summary Table

| Option                 | Purpose                       | Example                     |
| ---------------------- | ----------------------------- | --------------------------- |
| **FILES**              | Select specific files         | `FILES=('f1.csv','f2.csv')` |
| **PATTERN**            | Regex file selection          | `PATTERN='.*2025.*'`        |
| **FILE_FORMAT**        | Define file structure         | `TYPE='CSV'`                |
| **ON_ERROR**           | Control load failure behavior | `'CONTINUE'`                |
| **FORCE**              | Reload existing files         | `FORCE=TRUE`                |
| **PURGE**              | Delete loaded files           | `PURGE=TRUE`                |
| **VALIDATION_MODE**    | Validate data before load     | `RETURN_ERRORS`             |
| **SIZE_LIMIT**         | Restrict total data processed | `SIZE_LIMIT=5000000000`     |
| **MAX_FILE_SIZE**      | Split export file sizes       | `MAX_FILE_SIZE=104857600`   |
| **RETURN_FAILED_ONLY** | Show only failed loads        | `RETURN_FAILED_ONLY=TRUE`   |

> 💡 **Tip:** COPY options make the command flexible, efficient, and reliable for a variety of bulk data loading and unloading scenarios in Snowflake.