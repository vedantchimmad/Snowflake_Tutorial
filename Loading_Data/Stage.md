# 🏗️ Stages in Snowflake

---

## 🧭 Overview

In **Snowflake**, a **Stage** is a **storage location** used to **store data files** (like CSV, JSON, Parquet, Avro) before loading them into tables or after unloading data from tables.

> 🧠 Stages act as **intermediate data storage zones**, where files are placed before being ingested (via `COPY INTO`) or extracted (via `COPY INTO @stage`).

---

## 🧩 Types of Stages in Snowflake

| Stage Type | Description | Example |
|-------------|--------------|----------|
| **User Stage** | Each Snowflake user automatically gets a personal stage for storing files | `@~` |
| **Table Stage** | Each table has its own stage, used to store data related to that table | `@%TABLE_NAME` |
| **Internal Named Stage** | A named stage created by users for flexible data management within Snowflake | `@MY_INTERNAL_STAGE` |
| **External Stage** | A stage that references **external cloud storage** like AWS S3, Azure Blob, or Google Cloud Storage | `@MY_EXTERNAL_STAGE` |

---

## 🧱 1️⃣ User Stage

- Every Snowflake user has a **default personal stage**.
- Can be used for uploading or unloading files.

```sql
-- List files in your personal stage
LIST @~;

-- Upload file to your personal stage (using SnowSQL)
PUT file://C:/data/sales.csv @~;

-- Load data from your personal stage into a table
COPY INTO SALES FROM @~ FILE_FORMAT=(TYPE='CSV');
````

> 📌 Path: `@~` refers to the **current user’s stage**.

---

## 🧱 2️⃣ Table Stage

* Each table in Snowflake has an **implicit stage**.
* Ideal for **storing data specifically related to that table**.

```sql
-- Upload data to the table stage
PUT file://C:/data/sales.csv @%SALES;

-- Load data from table stage
COPY INTO SALES FROM @%SALES FILE_FORMAT=(TYPE='CSV');

-- List files stored in the table stage
LIST @%SALES;
```

> 📌 Path: `@%TABLE_NAME` represents the table-specific stage.

---

## 🧱 3️⃣ Internal Named Stage

* A **user-created internal stage** inside Snowflake.
* Allows multiple users or processes to share a common data storage location.

```sql
-- Create a named internal stage
CREATE OR REPLACE STAGE SALES_INTERNAL_STAGE;

-- Upload files
PUT file://C:/data/sales.csv @SALES_INTERNAL_STAGE;

-- Load data into table
COPY INTO SALES FROM @SALES_INTERNAL_STAGE FILE_FORMAT=(TYPE='CSV');

-- View stage files
LIST @SALES_INTERNAL_STAGE;
```

> ✅ Best for **team collaboration** or **structured ETL pipelines**.

---

## 🧱 4️⃣ External Stage

* Connects Snowflake with **cloud storage** like:

    * AWS S3
    * Azure Blob Storage
    * Google Cloud Storage (GCS)

### 🔹 Example: AWS S3 Stage

```sql
CREATE OR REPLACE STAGE SALES_S3_STAGE
URL = 's3://mybucket/salesdata/'
CREDENTIALS = (AWS_KEY_ID='XXXXXXXX' AWS_SECRET_KEY='XXXXXXXX')
FILE_FORMAT = (TYPE='CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"');
```

### 🔹 Example: Azure Stage

```sql
CREATE OR REPLACE STAGE SALES_AZURE_STAGE
URL = 'azure://mycontainer.blob.core.windows.net/salesdata/'
CREDENTIALS = (AZURE_SAS_TOKEN='?sv=xxxxxx');
```

### 🔹 Example: GCS Stage

```sql
CREATE OR REPLACE STAGE SALES_GCS_STAGE
URL = 'gcs://mybucket/salesdata/'
CREDENTIALS = (GCP_KEYFILE='{
  "type": "service_account",
  "project_id": "myproject",
  ...
}');
```

### 📜 List and Query Files

```sql
LIST @SALES_S3_STAGE;
```

---

## ⚙️ Stage File Management Commands

| Command              | Description                      | Example                                   |
| -------------------- | -------------------------------- | ----------------------------------------- |
| `PUT`                | Upload files from local to stage | `PUT file://C:/data/sales.csv @MY_STAGE;` |
| `LIST`               | View files stored in a stage     | `LIST @MY_STAGE;`                         |
| `REMOVE`             | Delete files from a stage        | `REMOVE @MY_STAGE/sales.csv;`             |
| `COPY INTO <table>`  | Load data from stage to table    | `COPY INTO SALES FROM @MY_STAGE;`         |
| `COPY INTO @<stage>` | Unload data from table to stage  | `COPY INTO @MY_STAGE FROM SALES;`         |

---

## 🧩 File Format Association

You can link a **file format** to a stage to define how the data should be interpreted.

```sql
CREATE FILE FORMAT CSV_FORMAT
TYPE = 'CSV'
FIELD_OPTIONALLY_ENCLOSED_BY='"'
SKIP_HEADER = 1;

CREATE OR REPLACE STAGE SALES_STAGE
FILE_FORMAT = CSV_FORMAT;
```

> 🧠 Now Snowflake automatically applies `CSV_FORMAT` when loading or unloading data from this stage.

---

## 📊 Internal vs External Stages

| Feature         | Internal Stage                      | External Stage                        |
| --------------- | ----------------------------------- | ------------------------------------- |
| **Storage**     | Inside Snowflake                    | Cloud storage (S3, Azure, GCS)        |
| **Security**    | Managed by Snowflake                | Managed via cloud credentials         |
| **Cost**        | Billed as part of Snowflake storage | Billed in respective cloud provider   |
| **Performance** | Faster for small data               | Ideal for large enterprise data lakes |
| **Example**     | `@MY_INTERNAL_STAGE`                | `@MY_S3_STAGE`                        |

---

## ⚡ Example: End-to-End Data Load

```sql
-- Step 1: Create Table
CREATE OR REPLACE TABLE SALES(
  ORDER_ID INT,
  CUSTOMER_ID INT,
  REGION STRING,
  SALES_AMOUNT FLOAT
);

-- Step 2: Create Stage
CREATE OR REPLACE STAGE SALES_STAGE;

-- Step 3: Upload File
PUT file://C:/data/sales.csv @SALES_STAGE;

-- Step 4: Load Data
COPY INTO SALES
FROM @SALES_STAGE
FILE_FORMAT = (TYPE='CSV' SKIP_HEADER=1);
```

---

## 🧠 Best Practices

| Practice                          | Description                            |
| --------------------------------- | -------------------------------------- |
| Use **Named Internal Stages**     | For organized, reusable data storage   |
| Use **External Stages**           | For data lakes and shared environments |
| Always **define file formats**    | Avoid parsing errors                   |
| **Compress files**                | For faster upload and reduced cost     |
| **Remove processed files**        | Keep stage clean and efficient         |
| Enable **encryption**             | For secure data transfer               |
| Use **LIST** and **COPY_HISTORY** | For monitoring and debugging           |

---

## 🧩 Summary

| Type                     | Description                  | Example        |
| ------------------------ | ---------------------------- | -------------- |
| **User Stage**           | Personal stage for each user | `@~`           |
| **Table Stage**          | Stage specific to a table    | `@%TABLE_NAME` |
| **Internal Named Stage** | Custom internal stage        | `@MY_STAGE`    |
| **External Stage**       | Linked to cloud storage      | `@MY_S3_STAGE` |

> 💡 **Tip:** Stages are the backbone of data loading and unloading in Snowflake — mastering them ensures efficient and secure data flow across your pipelines.

