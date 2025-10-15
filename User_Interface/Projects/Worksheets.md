# 🧾 Worksheet in Snowflake (Snowsight)

---

## 🧭 Overview

The **Worksheet** in **Snowflake Snowsight** is an **interactive SQL editor** that allows you to:
- Write and execute **SQL queries**
- Explore and manage **databases, tables, and views**
- Create **objects** such as schemas, stages, streams, and tasks
- View and export **query results**
- **Save**, **share**, and **organize** your work within **projects**

Worksheets are a key component of the **Snowflake UI (Snowsight)** — providing a web-based environment for developing and testing SQL and Snowpark code without needing external tools like VS Code or DBeaver.

---

## 🧱 Components of a Worksheet

| Component | Description | Example |
|------------|--------------|----------|
| **Query Editor** | Area to write and execute SQL statements | `SELECT * FROM EMPLOYEES;` |
| **Database Context** | Choose database, schema, and warehouse to run queries | `USE DATABASE SALES_DB;` |
| **Result Pane** | Displays query results (tables, charts, JSON) | Output of SQL queries |
| **History Tab** | Shows previously executed queries and results | Review and rerun queries |
| **Snippets** | Reusable SQL code fragments | `SELECT COUNT(*) FROM <table>` |
| **Sharing Options** | Share worksheet with other users or roles | Team collaboration |

---

## 🧩 1. Creating a Worksheet

### 📍 Steps:
1. Log in to **Snowsight UI** → [https://app.snowflake.com](https://app.snowflake.com)
2. Navigate to the **Projects** or **Worksheets** tab.
3. Click **➕ Create Worksheet**.
4. Assign a **name** (e.g., “Sales Analysis Q4”).
5. Select the **Warehouse**, **Database**, and **Schema**.

---

## ⚙️ 2. Setting Context in a Worksheet

Before executing queries, always define your environment:

```sql
USE ROLE SYSADMIN;
USE WAREHOUSE COMPUTE_WH;
USE DATABASE SALES_DB;
USE SCHEMA PUBLIC;
````

This ensures queries execute in the correct compute and data context.

---

## 🧮 3. Running Queries

### 🧠 Example 1: Simple SELECT Query

```sql
SELECT FIRST_NAME, LAST_NAME, SALARY
FROM EMPLOYEES
WHERE DEPARTMENT = 'SALES';
```

### 🧠 Example 2: Aggregate Query

```sql
SELECT REGION, SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM ORDERS
GROUP BY REGION
ORDER BY TOTAL_SALES DESC;
```

### 🧠 Example 3: Create Table

```sql
CREATE OR REPLACE TABLE CUSTOMER_SALES AS
SELECT CUSTOMER_ID, SUM(SALES_AMOUNT) AS TOTAL
FROM SALES
GROUP BY CUSTOMER_ID;
```

---

## 📊 4. Viewing and Exporting Results

| Feature              | Description                                       |
| -------------------- | ------------------------------------------------- |
| **Table View**       | Displays tabular results                          |
| **Chart View**       | Visualize query output as bar/line/pie charts     |
| **Export Options**   | Download results as **CSV**, **JSON**, or **TSV** |
| **Filter/Search**    | Quickly find data in large result sets            |
| **Save Result View** | Save output for dashboard or future use           |

---

## 🧠 5. Query History in Worksheet

You can view **previously executed queries** directly within the worksheet.

```sql
-- Example of checking your session queries
SHOW QUERIES IN SESSION;
```

> The History tab lets you:
>
> * Re-run queries
> * View execution time and cost
> * Inspect query plans (profiling and performance tuning)

---

## 🧩 6. Using Variables in Worksheets

Variables make SQL more dynamic.

```sql
-- Define and use variables
SET region = 'WEST';
SELECT * FROM SALES WHERE REGION = $region;
```

### 🧠 Example

```sql
SET min_sales = 1000;
SELECT * FROM SALES WHERE SALES_AMOUNT > $min_sales;
```

---

## 🧰 7. Multiple Statements and Execution Modes

You can run:

* **Single statements** (`Ctrl + Enter`)
* **All statements in worksheet** (`Ctrl + Shift + Enter`)
* **Selected text** (highlight + run)

### Example

```sql
USE DATABASE SALES_DB;
SELECT COUNT(*) FROM ORDERS;
SELECT MAX(SALES_AMOUNT) FROM ORDERS;
```

---

## 🔄 8. Saving and Sharing Worksheets

| Action                  | Description                           |
| ----------------------- | ------------------------------------- |
| **Save Worksheet**      | Store for future use within a project |
| **Rename Worksheet**    | Change title for clarity              |
| **Share Worksheet**     | Collaborate with teammates or roles   |
| **Duplicate Worksheet** | Create a copy for testing variations  |

You can share a worksheet by clicking **“Share” → Select Users or Roles**.

---

## 🧮 9. Integration with Snowflake Objects

Worksheets allow you to **create and manage** all types of Snowflake objects:

| Object Type  | Example SQL                                       |
| ------------ | ------------------------------------------------- |
| **Database** | `CREATE DATABASE SALES_DB;`                       |
| **Schema**   | `CREATE SCHEMA REPORTING;`                        |
| **Table**    | `CREATE TABLE EMPLOYEE (ID INT, NAME STRING);`    |
| **Stage**    | `CREATE STAGE my_s3_stage URL='s3://mybucket/';`  |
| **Stream**   | `CREATE STREAM order_stream ON TABLE orders;`     |
| **Task**     | `CREATE TASK daily_task SCHEDULE='1 DAY' AS ...;` |

---

## 📑 10. Snippets and Templates

Snowflake provides **built-in SQL snippets** for:

* Data loading (`COPY INTO`)
* Data transformation (`MERGE`, `UPDATE`)
* Analytics (`GROUP BY`, `ROLLUP`)
* Security (`CREATE ROLE`, `GRANT`)

Example:

```sql
-- Template for data load
COPY INTO SALES
FROM @MY_STAGE/sales_data.csv
FILE_FORMAT = (TYPE = CSV FIELD_OPTIONALLY_ENCLOSED_BY='"');
```

---

## 🧭 11. Worksheet Settings

| Setting               | Description                           |
| --------------------- | ------------------------------------- |
| **Warehouse**         | Select compute engine to run queries  |
| **Role**              | Choose role (e.g., SYSADMIN, ANALYST) |
| **Database & Schema** | Define current context                |
| **Query Timeout**     | Set auto-cancel duration              |
| **Auto-Save**         | Automatically save worksheet changes  |
| **Dark/Light Theme**  | Personalize interface style           |

---

## 🧩 12. Example End-to-End Workflow

```sql
-- Step 1: Set Context
USE ROLE SYSADMIN;
USE WAREHOUSE COMPUTE_WH;
USE DATABASE SALES_DB;
USE SCHEMA RAW;

-- Step 2: Load Data
COPY INTO RAW.ORDERS
FROM @my_s3_stage/orders.csv
FILE_FORMAT = (TYPE = CSV SKIP_HEADER=1);

-- Step 3: Transform
CREATE OR REPLACE TABLE ANALYTICS.SALES_SUMMARY AS
SELECT REGION, SUM(SALES_AMOUNT) AS TOTAL
FROM RAW.ORDERS
GROUP BY REGION;

-- Step 4: Verify
SELECT * FROM ANALYTICS.SALES_SUMMARY;
```

---

## 🧾 13. Best Practices

| Practice                | Recommendation                                   |
| ----------------------- | ------------------------------------------------ |
| ✅ Use `USE` statements  | Always set role, warehouse, database, and schema |
| 💾 Save often           | Prevent accidental loss of queries               |
| 🧹 Clean up temp tables | Drop unused temporary tables after testing       |
| 🔍 Check history        | Review performance and cost                      |
| 📊 Visualize results    | Use charts for insights                          |
| 🧠 Organize             | Group worksheets under logical projects          |

