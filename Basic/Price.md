# 💰 Snowflake Pricing Model

---

## 🧭 Overview

**Snowflake Pricing** is based on a **consumption-based model**, meaning you only pay for the resources you use.  
The total cost is divided into **three primary components**:

1. **Compute (Virtual Warehouses)**
2. **Storage**
3. **Cloud Services**

> 🧠 Snowflake offers flexible **pay-as-you-go** and **pre-purchased credit** options, allowing cost control based on usage patterns.

---

## 🧩 Key Pricing Components

| Category | Description | Billed As |
|-----------|--------------|-----------|
| ⚙️ **Compute (Warehouses)** | Processing power used for queries, ETL, transformations | Per-second credit usage |
| 💾 **Storage** | Data stored in Snowflake tables, stages, and time travel | Per TB per month |
| ☁️ **Cloud Services** | Metadata management, optimization, authentication | ~10% of total compute |
| 🔄 **Data Transfer** | Movement of data between regions or clouds | Per GB (varies by region) |

---

## 🧮 Snowflake Pricing Structure

| Cost Area | Metric | Notes |
|------------|--------|-------|
| 🧠 **Compute** | Credits / Second | Depends on warehouse size |
| 💽 **Storage** | $40–$50 per TB / Month | Compressed columnar storage |
| 🌍 **Data Transfer** | $0.05–$0.12 per GB | Between cloud regions/providers |
| 🔐 **Cloud Services** | 10% of Compute | Covers metadata and security ops |

> 💡 You can control costs using **Resource Monitors**, **Auto Suspend**, and **Auto Resume** features.

---

## ⚙️ Compute Pricing (Credit-based)

Snowflake charges compute costs in **credits**, which depend on **warehouse size** and **region**.

| Warehouse Size | Credits / Hour | Use Case |
|----------------|----------------|----------|
| X-Small | 1 | Small dev/test workloads |
| Small | 2 | Light reporting jobs |
| Medium | 4 | Daily ETL or analytics |
| Large | 8 | Multi-user queries |
| X-Large | 16 | Heavy transformations |
| 2X–6X Large | 32–256 | Enterprise-grade pipelines |

### 🧮 Example:
- Warehouse: `Medium` = 4 credits/hour  
- Run time: 15 minutes (0.25 hour)  
- **Credits used** = 4 × 0.25 = **1 credit**

If your plan costs $3 per credit:
> 💰 Cost = **$3 × 1 = $3** for 15 minutes of compute.

---

## 💾 Storage Pricing

| Storage Type | Description | Cost (approx.) |
|---------------|--------------|----------------|
| 🧱 **Active Storage** | Data currently stored | $40–$50 per TB/month |
| 🕒 **Time Travel Storage** | Retained historical data | Included within limit (1 day for standard) |
| ♻️ **Fail-safe Storage** | Backup for disaster recovery | Additional charge beyond time travel |
| 🗄️ **Stage Storage** | Data temporarily staged for loading | Same as active storage |

### 🧮 Example:
If you store **2 TB** of data for one month:
> 💰 Cost = 2 TB × $40 = **$80 per month**

---

## ☁️ Cloud Services Cost

Snowflake uses compute for background activities like:
- Query optimization
- Metadata management
- Security checks
- Access control

Usually, **<10% of total compute cost**.

| Service | Included? |
|----------|------------|
| Query compilation | ✅ |
| Authentication | ✅ |
| Result caching | ✅ |
| Metadata and statistics | ✅ |

---

## 🧩 Data Transfer Pricing

| Transfer Type | Description | Approx. Cost |
|----------------|--------------|---------------|
| Intra-region | Same region and cloud | Free |
| Inter-region | Different cloud regions | $0.05–$0.12 per GB |
| Cross-cloud | Between AWS, Azure, GCP | $0.12 per GB |
| Outbound to Internet | Download from Snowflake | Variable by provider |

> ⚠️ Tip: Keep data within the same region to **avoid cross-region fees**.

---

## 🧱 Example Cost Breakdown

| Component | Usage | Rate | Cost |
|------------|--------|------|------|
| Compute | 500 credits | $3 / credit | $1,500 |
| Storage | 2 TB | $40 / TB | $80 |
| Cloud Services | 10% of compute | $150 | $150 |
| Data Transfer | 50 GB | $0.10 / GB | $5 |
| **Total Monthly Cost** | — | — | **$1,735** |

---

## 🧩 Snowflake Editions and Pricing Tiers

| Edition | Description | Key Features |
|----------|--------------|---------------|
| ❄️ **Standard** | Entry-level | Core compute, storage, SQL |
| 💼 **Enterprise** | Business-grade | Time Travel (90 days), fail-safe |
| 🧠 **Business Critical** | Regulated industries | Data encryption, HIPAA, compliance |
| 🏛️ **Virtual Private Snowflake (VPS)** | Dedicated environment | Private deployment, max security |

> 💡 Edition choice affects **credit cost multiplier**:
- **Enterprise** → +10–15%
- **Business Critical** → +25–30%
- **VPS** → Custom pricing

---

## 💳 Pricing by Cloud Provider

| Cloud Provider | Region Example | Notes |
|----------------|----------------|-------|
| ☁️ AWS | us-east-1 | Base pricing |
| 🔷 Azure | east-us | Slightly higher credit rate |
| 🟢 GCP | us-central | Similar to AWS |

---

## 🧾 Resource Monitors (Cost Control)

Use **Resource Monitors** to set credit limits and prevent overspending.

```sql
CREATE RESOURCE MONITOR monitor_wh
  WITH CREDIT_QUOTA = 500
  TRIGGERS ON 90 PERCENT DO NOTIFY
  TRIGGERS ON 100 PERCENT DO SUSPEND;
````

---

## 🧠 Cost Optimization Best Practices

| Strategy                                  | Description                       |
| ----------------------------------------- | --------------------------------- |
| ⚙️ **Use Auto-Suspend/Resume**            | Stop warehouses when idle         |
| 🧮 **Right-Size Warehouses**              | Start small, scale up dynamically |
| 🧠 **Use Caching**                        | Avoid recomputation               |
| 🔄 **Use Serverless Tasks**               | Pay only for scheduled runs       |
| 📊 **Monitor Usage**                      | Use ACCOUNT\_USAGE views          |
| 💾 **Compress Data**                      | Save on storage                   |
| 🚀 **Use Multi-Cluster Only When Needed** | Avoid unnecessary scaling         |
| 🧩 **Separate Dev/Test Warehouses**       | Reduce interference and cost      |

---

## 🧮 View Account Usage Details

```sql
-- Warehouse credit consumption
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY
ORDER BY START_TIME DESC;

-- Storage usage
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.STORAGE_USAGE
ORDER BY USAGE_DATE DESC;

-- Daily credit usage
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.DAILY_USAGE
ORDER BY USAGE_DATE DESC;
```

---

## 📊 Monitoring in Snowsight

1. Open **Snowsight → Admin → Usage**
2. View tabs for:

    * Warehouse credit usage
    * Storage consumption
    * Query costs by user
    * Cloud services activity
3. Set alerts and usage limits.

---

## 🧩 Example Real-World Monthly Estimate

| Category                     | Usage                                                       | Cost        |
| ---------------------------- | ----------------------------------------------------------- | ----------- |
| ETL Warehouse (Large)        | 8 hours/day × 30 days = 240 hrs × 8 credits = 1,920 credits | \$5,760     |
| Analytics Warehouse (Medium) | 4 hours/day × 30 days = 120 hrs × 4 credits = 480 credits   | \$1,440     |
| Storage                      | 5 TB                                                        | \$200       |
| Cloud Services (\~10%)       | —                                                           | \$720       |
| **Total Monthly Cost**       | —                                                           | **\$8,120** |

---

## 🧩 Summary

| Concept            | Description                                           |
| ------------------ | ----------------------------------------------------- |
| **Pricing Model**  | Pay-as-you-go based on compute, storage, and services |
| **Compute Unit**   | Credits (per warehouse size and duration)             |
| **Storage Unit**   | Per TB per month                                      |
| **Edition Impact** | Higher tiers add advanced features and cost           |
| **Cost Control**   | Auto-suspend, resource monitors, caching              |
| **Transparency**   | Full visibility in Snowsight and ACCOUNT\_USAGE views |

> 💡 **Tip:** Start with small warehouses, enable auto-suspend, and monitor usage regularly to get the most value from Snowflake without overspending.

