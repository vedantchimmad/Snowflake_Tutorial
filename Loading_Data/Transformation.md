# 🔄 Transforming Data in Snowflake

---

## 🧭 Overview

Data transformation in **Snowflake** refers to the process of **modifying, cleansing, enriching, or reshaping** raw data into a usable format for analytics and reporting.  
It is typically performed using **SQL transformations**, **Snowflake Streams & Tasks**, or **Snowpark (Python, Java, Scala)**.

> 💡 Transformations in Snowflake can be applied **after loading data (ELT)** — leveraging Snowflake’s compute power to process data efficiently.

---

## ⚙️ Common Transformation Methods

| Method | Description | Use Case |
|--------|--------------|----------|
| **SQL Queries (DML)** | Basic transformations using SQL statements | Joins, filtering, aggregations |
| **CTAS (Create Table As Select)** | Create new transformed tables | Snapshot or derived tables |
| **Streams & Tasks** | Automate incremental transformations | Real-time data pipelines |
| **Snowpark API** | Transform data using Python, Java, or Scala | Complex logic or ML integration |
| **Stored Procedures** | Execute multi-step transformation workflows | Batch or scheduled transformations |
| **Materialized Views** | Precomputed, automatically refreshed transformations | Fast queries on preprocessed data |

---

## 🧩 1. SQL-Based Transformations

### 🔹 Basic Example

```sql
CREATE OR REPLACE TABLE CLEANED_SALES AS
SELECT
    ORDER_ID,
    CUSTOMER_ID,
    AMOUNT::DECIMAL(10,2) AS SALES_AMOUNT,
    TO_DATE(ORDER_DATE, 'YYYY-MM-DD') AS ORDER_DATE,
    CASE
        WHEN REGION IS NULL THEN 'UNKNOWN'
        ELSE REGION
    END AS REGION
FROM RAW_SALES
WHERE AMOUNT IS NOT NULL;
````

✅ **Explanation**

* Converts `AMOUNT` to decimal type
* Converts `ORDER_DATE` string to proper date format
* Replaces NULL regions with `'UNKNOWN'`
* Filters invalid sales records

---

## 🧩 2. Using `CTAS` — Create Table As Select

```sql
CREATE OR REPLACE TABLE CUSTOMER_SUMMARY AS
SELECT
    CUSTOMER_ID,
    COUNT(*) AS TOTAL_ORDERS,
    SUM(SALES_AMOUNT) AS TOTAL_SALES,
    AVG(SALES_AMOUNT) AS AVG_SALES
FROM CLEANED_SALES
GROUP BY CUSTOMER_ID;
```

> 💡 `CTAS` is an efficient way to store transformed data in a new physical table.

---

## 🧩 3. Using Views for Logical Transformations

```sql
CREATE OR REPLACE VIEW ACTIVE_CUSTOMERS AS
SELECT
    CUSTOMER_ID,
    CUSTOMER_NAME,
    EMAIL,
    LAST_PURCHASE_DATE
FROM CUSTOMERS
WHERE STATUS = 'ACTIVE';
```

✅ **Purpose:** Create a **logical layer** for transformations without storing additional data.

---

## 🧩 4. Using Materialized Views

```sql
CREATE OR REPLACE MATERIALIZED VIEW MONTHLY_SALES AS
SELECT
    DATE_TRUNC('MONTH', ORDER_DATE) AS MONTH,
    SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM CLEANED_SALES
GROUP BY 1;
```

> 🧠 Materialized views **store results physically** and **auto-refresh** for fast query performance.

---

## 🧩 5. Incremental Transformations using Streams & Tasks

### 🔹 Step 1: Create Stream

Tracks changes (inserts/updates/deletes) in a source table.

```sql
CREATE OR REPLACE STREAM SALES_STREAM ON TABLE CLEANED_SALES;
```

### 🔹 Step 2: Create Transformation Task

Runs on a schedule or event to process new changes.

```sql
CREATE OR REPLACE TASK SALES_AGGREGATE_TASK
WAREHOUSE = 'TRANSFORM_WH'
SCHEDULE = '1 MINUTE'
AS
MERGE INTO SALES_AGGREGATE AS T
USING (
    SELECT CUSTOMER_ID, SUM(SALES_AMOUNT) AS TOTAL_SALES
    FROM CLEANED_SALES
    WHERE METADATA$ACTION = 'INSERT'
    GROUP BY CUSTOMER_ID
) S
ON T.CUSTOMER_ID = S.CUSTOMER_ID
WHEN MATCHED THEN
    UPDATE SET T.TOTAL_SALES = T.TOTAL_SALES + S.TOTAL_SALES
WHEN NOT MATCHED THEN
    INSERT (CUSTOMER_ID, TOTAL_SALES) VALUES (S.CUSTOMER_ID, S.TOTAL_SALES);
```

✅ **Purpose:** Continuously update aggregate table based on new data arriving in `CLEANED_SALES`.

---

## 🧩 6. Data Transformation using Snowpark (Python)

```python
from snowflake.snowpark import Session
from snowflake.snowpark.functions import col, when

session = Session.builder.configs({...}).create()

df = session.table("RAW_SALES")

transformed = (
    df.with_column("REGION", when(col("REGION").is_null(), "UNKNOWN").otherwise(col("REGION")))
      .with_column("SALES_AMOUNT", col("AMOUNT").cast("DECIMAL(10,2)"))
      .filter(col("AMOUNT").is_not_null())
)

transformed.write.save_as_table("CLEANED_SALES", mode="overwrite")
```

> 💡 Snowpark lets you write **Python-based transformation logic** directly inside Snowflake.

---

## 🧩 7. Using Stored Procedures for Multi-Step ETL

```sql
CREATE OR REPLACE PROCEDURE RUN_ETL_PIPELINE()
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN
    INSERT INTO CLEANED_SALES
    SELECT * FROM RAW_SALES WHERE AMOUNT IS NOT NULL;

    INSERT INTO SALES_AGGREGATE
    SELECT CUSTOMER_ID, SUM(SALES_AMOUNT)
    FROM CLEANED_SALES
    GROUP BY CUSTOMER_ID;

    RETURN 'ETL Pipeline executed successfully.';
END;
$$;
```

✅ **Purpose:** Encapsulates ETL logic for reuse and scheduling.

---

## 🧩 8. Using Functions for Column-Level Transformations

| Function              | Example                                  | Description       |
| --------------------- | ---------------------------------------- | ----------------- |
| `CAST()`              | `CAST(AMOUNT AS DECIMAL(10,2))`          | Convert data type |
| `COALESCE()`          | `COALESCE(REGION, 'UNKNOWN')`            | Replace nulls     |
| `UPPER()` / `LOWER()` | `UPPER(CUSTOMER_NAME)`                   | Format text       |
| `DATE_TRUNC()`        | `DATE_TRUNC('MONTH', ORDER_DATE)`        | Truncate date     |
| `REGEXP_REPLACE()`    | `REGEXP_REPLACE(EMAIL, '\\s+', '')`      | Clean strings     |
| `CASE WHEN`           | `CASE WHEN STATUS='A' THEN 'ACTIVE' END` | Conditional logic |

---

## 🧰 9. Using Temporary or Transient Tables

```sql
CREATE OR REPLACE TEMPORARY TABLE TEMP_SALES AS
SELECT * FROM CLEANED_SALES WHERE SALES_AMOUNT > 500;
```

> 🧠 Use **temporary tables** for intermediate transformation steps — they auto-drop after session ends.

---

## 📊 Example: Full ELT Transformation Flow

```
1️⃣ Load Raw Data  →  RAW_SALES
      ↓
2️⃣ Clean & Standardize → CLEANED_SALES
      ↓
3️⃣ Aggregate & Summarize → SALES_AGGREGATE
      ↓
4️⃣ Create Analytical Views → SALES_ANALYSIS_VIEW
```

---

## 🧠 Best Practices

| Practice                                  | Description                            |
| ----------------------------------------- | -------------------------------------- |
| Use **CTAS** for efficient table creation | Avoids extra `INSERT` overhead         |
| Use **Streams & Tasks** for automation    | Real-time incremental transformations  |
| Apply **file formats & data types early** | Reduces errors in downstream processes |
| Monitor **query history**                 | Check transformation performance       |
| Use **Snowpark for complex logic**        | Leverage Python/Scala APIs             |
| Schedule transformations using **Tasks**  | Ensures consistency and automation     |

---

## 🧩 Summary

| Method                 | Use Case                                | Example                     |
| ---------------------- | --------------------------------------- | --------------------------- |
| **SQL Queries**        | Standard column and row transformations | `SELECT`, `UPDATE`, `CASE`  |
| **CTAS**               | Create transformed tables               | `CREATE TABLE AS SELECT`    |
| **Streams & Tasks**    | Incremental/automated updates           | Real-time pipelines         |
| **Snowpark**           | Complex logic with code                 | Python/Java/Scala           |
| **Materialized Views** | Precomputed fast queries                | Monthly or daily aggregates |
| **Stored Procedures**  | Multi-step ETL orchestration            | Full automation             |

> 💡 **Tip:** Use Snowflake’s compute scalability to perform transformations directly in the cloud, minimizing data movement and improving performance.
