# ☁️ Loading Data from AWS into Snowflake

---

## 🧭 Overview

Snowflake provides **seamless integration** with **Amazon Web Services (AWS)** for **data loading**.  
You can load data directly from **Amazon S3 buckets** into **Snowflake tables** using **stages** and the **COPY INTO** command.

> 💡 AWS → S3 → Snowflake is the most common cloud data ingestion pattern for modern data pipelines.

---

## 🧩 Steps to Load Data from AWS S3 to Snowflake

| Step | Description |
|------|--------------|
| 1️⃣ | Prepare and upload data to Amazon S3 |
| 2️⃣ | Create a Snowflake table |
| 3️⃣ | Create a stage to connect S3 and Snowflake |
| 4️⃣ | Use the `COPY INTO` command to load data |
| 5️⃣ | Verify loaded data |
| 6️⃣ | (Optional) Automate using **Snowpipe** for continuous loading |

---

## ☁️ 1. Upload Data to S3 Bucket

Upload your CSV/JSON/Parquet files to an S3 bucket.

**Example structure:**
```

s3://my-snowflake-bucket/sales_data/
├── sales_2025_01.csv
├── sales_2025_02.csv
└── sales_2025_03.csv

````

Ensure your **AWS IAM role** or **access key** has:
- `s3:GetObject`
- `s3:ListBucket`
- `s3:PutObject` (optional for automation)

---

## 🧩 2. Create Target Table in Snowflake

```sql
CREATE OR REPLACE TABLE SALES (
  ORDER_ID INT,
  CUSTOMER_ID INT,
  PRODUCT STRING,
  QUANTITY INT,
  PRICE FLOAT,
  ORDER_DATE DATE
);
````

---

## 🧩 3. Create a Stage (to Access S3)

There are two ways to create a stage:

### 🔹 Option 1: Using AWS Access Keys

```sql
CREATE OR REPLACE STAGE my_s3_stage
URL = 's3://my-snowflake-bucket/sales_data/'
CREDENTIALS = (
  AWS_KEY_ID = 'YOUR_AWS_ACCESS_KEY_ID'
  AWS_SECRET_KEY = 'YOUR_AWS_SECRET_ACCESS_KEY'
)
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);
```

### 🔹 Option 2: Using IAM Role (Recommended)

```sql
CREATE OR REPLACE STAGE my_s3_stage
URL = 's3://my-snowflake-bucket/sales_data/'
STORAGE_INTEGRATION = my_integration
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);
```

> 🧠 **IAM Integration** is more secure — credentials are managed via Snowflake integrations instead of being exposed in SQL.

---

## 🧩 4. Create Storage Integration (for IAM Role Access)

```sql
CREATE OR REPLACE STORAGE INTEGRATION my_integration
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = S3
ENABLED = TRUE
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123456789012:role/mySnowflakeRole'
STORAGE_ALLOWED_LOCATIONS = ('s3://my-snowflake-bucket/sales_data/');
```

### View integration details:

```sql
DESC INTEGRATION my_integration;
```

Copy the `STORAGE_AWS_IAM_USER_ARN` and `STORAGE_AWS_EXTERNAL_ID` and update your IAM role trust policy accordingly.

---

## 🧩 5. Load Data into Snowflake Table

### 🔹 Load all files from the stage

```sql
COPY INTO SALES
FROM @my_s3_stage
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1)
ON_ERROR = 'CONTINUE';
```

### 🔹 Load specific file

```sql
COPY INTO SALES
FROM @my_s3_stage/sales_2025_01.csv
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);
```

---

## 🧩 6. Verify the Loaded Data

```sql
SELECT * FROM SALES LIMIT 10;
```

### View Load History

```sql
SELECT * FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(TABLE_NAME=>'SALES'));
```

---

## 🧩 7. File Format Options

| Parameter         | Description                  | Example                        |
| ----------------- | ---------------------------- | ------------------------------ |
| `TYPE`            | File type                    | `'CSV'`, `'JSON'`, `'PARQUET'` |
| `FIELD_DELIMITER` | Column separator             | `','`                          |
| `SKIP_HEADER`     | Skip header row              | `1`                            |
| `NULL_IF`         | Treat specific value as NULL | `('NULL', 'null')`             |
| `TRIM_SPACE`      | Trim spaces around fields    | `TRUE`                         |
| `COMPRESSION`     | File compression type        | `'GZIP'`, `'BZIP2'`            |

Example:

```sql
CREATE FILE FORMAT my_csv_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1
NULL_IF = ('NULL', 'null');
```

---

## 🧩 8. Validate Stage and Files

```sql
LIST @my_s3_stage;
```

> Shows all files available in the stage (S3 bucket).

---

## 🧩 9. Copy Command — Key Options

| Option            | Description                         | Example                           |
| ----------------- | ----------------------------------- | --------------------------------- |
| `ON_ERROR`        | Skip or abort on error              | `'CONTINUE'`, `'ABORT_STATEMENT'` |
| `FORCE`           | Reload files even if already loaded | `TRUE`                            |
| `VALIDATION_MODE` | Validate data before load           | `'RETURN_ERRORS'`                 |
| `PURGE`           | Remove files after load             | `TRUE`                            |

Example:

```sql
COPY INTO SALES
FROM @my_s3_stage
FILE_FORMAT = (FORMAT_NAME = my_csv_format)
ON_ERROR = 'CONTINUE'
FORCE = TRUE;
```

---

## 🧩 10. Continuous Load using Snowpipe

**Snowpipe** automatically loads new files from S3 to Snowflake as they arrive.

### Step 1: Create a Pipe

```sql
CREATE OR REPLACE PIPE sales_pipe
AS
COPY INTO SALES
FROM @my_s3_stage
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER=1)
ON_ERROR = 'CONTINUE';
```

### Step 2: Configure S3 Event Notification

Add an S3 event trigger to publish to an **AWS SNS topic** linked to Snowpipe.

> 🧠 This makes loading **event-driven** — new files automatically trigger Snowpipe to load into Snowflake.

---

## 🧩 11. Performance & Cost Optimization Tips

| Tip                                                           | Description |
| ------------------------------------------------------------- | ----------- |
| ✅ Use **compressed files** (e.g., `.gz`) to reduce load time  |             |
| ✅ Prefer **larger files** (100–250 MB) over many small ones   |             |
| ✅ Leverage **parallel COPY INTO** for faster ingestion        |             |
| ✅ Use **staging tables** for raw loads before transformations |             |
| ✅ Enable **auto-suspend warehouses** to save credits          |             |

---

## 🧩 12. Example End-to-End Script

```sql
-- Create Integration
CREATE OR REPLACE STORAGE INTEGRATION aws_integration
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = S3
ENABLED = TRUE
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123456789012:role/mySnowflakeRole'
STORAGE_ALLOWED_LOCATIONS = ('s3://my-snowflake-bucket/data/');

-- Create Stage
CREATE OR REPLACE STAGE s3_stage
URL = 's3://my-snowflake-bucket/data/'
STORAGE_INTEGRATION = aws_integration
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);

-- Create Table
CREATE OR REPLACE TABLE customer_data (
  id INT,
  name STRING,
  city STRING,
  country STRING
);

-- Load Data
COPY INTO customer_data
FROM @s3_stage
ON_ERROR = 'CONTINUE';
```

---

## 📊 Visual Data Flow Diagram

```
        ┌────────────────────┐
        │   AWS S3 Bucket    │
        │ (Raw CSV/Parquet)  │
        └────────┬───────────┘
                 │
                 ▼
        ┌────────────────────┐
        │  Snowflake Stage    │
        │ (S3 Integration)    │
        └────────┬───────────┘
                 │
                 ▼
        ┌────────────────────┐
        │   COPY INTO Table   │
        │  (Sales, Customer)  │
        └────────┬───────────┘
                 │
                 ▼
        ┌────────────────────┐
        │   Query & Analyze   │
        │     BI / Reports    │
        └────────────────────┘
```

---

## 🧠 Summary Table

| Step | Command / Action | Description                   |
| ---- | ---------------- | ----------------------------- |
| 1    | Upload to S3     | Store CSV/JSON/Parquet files  |
| 2    | Create table     | Target table for loading      |
| 3    | Create stage     | Connect S3 and Snowflake      |
| 4    | COPY INTO        | Load data into table          |
| 5    | LIST @stage      | Verify staged files           |
| 6    | SELECT           | Validate loaded data          |
| 7    | Snowpipe         | Automate continuous ingestion |

---

> ✅ **Tip:** Always validate files before loading, monitor your `COPY_HISTORY`, and consider using **Snowpipe** for automation with minimal manual effort.
