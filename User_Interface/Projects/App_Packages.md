# 📦 Snowflake Native App Packages

---

## 🧭 Overview

**Snowflake Native App Packages** allow developers and data engineers to build, distribute, and manage full-fledged **data applications** directly within Snowflake’s ecosystem — **without moving data out** of the platform.  

These applications can include:
- SQL scripts  
- Streamlit dashboards  
- Stored procedures  
- UDFs and UDTFs  
- Custom logic and UI elements  

> 🧠 Think of Snowflake Native Apps as “Snowflake’s App Store for Data” — enabling secure, governed sharing and monetization of analytics applications.

---

## 🧩 Key Components

| Component | Description |
|------------|--------------|
| 📦 **App Package** | A bundle containing all assets, code, and metadata |
| 💾 **App Instance** | A deployed copy of the app package in a consumer account |
| 🧱 **Artifacts** | SQL scripts, Streamlit apps, stored procedures, and more |
| 🧮 **Setup Script** | Runs during installation to configure resources |
| 🔒 **Permissions** | Define secure access to objects inside the app |
| 📊 **Streamlit Integration** | Allows embedded dashboards and visual UIs |
| 🧰 **Marketplace Publishing** | Share or sell apps via Snowflake Marketplace |

---

## 🧱 Architecture Diagram

```

+--------------------------------------------------------+

| Snowflake Platform                                         |                 |           |
| ---------------------------------------------------------- | --------------- | --------- |
| 🧩 Native App Package                                      |                 |           |
| ├── setup.sql                                              |                 |           |
| ├── main.py                                                |                 |           |
| ├── manifest.yml                                           |                 |           |
| ├── streamlit\_app/                                        |                 |           |
| ├── stored\_procedures/                                    |                 |           |
| └── resources/                                             |                 |           |
| --------------------------------------------------------   |                 |           |
| 🔐 Secure Data Sharing                                     | 💻 Streamlit UI | SQL Logic |
| +--------------------------------------------------------+ |                 |           |

````

---

## ⚙️ Structure of an App Package

| File/Folder | Description |
|--------------|-------------|
| `manifest.yml` | Metadata defining app version, dependencies, and setup |
| `setup.sql` | SQL statements for initializing schemas, roles, and objects |
| `app/` | Contains main logic (SQL, Python, JS, etc.) |
| `streamlit_app/` | Contains Streamlit UI files (`.py`) |
| `procedures/` | Python or SQL stored procedures |
| `data/` | Sample datasets (optional) |
| `readme.md` | Documentation for users |

---

## 🧾 Example: `manifest.yml`

```yaml
manifest_version: 1
name: customer360_app
version: 1.0.0
author: "Vedant Data Solutions"
description: "Customer 360 analytics dashboard built on Snowflake"
artifacts:
  - setup_script: setup.sql
  - streamlit: streamlit_app/app.py
  - procedure: procedures/data_refresh.py
privileges:
  - SELECT ON DATABASE RAW_DATA
  - EXECUTE ON PROCEDURE data_refresh
````

---

## 🪜 Steps to Create a Snowflake Native App Package

### 🧩 Step 1: Create a New Database for Development

```sql
CREATE DATABASE CUSTOMER360_APP_DB;
USE DATABASE CUSTOMER360_APP_DB;
```

### 🧩 Step 2: Build Application Artifacts

* Create SQL scripts, procedures, and Streamlit UI
* Example directory:

  ```
  customer360/
  ├── manifest.yml
  ├── setup.sql
  ├── streamlit_app/
  │   └── app.py
  ├── procedures/
  │   └── data_refresh.py
  └── readme.md
  ```

### 🧩 Step 3: Package the App

```sql
CREATE APPLICATION PACKAGE CUSTOMER360_PACKAGE;
```

### 🧩 Step 4: Add Version

```sql
ALTER APPLICATION PACKAGE CUSTOMER360_PACKAGE ADD VERSION V1 USING '@app_stage';
```

### 🧩 Step 5: Publish to Marketplace (Optional)

```sql
ALTER APPLICATION PACKAGE CUSTOMER360_PACKAGE SET DISTRIBUTION = MARKETPLACE;
```

---

## 🧠 Example App Flow

```
[Data Engineer]
     │
     ▼
Develops App → Packages Assets → Publishes App → Consumers Install
     │                                      │
     ▼                                      ▼
SQL, Streamlit, Procedures             Interactive UI in Snowsight
```

---

## 🧩 Example Streamlit UI (Inside Snowflake App)

```python
import streamlit as st
import snowflake.connector

st.title("💡 Customer 360 Analytics Dashboard")

# Snowflake Connection
conn = snowflake.connector.connect(session_parameters={
    "WAREHOUSE": "COMPUTE_WH",
    "DATABASE": "CUSTOMER360_APP_DB"
})

# Query Snowflake data
cur = conn.cursor()
cur.execute("SELECT REGION, SUM(SALES) AS TOTAL_SALES FROM SALES GROUP BY REGION")
rows = cur.fetchall()

# Display results
st.bar_chart({r[0]: r[1] for r in rows})
```

---

## 🧩 Example Setup Script (`setup.sql`)

```sql
CREATE SCHEMA IF NOT EXISTS APP_SCHEMA;
CREATE OR REPLACE TABLE APP_SCHEMA.SALES_SUMMARY AS
SELECT REGION, SUM(SALES) AS TOTAL_SALES
FROM RAW.SALES
GROUP BY REGION;

GRANT SELECT ON TABLE APP_SCHEMA.SALES_SUMMARY TO APPLICATION ROLE app_user;
```

---

## 🧠 Snowflake Marketplace Integration

| Feature                    | Description                                     |
| -------------------------- | ----------------------------------------------- |
| 🌎 **App Distribution**    | Publish apps globally via Snowflake Marketplace |
| 💳 **Monetization**        | Offer free or paid apps                         |
| 🧾 **Usage Tracking**      | Monitor adoption and consumption                |
| 🔐 **Secure Data Sharing** | Maintain consumer data isolation                |
| 🔄 **Version Management**  | Easily update or roll back app versions         |

---

## 🧩 Example Use Cases

| Use Case                      | Description                                      |
| ----------------------------- | ------------------------------------------------ |
| 🧮 **Data Quality Dashboard** | Validate pipelines and show metrics in Streamlit |
| 💰 **Finance KPIs App**       | Show real-time metrics for revenue and profit    |
| 🧠 **Customer 360 Analytics** | Combine CRM, sales, and product data             |
| 📊 **ETL Health Monitor**     | Visualize ingestion success and error rates      |
| 🧾 **Inventory Dashboard**    | Show stock levels and supply trends              |

---

## 🧠 Security and Governance

| Feature                        | Description                                      |
| ------------------------------ | ------------------------------------------------ |
| 🔐 **Role-Based Access**       | Limit user access within app                     |
| 📦 **Object Containerization** | App data stays within Snowflake boundary         |
| 🧩 **Sandboxing**              | Consumers can use app without exposing base code |
| 🕵️ **Data Masking**           | Protect sensitive information                    |
| 📜 **Audit Logging**           | Track access and usage history                   |

---

## ⚙️ Lifecycle Management

| Stage           | Description                                      |
| --------------- | ------------------------------------------------ |
| **Development** | Create and test app                              |
| **Packaging**   | Bundle assets using `CREATE APPLICATION PACKAGE` |
| **Deployment**  | Install as `APPLICATION` in target accounts      |
| **Update**      | Add new versions with `ALTER PACKAGE`            |
| **Publish**     | Share app via Marketplace or Private Listing     |

---

## 🧩 Example Consumer Installation

```sql
INSTALL APPLICATION CUSTOMER360_PACKAGE VERSION V1;
```

Then access via:

* Snowsight → **Projects → Applications**
* Launch the Streamlit UI

---

## 🧰 Benefits for Data Engineers

| Benefit                     | Description                                 |
| --------------------------- | ------------------------------------------- |
| ⚡ **End-to-End Deployment** | Deploy logic + UI + data in one package     |
| 🔒 **Secure Sharing**       | Share without exposing source data          |
| 🧠 **Integrated Compute**   | Runs inside Snowflake’s compute layer       |
| 💻 **Built-in UI**          | Streamlit app embedded directly             |
| 📦 **Reusable Modules**     | Version-controlled, modular, and portable   |
| 🌍 **Monetization Ready**   | Publish to the global Snowflake Marketplace |

---

## 🧩 Summary

| Concept        | Description                                                            |
| -------------- | ---------------------------------------------------------------------- |
| **What**       | Native App Package = self-contained Snowflake application              |
| **Includes**   | SQL, Streamlit, procedures, UDFs, metadata                             |
| **Where Used** | Marketplace, internal projects, or enterprise analytics                |
| **For Whom**   | Data engineers, app developers, and data providers                     |
| **Goal**       | Build, share, and monetize data applications natively within Snowflake |

> 💡 **Tip:** Snowflake Native Apps let you deploy full-stack, data-driven apps securely — without ever exporting your data.
