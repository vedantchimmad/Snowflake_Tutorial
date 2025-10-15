# 📊 Dashboards in Snowflake (Snowsight)

---

## 🧭 Overview

**Dashboards in Snowflake (Snowsight)** enable users to **visualize data directly within Snowflake** using SQL queries — without needing external BI tools like Power BI or Tableau.  

These dashboards are created from **worksheet query results** and provide:
- **Interactive visualizations**
- **Live data insights**
- **Role-based access control**
- **Real-time refreshes**

> 🧠 Snowflake dashboards are part of **Snowsight**, the modern web interface for SQL development, visualization, and collaboration.

---

## 🧩 Key Features

| Feature | Description |
|----------|--------------|
| 📊 **Visualizations** | Create bar, line, pie, and table charts |
| 🧠 **SQL-Based** | Build dashboards directly from SQL queries |
| 🔄 **Real-Time Data** | Reflects live data from Snowflake tables and views |
| 🧩 **Drag-and-Drop UI** | No-code visual creation |
| ⚙️ **Customizable Layout** | Resize, move, and arrange visual cards |
| 🔐 **Secure Access** | Controlled by Snowflake roles and permissions |
| 🕒 **Auto Refresh** | Automatically updates with latest data |
| 💾 **Shareable** | Easily share dashboards with team members |

---

## 🧱 Dashboard Architecture

```

+----------------------------------------------------+

| Snowsight UI                                           |                                   |
| ------------------------------------------------------ | --------------------------------- |
| 📊 Dashboard Layer                                     | Charts, Metrics, Widgets          |
| 🧮 Query Layer                                         | SQL Queries (from Worksheets)     |
| 💾 Data Layer                                          | Tables, Views, Materialized Views |
| 🔐 Security Layer                                      | Roles, Permissions, Policies      |
| +----------------------------------------------------+ |                                   |

````

---

## 🧩 1. Creating a Dashboard

### 🪜 Steps:
1. Log in to **Snowsight** → [https://app.snowflake.com](https://app.snowflake.com)
2. Go to **Projects → Dashboards**
3. Click **➕ Create Dashboard**
4. Enter a **Name** (e.g., “Sales KPI Dashboard”)
5. Add new **visualizations (charts)** by linking SQL queries

---

## 🧮 2. Building Visualizations

### 🔹 Step 1: Add a Query
You can either:
- Use a query from an existing **Worksheet**
- Or create a **new SQL query** inside the dashboard editor

Example SQL:
```sql
SELECT REGION, SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM SALES
GROUP BY REGION;
````

### 🔹 Step 2: Create Visualization

Click **“Visualize” → Choose Chart Type:**

* 📈 Line Chart — For trends
* 📊 Bar Chart — For comparisons
* 🥧 Pie Chart — For proportions
* 📋 Table — For tabular display
* 📅 Time Series — For data over time

---

## 🎨 3. Visualization Customization

| Option                  | Description                                 |
| ----------------------- | ------------------------------------------- |
| **Chart Type**          | Change visualization (bar, line, pie, etc.) |
| **X / Y Axis**          | Define axes columns                         |
| **Color**               | Assign color grouping (e.g., REGION)        |
| **Labels**              | Enable or disable value labels              |
| **Sort Order**          | Sort results ascending/descending           |
| **Limit Rows**          | Restrict large datasets                     |
| **Title / Description** | Add informative titles for clarity          |

Example:

```sql
SELECT PRODUCT_CATEGORY, SUM(REVENUE) AS TOTAL_REVENUE
FROM SALES
GROUP BY PRODUCT_CATEGORY
ORDER BY TOTAL_REVENUE DESC;
```

Visualization Type → 📊 **Bar Chart**

---

## 🧠 4. Example Dashboard: Sales Overview

| Visualization            | SQL Query                                                                                                       | Description                      |
| ------------------------ | --------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| 💰 Total Sales by Region | `SELECT REGION, SUM(SALES_AMOUNT) AS TOTAL FROM SALES GROUP BY REGION;`                                         | Bar chart comparing sales        |
| 🕒 Monthly Sales Trend   | `SELECT DATE_TRUNC('month', ORDER_DATE) AS MONTH, SUM(SALES_AMOUNT) FROM SALES GROUP BY MONTH;`                 | Line chart showing monthly trend |
| 🧾 Top 5 Products        | `SELECT PRODUCT_NAME, SUM(SALES_AMOUNT) AS TOTAL FROM SALES GROUP BY PRODUCT_NAME ORDER BY TOTAL DESC LIMIT 5;` | Horizontal bar chart             |
| 🧮 Total Orders Count    | `SELECT COUNT(*) FROM ORDERS;`                                                                                  | Single metric card               |

---

## ⚙️ 5. Dashboard Settings

| Setting              | Description                               |
| -------------------- | ----------------------------------------- |
| **Refresh Interval** | Set auto-refresh (e.g., every 15 minutes) |
| **Warehouse**        | Select compute resource for queries       |
| **Role**             | Choose security role for execution        |
| **Theme**            | Switch between light/dark mode            |
| **Layout**           | Arrange visualizations in grid            |
| **Description**      | Add notes for context                     |

---

## 🧾 6. Using Filters in Dashboards

You can add **interactive filters** to make dashboards dynamic.

### Example:

```sql
SELECT REGION, SUM(SALES_AMOUNT) AS TOTAL
FROM SALES
WHERE ORDER_DATE BETWEEN :start_date AND :end_date
GROUP BY REGION;
```

### Filter Widgets

| Type             | Description                            |
| ---------------- | -------------------------------------- |
| 📅 Date Picker   | Filter data by date range              |
| 🧾 Dropdown      | Select region, category, etc.          |
| 🔢 Numeric Input | Filter by thresholds (e.g., min sales) |

---

## 🧩 7. Sharing Dashboards

### Sharing Options:

| Option             | Description                           |
| ------------------ | ------------------------------------- |
| **User Access**    | Share with specific users             |
| **Role-Based**     | Grant access to roles (e.g., ANALYST) |
| **Read-Only View** | Share view-only dashboards            |
| **Edit Access**    | Collaborate with edit privileges      |

Example SQL for granting access:

```sql
GRANT USAGE ON DATABASE ANALYTICS_DB TO ROLE analyst;
GRANT SELECT ON TABLE SALES TO ROLE analyst;
```

---

## 🧮 8. Scheduling and Automation

| Feature                   | Description                                       |
| ------------------------- | ------------------------------------------------- |
| **Auto Refresh**          | Dashboards refresh at fixed intervals             |
| **Task Integration**      | Combine with Snowflake Tasks for periodic updates |
| **Email Alerts (Future)** | Integration with alerting services                |
| **Version History**       | Save dashboard snapshots over time                |

---

## 🧰 9. Example: Sales KPI Dashboard Layout

```
------------------------------------------------------------
| 💰 Total Sales | 🛒 Total Orders | 📦 Top 5 Products     |
------------------------------------------------------------
| 📈 Monthly Sales Trend        | 📊 Regional Performance |
------------------------------------------------------------
| 🧾 Product Category Split     | 🕒 Last Updated: [auto]  |
------------------------------------------------------------
```

---

## 🧠 10. Best Practices for Data Engineers

| Practice                              | Recommendation                         |
| ------------------------------------- | -------------------------------------- |
| ✅ Use **views** instead of raw tables | For performance and reusability        |
| ⚙️ Use **materialized views**         | For heavy aggregations                 |
| 🧩 Parameterize SQL queries           | For filter-based dashboards            |
| 🔄 Set **refresh schedules**          | Keep dashboards up-to-date             |
| 🔐 Control access by roles            | Protect sensitive data                 |
| 📊 Optimize warehouse size            | Avoid over-provisioning                |
| 🧠 Save query results                 | Cache large results for faster reloads |

---

## 🧮 11. Example End-to-End Dashboard Setup

```sql
-- Step 1: Create Database and Schema
CREATE DATABASE ANALYTICS_DB;
CREATE SCHEMA DASHBOARD;

-- Step 2: Create Summary View
CREATE OR REPLACE VIEW DASHBOARD.SALES_SUMMARY AS
SELECT 
    REGION,
    DATE_TRUNC('month', ORDER_DATE) AS MONTH,
    SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM RAW.SALES
GROUP BY REGION, MONTH;

-- Step 3: Query for Dashboard
SELECT REGION, MONTH, TOTAL_SALES
FROM DASHBOARD.SALES_SUMMARY
ORDER BY MONTH;
```

Then, in Snowsight:

* Add a **Line Chart** → X = MONTH, Y = TOTAL\_SALES, Color = REGION
* Add filters for `REGION` or `MONTH`

---

## 🧩 Summary

| Concept         | Description                                 |
| --------------- | ------------------------------------------- |
| **Tool**        | Snowsight (Snowflake UI)                    |
| **Purpose**     | Build live, interactive dashboards from SQL |
| **Data Source** | Snowflake tables, views, or queries         |
| **Access**      | Role-based and secure                       |
| **Best For**    | Real-time analytics, KPIs, data exploration |

> 💡 **Tip:** Dashboards in Snowflake empower **data engineers and analysts** to go from **raw SQL to interactive insights** — all inside one platform, without exporting data anywhere.
```
