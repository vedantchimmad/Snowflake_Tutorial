# 🧩 SQL Commands Used in Snowflake

---

## 🧭 Overview

Snowflake supports a rich set of **SQL commands** for managing data, users, roles, warehouses, and more.  
These commands are grouped into several categories such as **Data Definition (DDL)**, **Data Manipulation (DML)**, **Data Control (DCL)**, and **Utility/Administrative** commands.

> 💡 Snowflake SQL syntax is **ANSI-compliant**, with cloud-specific enhancements for scalability and performance.

---

## 🧱 1️⃣ Data Definition Language (DDL)

Used to **define and manage database objects** like databases, schemas, tables, stages, and warehouses.

| Command | Description | Example |
|----------|--------------|----------|
| `CREATE DATABASE` | Create a new database | `CREATE DATABASE SALES_DB;` |
| `CREATE SCHEMA` | Create a new schema | `CREATE SCHEMA SALES_SCHEMA;` |
| `CREATE TABLE` | Create a new table | `CREATE TABLE EMP(ID INT, NAME STRING);` |
| `CREATE OR REPLACE` | Create or overwrite an object | `CREATE OR REPLACE TABLE PRODUCTS (...);` |
| `ALTER TABLE` | Modify existing table structure | `ALTER TABLE EMP ADD COLUMN DEPT STRING;` |
| `DROP TABLE` | Delete a table | `DROP TABLE EMP;` |
| `CREATE VIEW` | Create a logical view | `CREATE VIEW EMP_VIEW AS SELECT * FROM EMP;` |
| `CREATE STAGE` | Create stage for data loading | `CREATE STAGE MY_STAGE URL='s3://mybucket/';` |
| `CREATE FILE FORMAT` | Define how to parse files | `CREATE FILE FORMAT CSV_FORMAT TYPE='CSV';` |
| `CREATE PIPE` | Define Snowpipe for continuous loading | `CREATE PIPE SALES_PIPE AS COPY INTO SALES FROM @STAGE;` |
| `CREATE WAREHOUSE` | Create compute cluster | `CREATE WAREHOUSE COMPUTE_WH SIZE='SMALL';` |
| `ALTER WAREHOUSE` | Modify warehouse properties | `ALTER WAREHOUSE COMPUTE_WH RESUME;` |
| `DROP DATABASE` | Delete a database | `DROP DATABASE SALES_DB;` |

---

## 🧮 2️⃣ Data Manipulation Language (DML)

Used to **insert, update, delete, and query data**.

| Command | Description | Example |
|----------|-------------|----------|
| `INSERT` | Insert records into a table | `INSERT INTO EMP VALUES (1, 'John', 'HR');` |
| `UPDATE` | Update existing records | `UPDATE EMP SET DEPT='IT' WHERE ID=1;` |
| `DELETE` | Delete records from a table | `DELETE FROM EMP WHERE ID=2;` |
| `SELECT` | Retrieve records from one or more tables | `SELECT * FROM EMP;` |
| `MERGE` | Upsert (insert/update) data | `MERGE INTO TARGET USING SOURCE ON TARGET.ID=SOURCE.ID WHEN MATCHED THEN UPDATE SET ...;` |
| `COPY INTO` | Load data from stage to table | `COPY INTO SALES FROM @STAGE FILE_FORMAT=(TYPE='CSV');` |
| `UNLOAD (COPY INTO @STAGE)` | Export data to external location | `COPY INTO @STAGE/data.csv FROM SALES;` |
| `TRUNCATE` | Remove all rows but keep structure | `TRUNCATE TABLE SALES;` |

---

## 🔐 3️⃣ Data Control Language (DCL)

Used to **manage access control and privileges**.

| Command | Description | Example |
|----------|-------------|----------|
| `CREATE ROLE` | Create a new role | `CREATE ROLE ANALYST;` |
| `GRANT ROLE` | Assign a role to a user | `GRANT ROLE ANALYST TO USER JOHN;` |
| `REVOKE ROLE` | Remove a role from a user | `REVOKE ROLE ANALYST FROM USER JOHN;` |
| `GRANT` | Grant privileges on objects | `GRANT SELECT ON TABLE EMP TO ROLE ANALYST;` |
| `REVOKE` | Revoke privileges | `REVOKE SELECT ON TABLE EMP FROM ROLE ANALYST;` |
| `CREATE USER` | Create a user account | `CREATE USER JOHN PASSWORD='Abc@123';` |
| `ALTER USER` | Modify user attributes | `ALTER USER JOHN SET DEFAULT_ROLE=ANALYST;` |
| `DROP USER` | Remove a user | `DROP USER JOHN;` |
| `SHOW ROLES` | List all available roles | `SHOW ROLES;` |
| `SHOW GRANTS` | View privileges assigned | `SHOW GRANTS TO ROLE ANALYST;` |

---

## ⚙️ 4️⃣ Warehouse Management Commands

Used to manage compute resources (virtual warehouses).

| Command | Description | Example |
|----------|-------------|----------|
| `CREATE WAREHOUSE` | Create new compute cluster | `CREATE WAREHOUSE ANALYTICS_WH SIZE='MEDIUM';` |
| `ALTER WAREHOUSE` | Modify or start/stop warehouse | `ALTER WAREHOUSE ANALYTICS_WH SUSPEND;` |
| `DROP WAREHOUSE` | Delete warehouse | `DROP WAREHOUSE ANALYTICS_WH;` |
| `SHOW WAREHOUSES` | List all warehouses | `SHOW WAREHOUSES;` |
| `USE WAREHOUSE` | Switch to a specific warehouse | `USE WAREHOUSE ANALYTICS_WH;` |

---

## 🧠 5️⃣ Utility & Monitoring Commands

Used for **querying metadata, monitoring performance**, and **data lifecycle** operations.

| Command | Description | Example |
|----------|-------------|----------|
| `SHOW TABLES` | List all tables | `SHOW TABLES IN SCHEMA SALES_SCHEMA;` |
| `DESCRIBE TABLE` | Show table structure | `DESCRIBE TABLE EMP;` |
| `SHOW PIPES` | List Snowpipes | `SHOW PIPES;` |
| `SHOW STAGES` | List data stages | `SHOW STAGES;` |
| `SHOW FILE FORMATS` | List file formats | `SHOW FILE FORMATS;` |
| `SHOW DATABASES` | List databases | `SHOW DATABASES;` |
| `SHOW SCHEMAS` | List schemas | `SHOW SCHEMAS;` |
| `SHOW USERS` | List users | `SHOW USERS;` |
| `SHOW WAREHOUSES` | List all warehouses | `SHOW WAREHOUSES;` |
| `USE DATABASE` | Switch active database | `USE DATABASE SALES_DB;` |
| `USE ROLE` | Activate a specific role | `USE ROLE SYSADMIN;` |
| `USE SCHEMA` | Switch active schema | `USE SCHEMA SALES_SCHEMA;` |
| `USE WAREHOUSE` | Activate a specific warehouse | `USE WAREHOUSE COMPUTE_WH;` |

---

## 📈 6️⃣ Query History & Performance Commands

| Command | Description | Example |
|----------|-------------|----------|
| `QUERY_HISTORY` | View query execution logs | `SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY;` |
| `COPY_HISTORY` | View data load history | `SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.COPY_HISTORY;` |
| `PIPE_USAGE_HISTORY` | Track Snowpipe activity | `SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.PIPE_USAGE_HISTORY;` |
| `WAREHOUSE_LOAD_HISTORY` | Monitor warehouse usage | `SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY;` |
| `LOGIN_HISTORY` | Audit user logins | `SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY;` |

---

## 📚 7️⃣ Semi-Structured Data Commands

| Command | Description | Example |
|----------|-------------|----------|
| `FLATTEN()` | Expand nested JSON/ARRAY fields | `SELECT VALUE FROM TABLE(FLATTEN(INPUT => JSON_DATA));` |
| `PARSE_JSON()` | Parse string to JSON format | `SELECT PARSE_JSON(COLUMN_NAME) FROM TABLE;` |
| `TO_VARIANT()` | Convert value into Snowflake VARIANT | `SELECT TO_VARIANT(DATA) FROM TABLE;` |

---

## ⚡ 8️⃣ Time Travel & Cloning Commands

| Command | Description | Example |
|----------|-------------|----------|
| `CREATE TABLE CLONE` | Create copy of table using zero-copy clone | `CREATE TABLE SALES_CLONE CLONE SALES;` |
| `UNDROP TABLE` | Recover dropped table | `UNDROP TABLE SALES;` |
| `SELECT ... AT (TIMESTAMP => ...)` | Query historical data | `SELECT * FROM SALES AT (TIMESTAMP => '2024-12-31 23:59:00');` |

---

## 🧩 Summary

| Category | Key Commands |
|-----------|---------------|
| **DDL** | CREATE, ALTER, DROP |
| **DML** | SELECT, INSERT, UPDATE, DELETE, MERGE, COPY INTO |
| **DCL** | GRANT, REVOKE, CREATE ROLE, CREATE USER |
| **Warehouse Management** | CREATE/ALTER/DROP WAREHOUSE |
| **Monitoring** | SHOW, DESCRIBE, USE |
| **Advanced Features** | CLONE, UNDROP, TIME TRAVEL, PIPE |

---
