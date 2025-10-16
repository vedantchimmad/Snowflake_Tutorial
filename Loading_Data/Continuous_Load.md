# 🔄 Continuous Load in Snowflake

---

## 🧭 Overview

**Continuous Load** in Snowflake refers to **automated, near real-time data ingestion** using **Snowpipe**. Unlike bulk load, which ingests large datasets in batches, Snowpipe continuously loads data **as soon as it arrives** in a stage (internal or external).

> 🧠 Ideal for **streaming or incremental data pipelines**, real-time analytics, and event-driven workloads.

---

## 🧩 Key Concepts

| Concept | Description |
|---------|-------------|
| **Snowpipe** | Managed service for automatic, continuous data ingestion |
| **Stage** | Location where new files arrive before loading |
| **File Format** | Defines how Snowflake interprets staged files (CSV, JSON, Parquet, Avro) |
| **Pipe** | Object that defines the continuous load workflow (source stage + target table + COPY command) |
| **Event Notification** | Cloud events (e.g., S3 PUT) trigger Snowpipe to ingest files |
| **Automatic Load** | Snowpipe detects new files and loads them without manual intervention |
| **Stream / Task** | Optional objects to process incremental changes after ingestion |

---

## ⚙️ Steps for Continuous Load

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

### 3️⃣ Create File Format

```sql
CREATE OR REPLACE FILE FORMAT CSV_FORMAT
TYPE = 'CSV'
FIELD_OPTIONALLY_ENCLOSED_BY='"'
SKIP_HEADER = 1;
```

---

## ⚙️ 4️⃣ Create Snowpipe

```sql
CREATE OR REPLACE PIPE SALES_PIPE
AUTO_INGEST = TRUE
AS
COPY INTO SALES
FROM @SALES_STAGE
FILE_FORMAT = (FORMAT_NAME = 'CSV_FORMAT');
```

* `AUTO_INGEST = TRUE` enables **event-based triggers** for automatic ingestion.
* Pipe monitors the stage for **new files**.

---

## ⚙️ 5️⃣ Configure Event Notifications (AWS S3 Example)

* Create **S3 Event Notification** for PUT events.
* Connect it to **Snowflake notification service**.
* Snowpipe automatically **loads new files** when they appear in the stage.

---

## ⚙️ 6️⃣ Load Data with Snowpipe

* Simply place files in the stage:

```bash
# Using SnowSQL CLI
snowsql -c my_conn -q "PUT file:///local/path/sales_new.csv @SALES_STAGE;"
```

* Snowpipe detects the new file and executes COPY INTO automatically.

---

## ⚙️ 7️⃣ Monitor Snowpipe Load History

```sql
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.COPY_HISTORY
WHERE PIPE_NAME = 'SALES_PIPE'
ORDER BY LAST_LOAD_TIME DESC;
```

* Check **status**, **rows loaded**, and **errors**.

---

## 🧩 Best Practices for Continuous Load

| Practice                             | Description                                       |
| ------------------------------------ | ------------------------------------------------- |
| Use **staging** for incoming files   | Organize internal or external stages              |
| Configure **event-based triggers**   | Use S3/Azure/GCS notifications for auto ingestion |
| Use **compressed files**             | Faster ingestion and lower cost                   |
| Monitor **COPY\_HISTORY**            | Ensure all files are ingested successfully        |
| Define **file formats consistently** | Avoid parsing errors                              |
| Leverage **Streams & Tasks**         | Process incremental data post ingestion           |
| Use **role-based access control**    | Restrict who can modify pipes and stages          |

---

## ⚡ Continuous Load Workflow

```
[New File Arrives in Stage] 
       ↓  Event Trigger (S3, GCS, Azure)
[Snowpipe Detects File] 
       ↓  COPY INTO Table
[Target Table Updated] 
       ↓  Stream/Task for further processing (optional)
```

---

## 🧩 Summary

| Concept                | Description                                                   |
| ---------------------- | ------------------------------------------------------------- |
| **Continuous Load**    | Automated, real-time data ingestion                           |
| **Snowpipe**           | Managed service for continuous loading                        |
| **Stage**              | Location for new incoming files                               |
| **Pipe**               | Defines auto-ingest workflow                                  |
| **Event Notification** | Triggers Snowpipe when files arrive                           |
| **COPY\_HISTORY**      | Tracks file ingestion status                                  |
| **Best Practices**     | Use staging, event notifications, monitoring, and compression |

> 💡 **Tip:** Use **Snowpipe** for incremental or streaming data and **bulk load** for historical batch data to design efficient data pipelines.
