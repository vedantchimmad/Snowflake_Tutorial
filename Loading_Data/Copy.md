# 📋 COPY Command in Snowflake

---

## 🧭 Overview

The **`COPY INTO` command** in Snowflake is one of the most important SQL commands used to **load data from a stage into a table** or **unload data from a table into a stage**.

> 💡 It supports both **bulk** and **continuous** data loading (via Snowpipe), allowing ingestion from **internal** and **external stages** (like S3, Azure Blob, or GCS).

---

## 🧩 Syntax

### 🔹 Load Data *Into* a Table
```sql
COPY INTO <table_name>
FROM <stage_name | external_location>
[ FILES = ( '<file1>' [ , '<file2>' ... ] ) ]
[ PATTERN = '<regex_pattern>' ]
[ FILE_FORMAT = ( TYPE = '<file_type>' [ format_options ] ) ]
[ ON_ERROR = '<action>' ]
[ FORCE = TRUE | FALSE ]
[ PURGE = TRUE | FALSE ]
[ VALIDATION_MODE = RETURN_<errors|all_errors|n_rows> ]
[ SIZE_LIMIT = <number> ]
[ MAX_FILE_SIZE = <number> ];
````

### 🔹 Unload Data *From* a Table

```sql
COPY INTO @<stage_name>/<path>/
FROM <table_name>
FILE_FORMAT = ( TYPE = '<file_type>' [ format_options ] )
[ HEADER = TRUE | FALSE ]
[ SINGLE = TRUE | FALSE ]
[ OVERWRITE = TRUE | FALSE ];
```

---

## 🧱 Common Use Case: Load Data from Stage to Table

```sql
COPY INTO SALES
FROM @SALES_STAGE
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1)
ON_ERROR = 'CONTINUE';
```

✅ **Explanation:**

* `SALES` → Target table
* `@SALES_STAGE` → Internal or external stage
* `FILE_FORMAT` → CSV file interpretation
* `ON_ERROR='CONTINUE'` → Skips problematic rows

---

## ⚙️ Load Data from Specific File(s)

```sql
COPY INTO SALES
FROM @SALES_STAGE
FILES = ('sales_2025_01.csv', 'sales_2025_02.csv')
FILE_FORMAT = (TYPE='CSV');
```

---

## 🎯 Load Using File Pattern

```sql
COPY INTO SALES
FROM @SALES_STAGE
PATTERN = '.*2025.*.csv'
FILE_FORMAT = (TYPE='CSV');
```

> 🧠 Uses **regex pattern** to match multiple files automatically.

---

## 💾 Example: Load from External S3 Stage

```sql
CREATE OR REPLACE STAGE SALES_S3_STAGE
URL = 's3://mybucket/sales/'
CREDENTIALS = (AWS_KEY_ID='XXX' AWS_SECRET_KEY='XXX');

COPY INTO SALES
FROM @SALES_S3_STAGE
FILE_FORMAT = (TYPE='CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"')
ON_ERROR = 'SKIP_FILE';
```

---

## 📤 Unload Data from Table to Stage

```sql
COPY INTO @MY_STAGE/sales_export/
FROM SALES
FILE_FORMAT = (TYPE='CSV' COMPRESSION='GZIP')
HEADER = TRUE
OVERWRITE = TRUE;
```

✅ **Purpose:** Exports table data to internal/external stage in compressed CSV format.

---

## ⚙️ COPY INTO with File Format Reference

```sql
CREATE FILE FORMAT MY_CSV_FORMAT
TYPE='CSV'
FIELD_OPTIONALLY_ENCLOSED_BY='"'
SKIP_HEADER=1;

COPY INTO SALES
FROM @SALES_STAGE
FILE_FORMAT = (FORMAT_NAME = 'MY_CSV_FORMAT');
```

---

## ⚡ COPY INTO with JSON or Parquet

### 🔹 Load JSON Data

```sql
COPY INTO EVENTS
FROM @JSON_STAGE
FILE_FORMAT = (TYPE='JSON');
```

### 🔹 Load Parquet Data

```sql
COPY INTO SALES
FROM @PARQUET_STAGE
FILE_FORMAT = (TYPE='PARQUET');
```

> 🧠 Snowflake automatically reads **column names** and **data types** from Parquet and Avro formats.

---

## 🧩 Error Handling Options

| Option               | Description                                 |
| -------------------- | ------------------------------------------- |
| `'ABORT_STATEMENT'`  | Stop loading when an error occurs (default) |
| `'CONTINUE'`         | Skip problematic rows and continue loading  |
| `'SKIP_FILE'`        | Skip entire file if an error occurs         |
| `'SKIP_FILE_<num>%'` | Skip file if number of errors > % threshold |

Example:

```sql
COPY INTO SALES FROM @STAGE ON_ERROR='CONTINUE';
```

---

## 🧰 Advanced Options

| Option            | Description                           | Example                                                      |
| ----------------- | ------------------------------------- | ------------------------------------------------------------ |
| `FORCE=TRUE`      | Reload files even if already loaded   | `COPY INTO SALES FROM @STAGE FORCE=TRUE;`                    |
| `PURGE=TRUE`      | Delete files from stage after loading | `COPY INTO SALES FROM @STAGE PURGE=TRUE;`                    |
| `VALIDATION_MODE` | Validate data files without loading   | `COPY INTO SALES FROM @STAGE VALIDATION_MODE=RETURN_ERRORS;` |
| `SIZE_LIMIT`      | Restrict load size                    | `SIZE_LIMIT = 5000000000`                                    |
| `MAX_FILE_SIZE`   | Split large data into smaller chunks  | `MAX_FILE_SIZE = 5000000000`                                 |

---

## 📊 COPY Command Load History

Monitor recent data load operations:

```sql
SELECT *
FROM INFORMATION_SCHEMA.LOAD_HISTORY
WHERE TABLE_NAME = 'SALES'
ORDER BY LAST_LOAD_TIME DESC;
```

---

## 🧠 Best Practices

| Practice                     | Description                                 |
| ---------------------------- | ------------------------------------------- |
| Use **staging area**         | Store all files in internal/external stages |
| Create **file formats**      | Maintain consistent schema parsing          |
| Use **ON_ERROR**             | Handle bad records gracefully               |
| Enable **compression**       | Speeds up data transfer                     |
| Use **VALIDATION_MODE**      | Test data before full load                  |
| Monitor **COPY_HISTORY**     | Audit and debug ingestion issues            |
| Use **AUTO_INGEST Snowpipe** | For continuous (real-time) loads            |

---

## ⚡ COPY INTO Workflow Summary

```
[Source Files]
      ↓
 [Stage (@Internal or External)]
      ↓  COPY INTO
[Snowflake Table]
      ↓
 [Load History / Monitoring]
```

---

## 🧩 Summary

| Direction    | Command                          | Purpose                         |
| ------------ | -------------------------------- | ------------------------------- |
| Load         | `COPY INTO <table>`              | Load data from stage to table   |
| Unload       | `COPY INTO @stage`               | Export data from table to stage |
| File Support | CSV, JSON, Parquet, Avro         | Multiple file types supported   |
| Key Features | FORCE, ON_ERROR, VALIDATION_MODE | Flexible ingestion              |
| Monitoring   | LOAD_HISTORY                     | View job status and history     |

> 💡 **Tip:** Use `COPY INTO` for batch loads and integrate with **Snowpipe** for automated, event-driven, real-time ingestion.

