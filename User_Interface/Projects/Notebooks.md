# 📓 Snowflake Notebooks (Snowsight)

---

## 🧭 Overview

**Snowflake Notebooks** (available in **Snowsight**) are an **interactive development environment** inside Snowflake that allows you to combine:
- **SQL**, **Python**, and **Markdown**
- **Visualizations**, **data analysis**, and **data transformations**
- Direct integration with **Snowpark** and **Snowflake data objects**

> 🧠 Think of Snowflake Notebooks as **Jupyter-style notebooks**, but running **natively inside the Snowflake platform** with **full data security** and **compute efficiency**.

---

## 🧩 Key Features

| Feature | Description |
|----------|--------------|
| 🧮 **Multi-language support** | Combine SQL and Python cells in one environment |
| 📊 **Data visualization** | Build quick charts directly from query results |
| 🧠 **Integrated Snowpark** | Use Snowpark APIs for advanced data processing |
| 🔐 **Secure execution** | Runs directly inside your Snowflake environment |
| ⚙️ **Environment control** | Choose role, warehouse, and database context |
| 💾 **Versioned** | Automatically saves and version-controls notebooks |
| 👥 **Collaborative** | Share notebooks across teams or roles |
| 🚀 **Interactive outputs** | View dataframes, plots, and charts in-line |

---

## 🧱 Notebook Interface Layout

| Section | Description |
|----------|--------------|
| **Notebook Toolbar** | Run all cells, add cells, and save notebook |
| **Context Bar** | Select warehouse, role, database, and schema |
| **Cell Area** | Add SQL, Python, or Markdown cells |
| **Output Panel** | View results, logs, and visualizations |
| **File Panel** | View saved notebooks and folders |
| **Schedule Tab** | Automate execution (for scheduled runs) |

---

## 🧩 1. Creating a Notebook

### 📍 Steps:
1. Open **Snowsight UI** → [https://app.snowflake.com](https://app.snowflake.com)
2. Navigate to **Projects → Notebooks**
3. Click **➕ Create Notebook**
4. Select **SQL & Python** runtime
5. Set **Warehouse**, **Role**, **Database**, and **Schema**

---

## 🧮 2. Supported Cell Types

| Type | Purpose | Example |
|------|----------|----------|
| **SQL Cell** | Execute SQL queries directly | `SELECT * FROM SALES;` |
| **Python Cell** | Run Python code using Snowpark | `df = session.table("SALES").filter(col("REGION") == "WEST")` |
| **Markdown Cell** | Add documentation and headings | `## Sales Summary` |

---

## 🧠 3. Example: SQL + Python Workflow

### 🧩 Step 1: Query with SQL
```sql
-- SQL Cell
USE DATABASE SALES_DB;
USE SCHEMA PUBLIC;

SELECT REGION, SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM ORDERS
GROUP BY REGION;
````

### 🧩 Step 2: Process in Python

```python
# Python Cell
from snowflake.snowpark.functions import col
df = session.table("ORDERS").group_by(col("REGION")).agg({"SALES_AMOUNT": "sum"})
df.show()
```

### 🧩 Step 3: Visualize Results

```python
# Visualization Example
import matplotlib.pyplot as plt

data = df.to_pandas()
plt.bar(data['REGION'], data['SUM(SALES_AMOUNT)'])
plt.title("Sales by Region")
plt.xlabel("Region")
plt.ylabel("Total Sales")
plt.show()
```

### 🧩 Step 4: Document Using Markdown

```markdown
## 📊 Sales Insights
The **WEST** region leads in total sales for Q4, followed by **NORTH** and **EAST**.
```

---

## ⚙️ 4. Configuring Execution Context

Each notebook can have its **own execution context**, which you can configure from the **top context bar**:

| Setting            | Description                                |
| ------------------ | ------------------------------------------ |
| **Role**           | Security role used for queries             |
| **Warehouse**      | Compute engine for execution               |
| **Database**       | Default database                           |
| **Schema**         | Default schema                             |
| **Python Runtime** | Snowpark runtime version (e.g., 3.8, 3.10) |

```python
# Example: Setting context in Python
session.use_warehouse("COMPUTE_WH")
session.use_database("SALES_DB")
session.use_schema("PUBLIC")
```

---

## 🧮 5. Data Visualization Options

| Visualization              | Description                             |
| -------------------------- | --------------------------------------- |
| **Table View**             | Default tabular query result            |
| **Bar / Line / Pie Chart** | Quick chart creation from result set    |
| **Interactive Plots**      | Using matplotlib, seaborn, or Plotly    |
| **Snowsight Charts**       | Built-in no-code charts for SQL outputs |

Example Visualization:

```sql
SELECT PRODUCT_CATEGORY, SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM SALES
GROUP BY PRODUCT_CATEGORY;
```

> In Snowsight, click **“Visualize” → Bar Chart** to instantly view results.

---

## 📦 6. Integration with Snowpark

**Snowpark** is the Python API for working with Snowflake data directly inside notebooks.

### Example: Using Snowpark API

```python
from snowflake.snowpark import Session
from snowflake.snowpark.functions import col

df = session.table("CUSTOMERS")
filtered = df.filter(col("COUNTRY") == "USA")
filtered.show()
```

---

## 🔐 7. Scheduling and Automation

You can **schedule notebooks** to run automatically at specific intervals.

| Feature                    | Description                             |
| -------------------------- | --------------------------------------- |
| **Schedule Tab**           | Set daily, weekly, or hourly runs       |
| **Parameters**             | Pass runtime parameters to notebooks    |
| **Notifications**          | Send alerts on completion or failure    |
| **Integration with Tasks** | Trigger notebooks using Snowflake Tasks |

---

## 💾 8. Saving and Sharing Notebooks

| Action                 | Description                        |
| ---------------------- | ---------------------------------- |
| **Save Notebook**      | Automatically saves progress       |
| **Rename Notebook**    | Customize for clarity              |
| **Share Notebook**     | Share with specific users or roles |
| **Duplicate Notebook** | Clone for experimentation          |
| **Export Notebook**    | Download as `.ipynb` or `.json`    |

---

## 🧩 9. Example End-to-End Notebook

```markdown
# Customer Profitability Analysis

## Step 1: Load Data
```

```sql
SELECT CUSTOMER_ID, SUM(SALES_AMOUNT) - SUM(COST) AS PROFIT
FROM SALES
GROUP BY CUSTOMER_ID;
```

```markdown
## Step 2: Analyze in Python
```

```python
df = session.table("SALES")
profit_df = df.group_by("CUSTOMER_ID").agg((col("SALES_AMOUNT") - col("COST")).alias("PROFIT"))
profit_df.show()
```

```markdown
## Step 3: Visualize Top 5 Customers
```

```python
pdf = profit_df.to_pandas().nlargest(5, "PROFIT")
plt.bar(pdf["CUSTOMER_ID"], pdf["PROFIT"])
plt.title("Top 5 Customers by Profit")
plt.show()
```

---

## 🧭 10. Best Practices for Data Engineers

| Practice                       | Recommendation                                       |
| ------------------------------ | ---------------------------------------------------- |
| ✅ Set context early            | Always define warehouse, role, and schema            |
| 🧠 Modular design              | Use markdown for sections and clarity                |
| 🧩 Combine SQL + Python        | Use SQL for data extraction, Python for analytics    |
| 💾 Save often                  | Snowflake auto-saves but manual saves ensure backups |
| 📊 Use built-in visualizations | For quick insights                                   |
| 🔐 Manage access               | Share only with appropriate roles                    |
| ⚡ Use Snowpark DataFrames      | For better performance and scalability               |

