# 📦 Bulk Load in Snowflake

---

## 🧭 Overview

**Bulk Load** in Snowflake refers to the **process of loading large volumes of data** from external files (CSV, JSON, Parquet, Avro, ORC) into Snowflake tables using the **COPY INTO command**.  

> 🧠 Bulk load is typically used for **historical or batch data ingestion**, as opposed to real-time loading (Snowpipe).

---

## 🧩 Key Concepts

| Concept | Description |
|---------|-------------|
| **Stage** | Temporary storage location for files before loading |
| **File Format** | Defines how Snowflake interprets file data (CSV, JSON, Parquet, Avro) |
| **COPY INTO** | SQL command used to load staged files into a table |
| **Internal Stage** | Stage within Snowflake account (`@my_stage`) |
| **External Stage** | Cloud storage stage (AWS S3, Azure Blob, GCS) |
| **Bulk Load** | Load multiple files at once into a table |
| **Load History** | Tracks status of COPY INTO commands and loaded files |

---

## ⚙️ Steps for Bulk Load

### 1️⃣ Create Target Table
```sql
CREATE OR REPLACE TABLE SALES(
    ORDER_ID INT,
    CUSTOMER_ID INT,
    REGION STRING,
    PRODUCT STRING,
    SALES_AMOUNT FLOAT,
    ORDER_DATE DATE
);
````

### 2️⃣ Create Stage

#### 🔹 Internal Stage

```sql
CREATE OR REPLACE STAGE SALES_STAGE;
```

#### 🔹 External Stage (AWS S3)

```sql
CREATE OR REPLACE STAGE SALES_S3_STAGE
URL='s3://mybucket/sales/'
CREDENTIALS=(AWS_KEY_ID='XXXX' AWS_SECRET_KEY='XXXX');
```

### 3️⃣ Upload Files to Stage

#### 🔹 Using SnowSQL CLI

```bash
# Upload local CSV files to internal stage
snowsql -c my_conn -q "PUT file:///local/path/sales_*.csv @SALES_STAGE;"
```

### 4️⃣ Create File Format (Optional)

```sql
CREATE OR REPLACE FILE FORMAT CSV_FORMAT
TYPE = 'CSV'
FIELD_OPTIONALLY_ENCLOSED_BY='"'
SKIP_HEADER = 1;
```

---

## ⚙️ 5️⃣ Load Data Using COPY INTO

### 🔹 Bulk Load Multiple Files

```sql
COPY INTO SALES
FROM @SALES_STAGE
FILE_FORMAT = (FORMAT_NAME = 'CSV_FORMAT');
```

* Loads **all files in the stage** into the target table.
* Supports wildcards (e.g., `sales_*.csv`) for multiple files.

### 🔹 Load JSON Data

```sql
COPY INTO SALES
FROM @SALES_STAGE
FILE_FORMAT = (TYPE = 'JSON');
```

### 🔹 Load Parquet / Avro Data

```sql
COPY INTO SALES
FROM @SALES_STAGE
FILE_FORMAT = (TYPE = 'PARQUET');
```

---

## ⚙️ 6️⃣ Monitor Load History

```sql
SELECT *
FROM INFORMATION_SCHEMA.LOAD_HISTORY
WHERE TABLE_NAME = 'SALES'
ORDER BY LAST_LOAD_TIME DESC;
```

* Check **loaded files**, **number of rows**, and **status** (success/failure).

---

## 🧩 Best Practices for Bulk Load

| Practice                        | Description                                   |
| ------------------------------- | --------------------------------------------- |
| Use **staging**                 | Keep files in a stage before loading          |
| Use **file formats**            | Standardize CSV, JSON, Parquet, or Avro       |
| Load **multiple files** at once | Efficient for large datasets                  |
| Enable **parallel loading**     | Snowflake automatically optimizes load        |
| Monitor **LOAD\_HISTORY**       | Detect failed or partial loads                |
| Compress files                  | Use gzip or other compression for faster load |
| Validate **row counts**         | Ensure all data is loaded correctly           |

---

## ⚡ Example: Bulk Load Workflow

```
[Local CSV files] 
      ↓  PUT
[Internal Stage @SALES_STAGE] 
      ↓  COPY INTO
[SALES Table in Snowflake]
      ↓  VERIFY
[Load History & Row Counts]
```

---

## 🧩 Summary

| Concept           | Description                                     |
| ----------------- | ----------------------------------------------- |
| **Bulk Load**     | Loading large volumes of data from staged files |
| **Stage**         | Temporary storage (internal or external)        |
| **COPY INTO**     | Command to move data from stage to table        |
| **File Format**   | Specifies data type and parsing rules           |
| **Monitoring**    | Use LOAD\_HISTORY and INFORMATION\_SCHEMA       |
| **Best Practice** | Use compression, parallel files, and validation |

> 💡 **Tip:** For large historical datasets, use **bulk load** with staged files and optimized file formats (Parquet or compressed CSV) for faster and cost-efficient ingestion.

