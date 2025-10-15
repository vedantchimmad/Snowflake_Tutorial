# ❄️ Snowflake User Interface (UI) Guide

---

## 🧭 Overview

The **Snowflake User Interface (UI)** — also known as the **Snowflake Web Console** or **Snowsight** — is a **web-based graphical interface** that allows users to manage databases, run SQL queries, monitor performance, and manage security without command-line tools.

It provides **intuitive navigation**, **powerful SQL editing**, and **visual analytics** for data engineers, analysts, and administrators.

---

## 🧱 Snowflake UI Components

| Section | Description | Icon / Purpose |
|----------|--------------|----------------|
| **🏠 Home Dashboard** | Displays recent queries, worksheets, and warehouses. | Quick navigation panel |
| **🧮 Worksheets** | Used to write, execute, and share SQL queries. | Interactive SQL editor |
| **🗂️ Databases** | View, create, and manage databases, schemas, tables, and views. | Data management section |
| **⚙️ Warehouses** | Manage compute clusters (virtual warehouses). | Start/stop and monitor compute usage |
| **📈 Query History** | Track execution time, status, and performance metrics of queries. | Query log with filters |
| **👤 Account / Admin** | Manage users, roles, integrations, and account settings. | Security and access control |
| **📊 Dashboards** | Create and visualize datasets using Snowsight visualizations. | BI-style charts and tables |
| **🧩 Marketplace** | Access third-party datasets and Snowflake data applications. | Data sharing hub |
| **🧰 Data Explorer** | Browse data objects across databases and schemas. | Object-level navigation |

---

## 🧮 1. Worksheets (SQL Editor)

### ✨ Features
- Syntax highlighting and autocomplete
- Multi-tab support for multiple queries
- Execute full or partial queries
- Query result visualization (tables, charts)
- Export query results to CSV, JSON, or Excel

### 🧑‍💻 Example Query
```sql
USE DATABASE SALES_DB;
USE SCHEMA PUBLIC;

SELECT REGION, SUM(AMOUNT) AS TOTAL_SALES
FROM ORDERS
GROUP BY REGION
ORDER BY TOTAL_SALES DESC;
````

---

## 🗂️ 2. Databases and Schemas

### 📦 Databases Section

* View all databases and their schemas
* Create and manage tables, views, and stages
* View metadata and storage details

### 🧱 Common Actions

| Action          | Description                 | Example                                         |
| --------------- | --------------------------- | ----------------------------------------------- |
| Create Database | Add a new logical container | `CREATE DATABASE analytics_db;`                 |
| Create Schema   | Group objects logically     | `CREATE SCHEMA staging;`                        |
| Create Table    | Define data structure       | `CREATE TABLE customers (id INT, name STRING);` |

---

## ⚙️ 3. Warehouses (Compute)

### 🧰 Overview

* Each **warehouse** is a compute cluster used for executing SQL statements.
* You can create, start, suspend, or resize warehouses directly from the UI.

### ⚡ Warehouse Options

| Option             | Description                                   |
| ------------------ | --------------------------------------------- |
| **Name**           | Identifier for the warehouse                  |
| **Size**           | Determines compute power (X-Small → 6X-Large) |
| **Auto Suspend**   | Automatically shuts down after inactivity     |
| **Auto Resume**    | Starts automatically when a query is run      |
| **Scaling Policy** | Defines behavior for multi-cluster scaling    |

---

## 📈 4. Query History

### 🔍 Features

* View past query executions
* Analyze performance metrics (execution time, bytes scanned)
* Identify bottlenecks using detailed query profiles
* Filter by user, time, or warehouse

### 🔹 Example Use

| Column             | Meaning                          |
| ------------------ | -------------------------------- |
| **Query ID**       | Unique identifier for each query |
| **Execution Time** | Total time taken                 |
| **Status**         | Success, Failed, or Queued       |
| **Bytes Scanned**  | Data volume processed            |

---

## 🔐 5. Admin and Security Management

### 👥 Roles and Permissions

* Manage **users**, **roles**, and **grants**
* Assign permissions for databases, schemas, and warehouses

### 🧾 Example

```sql
CREATE ROLE data_engineer;
GRANT USAGE ON WAREHOUSE compute_wh TO ROLE data_engineer;
GRANT ROLE data_engineer TO USER vedant;
```

### 🧱 Account Settings

* Configure **network policies**, **SSO integration**, **resource monitors**, and **account parameters**.

---

## 📊 6. Dashboards and Visualizations (Snowsight)

### 📉 Features

* Create **interactive dashboards** using query results
* Visualize data using **charts, bars, and pie graphs**
* Schedule automatic data refresh
* Share dashboards with roles or users

### 🔹 Example Visualization Query

```sql
SELECT PRODUCT_CATEGORY, SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM SALES
GROUP BY PRODUCT_CATEGORY;
```

Then choose a **Bar Chart** in Snowsight visualization options.

---

## 🧩 7. Snowflake Marketplace

### 🌍 Overview

* Explore **third-party datasets** and **data services**
* Subscribe and use shared datasets directly within your account

### 🔹 Example Use Cases

| Type         | Example                                  |
| ------------ | ---------------------------------------- |
| Public Data  | COVID-19 datasets, financial market data |
| Vendor Data  | Weather data, demographic data           |
| Partner Apps | Data exchange via secure share           |

---

## 🧰 8. Data Explorer

### 🔎 Features

* Browse all **databases, schemas, and tables**
* Preview data samples
* View object details (columns, data types, constraints)
* Easily navigate to SQL worksheets for querying

---

## 🧩 UI Navigation Flow

```
Home Dashboard
   ├── Worksheets → Run SQL Queries
   ├── Databases → Explore Tables / Views
   ├── Warehouses → Manage Compute
   ├── Query History → Monitor Performance
   ├── Dashboards → Visualize Results
   └── Admin → Manage Users / Roles / Settings
```

---

## 🧠 Tips for Data Engineers

* Use **Snowsight worksheets** for multi-tab SQL development.
* Enable **Auto Suspend** to save compute cost.
* Regularly check **Query History** for performance optimization.
* Manage **roles** carefully for secure data access.
* Create **dashboards** for quick KPI monitoring.

---
