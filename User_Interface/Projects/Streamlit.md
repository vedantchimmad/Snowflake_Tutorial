# 🚀 Streamlit in Snowflake (Native App Framework)

---

## 🧭 Overview

**Streamlit in Snowflake** is a feature that allows developers and data engineers to **build and deploy interactive web applications** **directly inside Snowflake** — without needing external servers or data movement.

This feature combines:
- **Streamlit** (a Python-based web app framework)
- **Snowflake compute and storage**
- **Snowpark** (for data processing and ML logic)

> 🧠 It’s like building a **dashboard or data app directly on top of your Snowflake data warehouse**, all running *securely within Snowflake’s environment*.

---

## 🧩 Key Features

| Feature | Description |
|----------|--------------|
| 🧠 **No External Hosting** | Apps run entirely inside Snowflake |
| ⚡ **Live Data Access** | Directly query Snowflake tables via Snowpark |
| 🔐 **Secure & Governed** | Uses Snowflake roles, policies, and compute |
| 🧩 **Built with Streamlit** | Use standard Streamlit code syntax |
| 💡 **Instant Deployments** | Publish apps as **Native Apps** in minutes |
| 💻 **Python Integration** | Combine SQL + Snowpark + Python logic |
| 📊 **Visualization Ready** | Easily visualize data with charts, filters, and widgets |

---

## 🧱 Architecture Overview

```

+-------------------------------------------------------+

| Snowflake Cloud                                           |                                 |
| --------------------------------------------------------- | ------------------------------- |
| 🧠 Data Layer                                             | Tables, Views, Schemas          |
| ⚙️ Compute Layer                                          | Warehouses, Snowpark, Tasks     |
| 🧩 App Layer                                              | Streamlit Apps (Native Apps)    |
| 🔐 Governance Layer                                       | Roles, Policies, Access Control |
| +-------------------------------------------------------+ |                                 |

````

> The **Streamlit App** runs **natively** within Snowflake’s **Native App Framework**, leveraging **Snowpark** for data operations and **Streamlit UI** for visualization.

---

## 🧩 1. Creating a Streamlit App in Snowflake

### 📍 Steps:

1. **Open Snowsight UI**
   - Go to [https://app.snowflake.com](https://app.snowflake.com)
   - Navigate to **Projects → Streamlit**

2. **Create a New Streamlit App**
   - Click **➕ Create → Streamlit App**
   - Give your app a name (e.g., `Sales_Dashboard`)
   - Choose **Warehouse**, **Database**, and **Schema**

3. **Start Coding**
   - The editor opens with a default `streamlit_app.py`
   - Write your Streamlit + Snowpark code inside

4. **Run the App**
   - Click **Run** ▶️ to preview your app live
   - Save or publish when ready

---

## 💻 2. Basic Example: Query and Display Data

```python
import streamlit as st
from snowflake.snowpark import Session
from snowflake.snowpark.functions import col

# 🧠 Create Snowflake session
session = Session.builder.getOrCreate()

# 🧮 Query Snowflake table
df = session.table("SALES").group_by("REGION").agg({"SALES_AMOUNT": "sum"})

# 🧾 Convert to Pandas
data = df.to_pandas()

# 📊 Streamlit UI
st.title("📈 Regional Sales Dashboard")
st.bar_chart(data.set_index("REGION"))
````

---

## 🧮 3. Example: Filtering Data with Widgets

```python
import streamlit as st
from snowflake.snowpark import Session
from snowflake.snowpark.functions import col

st.title("🧭 Sales Explorer")

session = Session.builder.getOrCreate()

# Dropdown filter
regions = ["EAST", "WEST", "NORTH", "SOUTH"]
selected_region = st.selectbox("Select Region:", regions)

# Query filtered data
df = session.table("ORDERS").filter(col("REGION") == selected_region)
data = df.to_pandas()

# Display
st.dataframe(data)
st.metric(label="Total Orders", value=len(data))
```

---

## 🧠 4. Using SQL in Streamlit (Direct Queries)

```python
import streamlit as st
import pandas as pd

st.title("💰 Top 10 Customers")

data = st.experimental_connection("snowflake").query("""
    SELECT CUSTOMER_NAME, SUM(SALES_AMOUNT) AS TOTAL
    FROM SALES
    GROUP BY CUSTOMER_NAME
    ORDER BY TOTAL DESC
    LIMIT 10
""")

st.table(data)
```

> ✅ You can use `st.experimental_connection("snowflake")` to directly connect to your Snowflake environment — no manual credentials needed.

---

## ⚙️ 5. Configuring Streamlit Environment

| Setting       | Description                           |
| ------------- | ------------------------------------- |
| **Warehouse** | Compute resource for queries          |
| **Database**  | Where data is stored                  |
| **Schema**    | Logical grouping of objects           |
| **Role**      | Defines access permissions            |
| **Session**   | Snowpark context for Python execution |

```python
# Example configuration
session.use_warehouse("COMPUTE_WH")
session.use_database("SALES_DB")
session.use_schema("PUBLIC")
```

---

## 📦 6. Publishing a Streamlit App

Once your app is ready:

1. Click **Publish → Create Streamlit App**
2. Choose **Warehouse**, **Database**, **Schema**
3. Assign **roles and access** to viewers
4. App becomes visible under **Projects → Streamlit Apps**

---

## 🧰 7. Example: Advanced Streamlit Dashboard

```python
import streamlit as st
from snowflake.snowpark import Session
from snowflake.snowpark.functions import col
import matplotlib.pyplot as plt

st.set_page_config(page_title="Sales Overview", layout="wide")

session = Session.builder.getOrCreate()

# Data Query
df = session.table("SALES").group_by("REGION").agg({"SALES_AMOUNT": "sum"})
data = df.to_pandas()

# Layout
col1, col2 = st.columns(2)

with col1:
    st.metric("Total Sales", f"${data['SUM(SALES_AMOUNT)'].sum():,.2f}")

with col2:
    fig, ax = plt.subplots()
    ax.bar(data["REGION"], data["SUM(SALES_AMOUNT)"])
    st.pyplot(fig)
```

---

## 🧩 8. Integration with Snowpark

Streamlit Apps in Snowflake can use **Snowpark DataFrames** for scalable data operations.

```python
from snowflake.snowpark import Session
from snowflake.snowpark.functions import avg

session = Session.builder.getOrCreate()

# Create Snowpark DataFrame
sales_df = session.table("SALES")

# Transformation
summary = sales_df.group_by("REGION").agg(avg("SALES_AMOUNT").alias("AVG_SALES"))
summary.show()
```

---

## 📊 9. Visualization Options

| Library                                                | Use Case                   |
| ------------------------------------------------------ | -------------------------- |
| **Streamlit charts** (`st.line_chart`, `st.bar_chart`) | Quick insights             |
| **Matplotlib / Seaborn**                               | Advanced plots             |
| **Plotly**                                             | Interactive dashboards     |
| **Altair**                                             | Declarative visualizations |

---

## 🔐 10. Security and Governance

| Aspect                      | Description                                          |
| --------------------------- | ---------------------------------------------------- |
| **Access Control**          | Based on Snowflake roles and privileges              |
| **Data Governance**         | Apps respect masking policies and row-level security |
| **Compute Isolation**       | Runs on assigned warehouses                          |
| **No External Data Egress** | Everything stays within Snowflake                    |

---

## 🧭 11. Example Use Cases

| Use Case                     | Description                                    |
| ---------------------------- | ---------------------------------------------- |
| 📊 **Sales Dashboard**       | Track regional and product sales KPIs          |
| 🧮 **Data Quality Monitor**  | Visualize nulls, duplicates, and schema drift  |
| 🧠 **ML Predictions Viewer** | Display Snowpark ML model outputs              |
| 🧾 **Financial Reports**     | Create interactive business reports            |
| 🔍 **ETL Log Monitor**       | Real-time visibility into data pipeline health |

---

## 🧠 12. Best Practices for Data Engineers

| Practice                               | Recommendation                          |
| -------------------------------------- | --------------------------------------- |
| ✅ Use **Snowpark Sessions**            | For all data access                     |
| 📊 Use **Pandas for light transforms** | Convert small datasets to Pandas for UI |
| 🧠 Avoid large data in UI              | Paginate or summarize results           |
| 🔐 Apply role-based control            | Use `USE ROLE` wisely                   |
| 💾 Store app version                   | Use Git integration for Streamlit code  |
| 🚀 Reuse components                    | Use sidebar for filters and navigation  |

---

## 🧩 Summary

| Concept        | Description                             |
| -------------- | --------------------------------------- |
| **Technology** | Streamlit + Snowpark inside Snowflake   |
| **Purpose**    | Build & deploy data apps natively       |
| **Security**   | Fully governed under Snowflake IAM      |
| **Compute**    | Uses Snowflake virtual warehouses       |
| **Best For**   | Dashboards, analytics, data exploration |

> 💡 **In short**: Streamlit in Snowflake lets data engineers go from **data → insight → interactive app** — all inside the same secure platform.