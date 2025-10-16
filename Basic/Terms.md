# 📚 Common Terms Used in Snowflake

---

## 🧭 Overview

Snowflake has its own set of **core concepts and terminology** that every data engineer should understand. These terms relate to **data storage, compute, access control, and data management**.

> 🧠 Understanding these terms is essential to design efficient pipelines, manage resources, and maintain security in Snowflake.

---

## 🧩 Key Snowflake Terms

| Term | Description | Example / Notes |
|------|-------------|----------------|
| **Database** | Logical container for schemas and tables | `CREATE DATABASE SALES_DB;` |
| **Schema** | Container for tables, views, and other objects | `CREATE SCHEMA PUBLIC;` |
| **Table** | Structured data stored in Snowflake | `SALES`, `CUSTOMERS` |
| **View** | Virtual table defined by a query | `CREATE VIEW SALES_VIEW AS SELECT * FROM SALES;` |
| **Stage** | Temporary or permanent location to hold files before loading | `@SALES_STAGE` |
| **File Format** | Defines how to interpret staged files | CSV, JSON, Parquet, Avro |
| **Warehouse** | Virtual compute engine for executing queries | `ANALYTICS_WH` |
| **Virtual Warehouse** | Same as warehouse, can scale independently | Size: X-Small to 6X-Large |
| **Role** | Collection of privileges for access control | `ANALYST_ROLE`, `ETL_ROLE` |
| **User** | Account accessing Snowflake, assigned roles | `vedant_user` |
| **Privilege** | Permission to perform an action on an object | `SELECT`, `INSERT`, `USAGE` |
| **Resource Monitor** | Tool to track and control credit usage | Set credit quotas for warehouses |
| **Stage Object** | Internal, table, or user stage | `@%my_table` or `@~` |
| **Time Travel** | Feature to query historical data for a period | Query deleted or updated rows |
| **Fail-safe** | Backup storage for disaster recovery | Retains data after Time Travel period |
| **Snowpipe** | Continuous data ingestion service | Auto-load files into a table |
| **External Table** | Table that queries data directly in cloud storage | S3, Azure Blob, GCS |
| **Materialized View** | Precomputed view for faster query performance | `CREATE MATERIALIZED VIEW SALES_MV AS SELECT ...` |
| **Task** | Scheduled or recurring SQL statement | ETL automation |
| **Stage File** | Individual file stored in a stage | `sales_2025.csv` |
| **COPY INTO** | Command to load data from stage into table | Bulk load operation |
| **Internal Stage** | Stage inside Snowflake account | `@my_stage` |
| **External Stage** | Stage in cloud storage outside Snowflake | `@s3_stage` |
| **Snowflake Account** | Overall environment for your data, compute, and services | Each org/account has unique name |
| **Virtual Private Snowflake (VPS)** | Dedicated Snowflake deployment | For maximum security and compliance |
| **Snowsight** | Snowflake web-based UI | Used for queries, dashboards, and monitoring |
| **Streams** | Track changes in tables for CDC | Works with tasks and Snowpipe |
| **Tasks** | Automated SQL execution for pipelines | `CREATE TASK daily_sales_task ...` |
| **Cloning** | Instant copy of database, schema, or table | Zero-copy clone for testing |
| **Stages & File Formats** | Predefined objects to load/unload data | Used in COPY INTO operations |
| **Query Profile** | Detailed execution plan of queries | Helps in performance tuning |
| **Virtual Private Snowflake (VPS)** | Isolated deployment for organizations | Enhanced security & compliance |
| **Marketplace** | Snowflake platform for apps and datasets | Buy/sell data and native apps |

---

## ⚡ Additional Concepts

| Concept | Description |
|---------|-------------|
| **Compute vs Storage Separation** | Compute (warehouse) is separate from storage; can scale independently |
| **Multi-Cluster Warehouse** | Multiple clusters of compute to handle concurrency |
| **Result Caching** | Snowflake caches query results to avoid recomputation |
| **Zero-Copy Cloning** | Create clones without physically copying data |
| **Multi-Cloud Support** | Works on AWS, Azure, GCP |
| **Continuous Data Sharing** | Securely share live data across accounts without copying |

---

## 🧠 Best Practices

- Use **roles** to manage access instead of direct user privileges.  
- Keep **stages organized** for efficient loading/unloading.  
- Leverage **Time Travel** and **Fail-safe** for data recovery.  
- Use **Tasks** and **Streams** for ETL automation.  
- Monitor **warehouses and usage** to control costs.  

---

## 🧩 Summary

| Category | Key Terms |
|----------|-----------|
| **Compute** | Warehouse, Virtual Warehouse, Multi-Cluster Warehouse |
| **Storage** | Database, Schema, Table, Stage, File Format |
| **Access Control** | Role, User, Privilege, Resource Monitor |
| **Data Management** | Time Travel, Fail-safe, Streams, Tasks, Materialized View |
| **ETL / Loading** | COPY INTO, Snowpipe, External Table, Stage File |
| **UI & Monitoring** | Snowsight, Query Profile, Marketplace |

> 💡 **Tip:** Mastering these terms will make **Snowflake easier to navigate and manage**, enabling effective data engineering workflows.
```
