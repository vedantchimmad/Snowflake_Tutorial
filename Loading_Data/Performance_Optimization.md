# 🚀 Performance Optimization in Snowflake

---

## 🧭 Overview

Performance optimization in **Snowflake** ensures your queries, transformations, and data pipelines run **efficiently**, **cost-effectively**, and **scalably**.  
It involves tuning **warehouse configurations**, **data structures**, and **query design** to maximize processing speed while minimizing credit consumption.

> 💡 Snowflake automatically handles many optimizations (scaling, caching, clustering), but engineers can further fine-tune performance for large workloads.

---

## ⚙️ Key Areas of Optimization

| Area | Description | Focus |
|------|--------------|--------|
| **Virtual Warehouse Optimization** | Tune compute clusters | Size, scaling, caching |
| **Query Optimization** | Optimize SQL queries | Joins, filters, predicates |
| **Data Modeling** | Organize tables efficiently | Star schema, clustering |
| **Caching Mechanisms** | Use result & metadata cache | Avoid recomputation |
| **Storage Management** | Optimize micro-partitions | Clustering, pruning |
| **Monitoring & Profiling** | Analyze query performance | Query profile, history |

---

## 🧩 1. Virtual Warehouse Optimization

### 🔹 Choose Proper Warehouse Size
- Start small (e.g., `X-SMALL`, `SMALL`) and scale up only if needed.
- Larger warehouses process queries faster but cost more.

```sql
CREATE WAREHOUSE ETL_WH
WITH WAREHOUSE_SIZE = 'MEDIUM'
AUTO_SUSPEND = 60
AUTO_RESUME = TRUE;
````

### 🔹 Enable Auto-Suspend & Auto-Resume

* **Auto-suspend** stops compute when idle → saves credits.
* **Auto-resume** restarts automatically when queries arrive.

### 🔹 Use Multi-Cluster Warehouse for Concurrency

Handles high concurrent workloads efficiently.

```sql
ALTER WAREHOUSE BI_WH
SET MIN_CLUSTER_COUNT = 1
    MAX_CLUSTER_COUNT = 3
    SCALING_POLICY = 'ECONOMY';
```

✅ **Benefit:** Improves concurrency without query queuing.

---

## 🧩 2. Query Optimization

### 🔹 Use Selective Columns

Avoid `SELECT *` — query only needed columns.

```sql
SELECT CUSTOMER_ID, TOTAL_SALES FROM SALES;
```

### 🔹 Filter Early

Apply **WHERE** and **LIMIT** early to reduce scanned data.

```sql
SELECT * FROM SALES WHERE REGION = 'APAC';
```

### 🔹 Optimize Joins

* Use **JOIN ON** with indexed or clustered columns.
* Filter data before joining.
* Prefer **INNER JOIN** if possible — smaller intermediate results.

### 🔹 Use CTEs or Temp Tables for Complex Queries

```sql
WITH CLEANED AS (
  SELECT * FROM SALES WHERE AMOUNT > 0
)
SELECT REGION, SUM(AMOUNT)
FROM CLEANED
GROUP BY REGION;
```

✅ **Benefit:** Reduces repeated subquery computation.

---

## 🧩 3. Data Modeling Optimization

### 🔹 Use Star or Snowflake Schema

Organize data into **fact** and **dimension** tables for clarity and efficiency.

### 🔹 Cluster Large Tables

Manual clustering improves pruning efficiency.

```sql
ALTER TABLE SALES
CLUSTER BY (REGION, ORDER_DATE);
```

> 🧠 Clustering helps Snowflake skip unnecessary micro-partitions during query execution.

### 🔹 Partitioning (Implicit)

Snowflake **automatically partitions data** into **micro-partitions (~16MB)**.
To maximize partition pruning:

* Apply **filters on clustered columns**
* Avoid unnecessary type casting in WHERE clause

---

## 🧩 4. Caching Mechanisms

| Cache Type          | Description                                   | Example                                 |
| ------------------- | --------------------------------------------- | --------------------------------------- |
| **Result Cache**    | Stores results of previous queries (24 hours) | Re-running same query → instant results |
| **Metadata Cache**  | Caches table stats and structure              | Faster metadata access                  |
| **Warehouse Cache** | Holds data in compute memory during session   | Speeds up repetitive queries            |

### 💡 Enable Caching Benefits

* Run similar queries within **same warehouse session**.
* Avoid frequent **warehouse suspend/resume** to retain cache.

---

## 🧩 5. Micro-Partition Pruning

Snowflake automatically divides data into **micro-partitions** (columnar storage).
You can enhance pruning efficiency using **clustered columns** and **query filters**.

```sql
SELECT * FROM SALES
WHERE ORDER_DATE >= '2025-01-01' AND REGION = 'APAC';
```

✅ **Benefit:** Only relevant partitions are scanned → faster execution.

---

## 🧩 6. Materialized Views for Precomputation

```sql
CREATE OR REPLACE MATERIALIZED VIEW MONTHLY_SALES AS
SELECT DATE_TRUNC('MONTH', ORDER_DATE) AS MONTH,
       SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM SALES
GROUP BY 1;
```

✅ **Use Case:** For repetitive queries with heavy aggregations.

> ⚡ Snowflake automatically refreshes materialized views incrementally.

---

## 🧩 7. Query Profile Analysis

### 🔹 View Query Execution Plan

```sql
SELECT * FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())
WHERE QUERY_TEXT ILIKE '%SALES%';
```

Then inspect in **Snowsight → Query Profile Tab**:

* Scan time
* Bytes processed
* Partition pruning
* Execution steps

✅ **Goal:** Identify bottlenecks like large scans or inefficient joins.

---

## 🧩 8. Statistics & Metadata

Snowflake automatically collects **metadata statistics** (row count, min/max values, etc.) for efficient pruning.

You can view stats using:

```sql
SHOW TABLES LIKE 'SALES';
DESC TABLE SALES;
```

---

## 🧩 9. Automatic Clustering

Let Snowflake manage clustering maintenance automatically.

```sql
ALTER TABLE SALES
CLUSTER BY (REGION, ORDER_DATE);
ALTER TABLE SALES
SET AUTO_CLUSTERING = TRUE;
```

✅ **Use Case:** Ideal for continuously changing large tables.

---

## 🧩 10. Scaling & Resource Management

| Feature                     | Purpose                                 | Example                                             |
| --------------------------- | --------------------------------------- | --------------------------------------------------- |
| **Multi-Cluster Warehouse** | Scales horizontally under load          | `MIN_CLUSTER_COUNT = 2`                             |
| **Warehouse Size**          | Scales vertically for faster processing | `LARGE`, `XLARGE`                                   |
| **Resource Monitors**       | Prevent credit overuse                  | `CREATE RESOURCE MONITOR monitor1 CREDIT_QUOTA=100` |

---

## 🧩 11. Using Query Acceleration Service (QAS)

For very large, complex queries:

```sql
ALTER WAREHOUSE ANALYTICS_WH SET QUERY_ACCELERATION_MAX_SCALE_FACTOR=8;
```

✅ **Benefit:** Offloads parts of query execution to optimized compute for faster results.

---

## 🧩 12. Performance Monitoring Tools

| Tool                    | Description                     | Usage                                            |
| ----------------------- | ------------------------------- | ------------------------------------------------ |
| **Query History**       | View query performance metrics  | `INFORMATION_SCHEMA.QUERY_HISTORY`               |
| **Snowsight Dashboard** | Visual query profile analysis   | GUI                                              |
| **Account Usage Views** | Track warehouse usage & credits | `SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY` |
| **Resource Monitors**   | Prevent warehouse overuse       | Alerts for credit thresholds                     |

---

## 🧠 Best Practices Summary

| Area                | Best Practice                          | Benefit                              |
| ------------------- | -------------------------------------- | ------------------------------------ |
| **Compute**         | Auto-suspend & auto-resume warehouses  | Save credits                         |
| **Caching**         | Reuse cached results                   | Faster response                      |
| **Query Design**    | Avoid SELECT *                         | Reduce data scan                     |
| **Data Structure**  | Use clustering for large tables        | Improve pruning                      |
| **Materialization** | Use materialized views                 | Precompute heavy aggregations        |
| **Automation**      | Use Streams & Tasks                    | Schedule transformations efficiently |
| **Monitoring**      | Use Query Profile and Resource Monitor | Detect bottlenecks early             |

---

## 📊 Example: Optimized Query Flow

```
[Data in Micro-Partitions]
       ↓
[Warehouse Caches Active Data]
       ↓
[Clustered by REGION, ORDER_DATE]
       ↓
[Query Uses Filter → Prunes Irrelevant Data]
       ↓
[Cached Results Reused for Similar Queries]
```

---

## 🧩 Summary Table

| Optimization Type   | Technique                         | Example / Feature          |
| ------------------- | --------------------------------- | -------------------------- |
| **Compute**         | Resize or multi-cluster warehouse | `ALTER WAREHOUSE`          |
| **Query**           | Filter early, avoid SELECT *      | `WHERE REGION='APAC'`      |
| **Data Storage**    | Cluster & prune partitions        | `CLUSTER BY`               |
| **Caching**         | Result / Metadata cache           | Automatic                  |
| **Materialization** | Materialized views                | `CREATE MATERIALIZED VIEW` |
| **Automation**      | Streams & Tasks                   | Incremental updates        |
| **Monitoring**      | Query history & profile           | Snowsight Dashboard        |

> 💡 **Tip:** Optimize iteratively — start by analyzing query profiles, reduce scanned data, leverage caching, and then scale compute only when necessary.
