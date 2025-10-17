# ❄️ Snowpipe in Snowflake

---

## 🧭 Overview

**Snowpipe** is Snowflake’s **continuous data ingestion** service that automatically loads data as soon as it becomes available in a cloud storage location (like AWS S3, Azure Blob, or GCP Storage).

Unlike the manual `COPY INTO` command, **Snowpipe** is **serverless**, **auto-scaling**, and **event-driven**, enabling **real-time or near real-time** data ingestion.

> 💡 Snowpipe eliminates the need for batch jobs — new data is automatically loaded into Snowflake tables when files arrive in a stage.

---

## 🧩 Key Features

| Feature | Description |
|----------|--------------|
| **Continuous Data Loading** | Automatically loads new files as soon as they appear in a stage. |
| **Serverless Compute** | Snowflake manages the compute resources automatically — no warehouse management needed. |
| **Auto-Scalable** | Automatically scales based on incoming data volume. |
| **Event-Driven Architecture** | Integrates with AWS S3, Azure Blob, or GCP notifications. |
| **Cost-Efficient** | You’re charged only for the compute time used during data ingestion. |
| **Fault Tolerant** | Automatically retries failed loads and maintains file history. |

---

## 🧩 Snowpipe Architecture

```

┌──────────────────────────┐
│   Cloud Storage (S3,     │
│   Azure Blob, GCP)       │
└─────────────┬────────────┘
│
▼
┌──────────────────────────┐
│    Snowflake Stage       │
│ (External/Internal Stage)│
└─────────────┬────────────┘
│
▼
┌──────────────────────────┐
│     Snowpipe Service     │
│ (Monitors, Loads, Logs)  │
└─────────────┬────────────┘
│
▼
┌──────────────────────────┐
│  Target Table in Snowflake│
└──────────────────────────┘

````

---

## 🧩 1. Components of Snowpipe

| Component | Description |
|------------|--------------|
| **Stage** | Storage location where files are uploaded (S3, Azure, GCP, or internal). |
| **Pipe** | Defines how data is loaded (includes COPY INTO statement). |
| **File Format** | Defines the format (CSV, JSON, Parquet, etc.) and parsing rules. |
| **Storage Integration** | Connects Snowflake securely with cloud storage. |
| **Event Notification** | Triggers Snowpipe to start loading when files arrive. |

---

## 🧩 2. Creating a File Format

```sql
CREATE OR REPLACE FILE FORMAT my_csv_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1
NULL_IF = ('NULL', 'null');
````

---

## 🧩 3. Create an External Stage

For example, from **AWS S3**:

```sql
CREATE OR REPLACE STAGE my_s3_stage
URL = 's3://my-snowflake-bucket/data/'
STORAGE_INTEGRATION = s3_integration
FILE_FORMAT = my_csv_format;
```

---

## 🧩 4. Create Target Table

```sql
CREATE OR REPLACE TABLE SALES (
  ORDER_ID INT,
  CUSTOMER_ID INT,
  PRODUCT STRING,
  PRICE FLOAT,
  ORDER_DATE DATE
);
```

---

## 🧩 5. Create a Pipe

A **pipe** defines the Snowpipe data ingestion process.

```sql
CREATE OR REPLACE PIPE sales_pipe
AS
COPY INTO SALES
FROM @my_s3_stage
FILE_FORMAT = (FORMAT_NAME = my_csv_format)
ON_ERROR = 'CONTINUE';
```

### ✅ Notes:

* The pipe uses the `COPY INTO` command under the hood.
* The file load status is tracked by Snowpipe (files aren’t reloaded unless forced).

---

## 🧩 6. Triggering Snowpipe

### Option 1️⃣ — **Manually (REST API)**

Invoke the Snowpipe using REST API call:

```bash
POST https://<account>.snowflakecomputing.com/v1/data/pipes/sales_pipe/insertFiles
{
  "files": ["sales_2025_01.csv", "sales_2025_02.csv"]
}
```

### Option 2️⃣ — **Event-Driven (Recommended)**

Use cloud event notifications to trigger Snowpipe automatically:

* **AWS** → S3 Event Notifications → SNS → Snowpipe
* **Azure** → Event Grid → Snowpipe
* **GCP** → Pub/Sub → Snowpipe

> 🧠 When a file lands in cloud storage, the event notification triggers Snowpipe automatically to ingest the data.

---

## 🧩 7. Check Pipe Status

```sql
SHOW PIPES;
```

Output includes:

* Pipe name
* State (Running, Stopped)
* Auto-ingest status
* Last file load timestamp

---

## 🧩 8. Monitor Load History

### View Load History for a Specific Table

```sql
SELECT * FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(TABLE_NAME=>'SALES'))
ORDER BY LAST_LOAD_TIME DESC;
```

### Check Specific Pipe History

```sql
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.PIPE_USAGE_HISTORY
WHERE PIPE_NAME='SALES_PIPE';
```

---

## 🧩 9. Pausing or Resuming a Pipe

```sql
ALTER PIPE sales_pipe SET PIPE_EXECUTION_PAUSED = TRUE;  -- Pause
ALTER PIPE sales_pipe SET PIPE_EXECUTION_PAUSED = FALSE; -- Resume
```

---

## 🧩 10. Validate Snowpipe Configuration

### List Files in Stage

```sql
LIST @my_s3_stage;
```

### Test Copy in Validation Mode

```sql
COPY INTO SALES
FROM @my_s3_stage
VALIDATION_MODE='RETURN_ERRORS';
```

---

## 🧩 11. Snowpipe with Internal Stages

If using an **internal stage** (Snowflake-managed), files can be uploaded using Snowflake’s Web UI or SnowSQL:

```bash
PUT file://C:\data\sales_2025_01.csv @%SALES;
```

Then, create a pipe:

```sql
CREATE OR REPLACE PIPE internal_sales_pipe
AS
COPY INTO SALES FROM @%SALES
FILE_FORMAT = (TYPE='CSV' SKIP_HEADER=1);
```

---

## 🧩 12. Monitoring Snowpipe Activity

| View                                         | Description                        |
| -------------------------------------------- | ---------------------------------- |
| `SNOWFLAKE.ACCOUNT_USAGE.PIPE_USAGE_HISTORY` | Snowpipe load performance and cost |
| `INFORMATION_SCHEMA.COPY_HISTORY`            | File load success/failure details  |
| `SNOWFLAKE.ACCOUNT_USAGE.LOAD_HISTORY`       | Comprehensive load metrics         |
| `SHOW PIPES`                                 | Lists active/inactive pipes        |

---

## 🧩 13. Snowpipe Costing

| Component                     | Description                    | Billed?             |
| ----------------------------- | ------------------------------ | ------------------- |
| **Snowpipe Compute**          | Serverless compute for loading | ✅ Yes (per second)  |
| **Cloud Storage**             | File storage in S3/Azure/GCP   | ✅ Yes (by provider) |
| **Warehouse**                 | Not required for Snowpipe      | ❌ No                |
| **Data Storage in Snowflake** | Once loaded                    | ✅ Yes               |

> 💰 Snowpipe is cost-efficient — pay only for ingestion time, not idle compute.

---

## 🧩 14. Error Handling in Snowpipe

| Scenario                 | Snowpipe Behavior        |
| ------------------------ | ------------------------ |
| File already loaded      | Skips automatically      |
| File partially loaded    | Retries automatically    |
| File with data errors    | Logs error and continues |
| Event notification fails | Retried periodically     |

---

## 🧩 15. Example End-to-End Snowpipe Setup (AWS Example)

```sql
-- 1️⃣ Create File Format
CREATE FILE FORMAT csv_format TYPE='CSV' SKIP_HEADER=1;

-- 2️⃣ Create Stage
CREATE STAGE s3_stage
URL='s3://my-snowflake-bucket/data/'
STORAGE_INTEGRATION = s3_integration
FILE_FORMAT = csv_format;

-- 3️⃣ Create Table
CREATE TABLE SALES (
  ORDER_ID INT,
  PRODUCT STRING,
  PRICE FLOAT
);

-- 4️⃣ Create Pipe
CREATE OR REPLACE PIPE sales_pipe
AS
COPY INTO SALES
FROM @s3_stage
FILE_FORMAT = (FORMAT_NAME = csv_format)
ON_ERROR = 'CONTINUE';

-- 5️⃣ Verify Pipe
SHOW PIPES;

-- 6️⃣ Monitor Snowpipe Activity
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.PIPE_USAGE_HISTORY
WHERE PIPE_NAME = 'SALES_PIPE';
```

---

## 🧠 Best Practices

| Best Practice                                               | Benefit                     |
| ----------------------------------------------------------- | --------------------------- |
| Use **event notifications** instead of manual REST triggers | Fully automated ingestion   |
| Use **compressed files** (e.g., `.gz`)                      | Faster loading & lower cost |
| Validate files with `VALIDATION_MODE`                       | Prevents failed loads       |
| Store raw data in staging tables first                      | Easier data correction      |
| Monitor load history regularly                              | Detect issues early         |
| Set `ON_ERROR='CONTINUE'` for production pipelines          | Avoid full job failures     |

---

## 📊 Summary Table

| Component         | Description                | Example                         |
| ----------------- | -------------------------- | ------------------------------- |
| **Pipe**          | Definition of load process | `CREATE PIPE ... COPY INTO ...` |
| **Stage**         | Location of files          | `@my_s3_stage`                  |
| **File Format**   | Defines file structure     | `CREATE FILE FORMAT`            |
| **Event Trigger** | Auto-ingestion             | S3 → SNS → Snowpipe             |
| **Monitoring**    | Track loads                | `PIPE_USAGE_HISTORY`            |
| **Cost**          | Pay per ingestion second   | Serverless billing              |

---

> ✅ **Tip:**
> Use Snowpipe for **real-time ingestion** and schedule regular clean-up/validation jobs to maintain performance and data consistency.

