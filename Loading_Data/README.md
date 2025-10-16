# 📥 Loading Data in Snowflake

---

## 🧭 Overview

**Loading Data in Snowflake** refers to the process of **ingesting structured or semi-structured data** from external sources into Snowflake tables. Snowflake supports multiple **loading mechanisms**, including bulk load, continuous load, and real-time ingestion.

> 🧠 Snowflake separates **storage** and **compute**, so data can be loaded quickly and efficiently without affecting queries on existing tables.

---

## 🧩 Data Loading Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| 🗄️ **Bulk Load (COPY INTO)** | Load data from staged files into tables | CSV, JSON, Parquet, Avro, ORC |
| 🔄 **Continuous Load (Snowpipe)** | Auto-ingest files as they arrive in a stage | Near real-time streaming |
| 🧱 **Manual Insert** | Use `INSERT` statements to load rows | Small datasets or testing |
| 🔗 **External Tables** | Query data directly in cloud storage without loading | S3, Azure Blob, GCS |
| 📊 **Third-Party Tools** | ETL/ELT tools like Matillion, Fivetran, Informatica | Enterprise data pipelines |

---

## 🧱 Stages in Snowflake

A **stage** is a **location for storing files** before loading them into tables.

| Stage Type | Description | Example |
|------------|-------------|---------|
| **Internal Stage** | Storage within Snowflake | `@my_stage` |
| **User Stage** | Personal temporary storage for a user | `@~` |
| **Table Stage** | Auto-created stage for a specific table | `@%my_table` |
| **External Stage** | Cloud storage outside Snowflake | `@my_s3_stage` |

---

## ⚙️ 1. Creating a Table

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

---

## ⚙️ 2. Creating a Stage

### 🔹 Internal Stage

```sql
CREATE OR REPLACE STAGE SALES_STAGE;
```

### 🔹 External Stage (AWS S3)

```sql
CREATE OR REPLACE STAGE SALES_S3_STAGE
URL='s3://mybucket/sales/'
CREDENTIALS=(AWS_KEY_ID='XXXX' AWS_SECRET_KEY='XXXX');
```

---

## ⚙️ 3. Uploading Files to Stage

### 🔹 SnowSQL Command (CLI)

```bash
# Upload local file to internal stage
snowsql -c my_conn -q "PUT file:///local/path/sales.csv @SALES_STAGE;"
```

---

## ⚙️ 4. Loading Data into Table

### 🔹 Using COPY INTO for CSV

```sql
COPY INTO SALES
FROM @SALES_STAGE/sales.csv
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);
```

### 🔹 File Format for JSON

```sql
COPY INTO SALES
FROM @SALES_STAGE/sales.json
FILE_FORMAT = (TYPE = 'JSON');
```

---

## 🧩 5. Continuous Data Loading with Snowpipe

### 🔹 Create Snowpipe

```sql
CREATE OR REPLACE PIPE SALES_PIPE
AUTO_INGEST = TRUE
AS
COPY INTO SALES
FROM @SALES_STAGE
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);
```

### 🔹 Enable Event Notification (AWS S3)

* Configure **S3 Event Notification** to trigger Snowpipe on new file arrival.

> ✅ Snowpipe allows **near real-time ingestion** without manual intervention.

---

## ⚙️ 6. Manual Inserts

```sql
INSERT INTO SALES (ORDER_ID, CUSTOMER_ID, REGION, PRODUCT, SALES_AMOUNT, ORDER_DATE)
VALUES (101, 1, 'East', 'Laptop', 1200, '2025-10-15');
```

---

## ⚙️ 7. Loading Semi-Structured Data (JSON, Parquet, Avro)

### 🔹 JSON Example

```sql
COPY INTO SALES
FROM @SALES_STAGE/sales.json
FILE_FORMAT = (TYPE = 'JSON');
```

### 🔹 Parquet Example

```sql
COPY INTO SALES
FROM @SALES_STAGE/sales.parquet
FILE_FORMAT = (TYPE = 'PARQUET');
```

---

## ⚙️ 8. Loading Data from External Tables

```sql
CREATE EXTERNAL TABLE SALES_EXT (
    ORDER_ID INT,
    CUSTOMER_ID INT,
    REGION STRING,
    PRODUCT STRING,
    SALES_AMOUNT FLOAT,
    ORDER_DATE DATE
)
WITH LOCATION = '@my_s3_stage'
FILE_FORMAT = (TYPE = 'CSV');
```

* Data remains in **S3**; queries read it directly.

---

## 🧩 9. Monitoring Load History

```sql
-- View COPY INTO command history
SELECT *
FROM INFORMATION_SCHEMA.LOAD_HISTORY
WHERE TABLE_NAME = 'SALES'
ORDER BY LAST_LOAD_TIME DESC;
```

---

## 🧩 10. Best Practices for Data Engineers

| Practice                                     | Description                                         |
| -------------------------------------------- | --------------------------------------------------- |
| 🧾 Use **file formats** for standardization  | CSV, JSON, Parquet, Avro                            |
| 🔄 Use **Snowpipe** for continuous ingestion | Near real-time updates                              |
| 📦 Use **staging** for bulk loads            | Avoid direct uploads to table                       |
| 🧩 Monitor **LOAD\_HISTORY**                 | Track failed or partial loads                       |
| ⚡ Use **multi-file load**                    | Load multiple files in one COPY command             |
| 🕵️ Validate Data                            | Check row count and transformations after load      |
| 🔐 Secure Stages                             | Use proper roles and encryption for external stages |

---

## 🧮 Example End-to-End Load Flow

```
[Local CSV] → PUT → [Internal Stage @SALES_STAGE] → COPY INTO → [SALES Table]
             → Optional: Snowpipe auto-ingest → Near Real-time Table Updates
```

---

## 🧩 Summary

| Concept            | Description                                              |
| ------------------ | -------------------------------------------------------- |
| **Stage**          | Temporary storage for files before loading               |
| **COPY INTO**      | Bulk load command for tables                             |
| **Snowpipe**       | Continuous auto-ingest for real-time data                |
| **File Formats**   | Define how Snowflake interprets CSV, JSON, Parquet, etc. |
| **Monitoring**     | Use LOAD\_HISTORY and INFORMATION\_SCHEMA for tracking   |
| **Best Practices** | Use staging, auto-ingest, and validate loaded data       |

> 💡 **Tip:** For efficient pipelines, combine **bulk COPY for historical data** and **Snowpipe for incremental updates**.

