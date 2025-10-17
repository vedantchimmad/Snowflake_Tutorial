# ☁️ Loading Data from **Azure** into Snowflake

---

## 🧭 Overview

Snowflake provides seamless integration with **Microsoft Azure** for data ingestion.  
You can easily load structured or semi-structured data (CSV, JSON, Parquet, Avro, ORC) from **Azure Blob Storage** or **Azure Data Lake Storage (ADLS)** directly into **Snowflake tables**.

> 💡 Common Pattern:  
> Azure Storage → Snowflake Stage → COPY INTO → Target Table

---

## 🧩 Steps to Load Data from Azure into Snowflake

| Step | Description |
|------|--------------|
| 1️⃣ | Upload data to **Azure Blob Storage** or **ADLS** |
| 2️⃣ | Create a **Snowflake table** |
| 3️⃣ | Create an **external stage** linked to Azure |
| 4️⃣ | Load data using **COPY INTO** |
| 5️⃣ | Verify loaded data |
| 6️⃣ | (Optional) Automate with **Snowpipe** for continuous ingestion |

---

## ☁️ 1. Upload Data to Azure Storage

Upload files to your Azure Blob Storage container (e.g., CSV, JSON, Parquet).

**Example structure:**
```

azure://mycontainer/data/
├── sales_2025_01.csv
├── sales_2025_02.csv
└── sales_2025_03.csv

````

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

## 🧩 3. Create a Storage Integration (for Secure Access)

Snowflake accesses Azure data through a **Storage Integration**, which securely manages credentials.

```sql
CREATE OR REPLACE STORAGE INTEGRATION azure_int
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = AZURE
ENABLED = TRUE
AZURE_TENANT_ID = 'your-azure-tenant-id'
STORAGE_ALLOWED_LOCATIONS = ('azure://mycontainer/data/');
```

### View Integration Details:

```sql
DESC INTEGRATION azure_int;
```

> The output includes:
>
> * `AZURE_CONSENT_URL` → used by Azure admin to authorize Snowflake
> * `AZURE_MULTI_TENANT_APP_NAME` → Snowflake's app registration name

✅ **Step:** Azure Admin must open the `AZURE_CONSENT_URL` to grant Snowflake access.

---

## 🧩 4. Create a Stage in Snowflake

```sql
CREATE OR REPLACE STAGE azure_stage
URL = 'azure://mycontainer/data/'
STORAGE_INTEGRATION = azure_int
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);
```

### Verify Stage Connection

```sql
LIST @azure_stage;
```

---

## 🧩 5. Load Data into Snowflake

### 🔹 Load All Files from the Stage

```sql
COPY INTO SALES
FROM @azure_stage
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1)
ON_ERROR = 'CONTINUE';
```

### 🔹 Load Specific File

```sql
COPY INTO SALES
FROM @azure_stage/sales_2025_01.csv
FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);
```

---

## 🧩 6. Verify Loaded Data

```sql
SELECT * FROM SALES LIMIT 10;
```

### Check Load History

```sql
SELECT * FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(TABLE_NAME=>'SALES'));
```

---

## 🧩 7. File Format Configuration

```sql
CREATE OR REPLACE FILE FORMAT azure_csv_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1
NULL_IF = ('NULL', 'null')
TRIM_SPACE = TRUE;
```

---

## 🧩 8. COPY Command — Key Options

| Option            | Description                         | Example                           |
| ----------------- | ----------------------------------- | --------------------------------- |
| `ON_ERROR`        | Skip or abort on error              | `'CONTINUE'`, `'ABORT_STATEMENT'` |
| `FORCE`           | Reload files even if already loaded | `TRUE`                            |
| `PURGE`           | Delete files after load             | `TRUE`                            |
| `VALIDATION_MODE` | Check data without loading          | `'RETURN_ERRORS'`                 |
| `PATTERN`         | Filter specific files               | `'.*2025_.*.csv'`                 |

Example:

```sql
COPY INTO SALES
FROM @azure_stage
FILE_FORMAT = (FORMAT_NAME = azure_csv_format)
ON_ERROR = 'CONTINUE'
FORCE = TRUE;
```

---

## 🧩 9. Continuous Loading with Snowpipe

**Snowpipe** automatically loads data from Azure as soon as files are uploaded.

### Step 1: Create a Pipe

```sql
CREATE OR REPLACE PIPE sales_pipe
AS
COPY INTO SALES
FROM @azure_stage
FILE_FORMAT = (TYPE='CSV' SKIP_HEADER=1)
ON_ERROR='CONTINUE';
```

### Step 2: Enable Azure Event Grid Notification

Configure your Azure Blob Storage container to publish **event notifications** to Snowflake’s Snowpipe endpoint.

> 🧠 Snowpipe supports event-driven, serverless data ingestion for near real-time pipelines.

---

## 🧩 10. Performance Optimization Tips

| Tip                                                            | Description |
| -------------------------------------------------------------- | ----------- |
| ✅ Use **compressed files** (e.g., `.gz`, `.parquet`)           |             |
| ✅ Combine small files into larger batches (100–250MB)          |             |
| ✅ Use **parallel COPY INTO** for faster loading                |             |
| ✅ Use **auto-suspend warehouses** to save credits              |             |
| ✅ Stage & transform data before merging into production tables |             |

---

## 🧩 11. Example End-to-End Script

```sql
-- Step 1: Create Integration
CREATE OR REPLACE STORAGE INTEGRATION azure_integration
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = AZURE
ENABLED = TRUE
AZURE_TENANT_ID = '72f988bf-86f1-41af-91ab-2d7cd011db47'
STORAGE_ALLOWED_LOCATIONS = ('azure://mycontainer/data/');

-- Step 2: Create Stage
CREATE OR REPLACE STAGE azure_stage
URL = 'azure://mycontainer/data/'
STORAGE_INTEGRATION = azure_integration
FILE_FORMAT = (TYPE='CSV' SKIP_HEADER=1);

-- Step 3: Create Table
CREATE OR REPLACE TABLE customer_data (
  id INT,
  name STRING,
  city STRING,
  country STRING
);

-- Step 4: Load Data
COPY INTO customer_data
FROM @azure_stage
ON_ERROR='CONTINUE';
```

---

## 🧩 12. Visual Data Flow

```
        ┌──────────────────────┐
        │  Azure Blob Storage  │
        │   (CSV / JSON files) │
        └─────────┬────────────┘
                  │
                  ▼
        ┌──────────────────────┐
        │  Snowflake Stage      │
        │ (Azure Integration)   │
        └─────────┬────────────┘
                  │
                  ▼
        ┌──────────────────────┐
        │  COPY INTO Table      │
        │   (Sales, Customer)   │
        └─────────┬────────────┘
                  │
                  ▼
        ┌──────────────────────┐
        │ Query & Transform     │
        │  (BI / Analytics)     │
        └──────────────────────┘
```

---

## 🧩 13. Validation & Monitoring

### Check Staged Files

```sql
LIST @azure_stage;
```

### Validate COPY Load

```sql
COPY INTO SALES
FROM @azure_stage
VALIDATION_MODE = 'RETURN_ERRORS';
```

### Monitor Load History

```sql
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.COPY_HISTORY
WHERE TABLE_NAME='SALES';
```

---

## 🧠 Summary Table

| Step | Command / Action   | Description                     |
| ---- | ------------------ | ------------------------------- |
| 1    | Upload to Azure    | Store files in Blob or ADLS     |
| 2    | Create table       | Define target Snowflake table   |
| 3    | Create integration | Secure Azure authentication     |
| 4    | Create stage       | Link Azure storage to Snowflake |
| 5    | COPY INTO          | Load data from stage            |
| 6    | LIST @stage        | Verify files                    |
| 7    | Snowpipe           | Automate continuous ingestion   |

---

> ✅ **Pro Tip:**
> Always test your COPY operation with `VALIDATION_MODE` before actual load, and use **Snowpipe** for automated, event-driven ingestion from Azure storage.

