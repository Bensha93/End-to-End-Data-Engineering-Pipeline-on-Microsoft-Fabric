# 🏗️ End-to-End Data Engineering Pipeline on Microsoft Fabric  
### *Kaggle Superstore Sales — Lakehouse Pipeline (Landing → Bronze → Silver → Silver+ → Gold)*

## 📘 Project Overview

This project demonstrates a **fully automated, parameter-driven data engineering pipeline** built on **Microsoft Fabric Lakehouse**.  
It extracts raw data from **Kaggle**, cleans, transforms, and enriches it across a **multi-layered architecture**, and finally produces **Gold-level analytical views** ready for reporting and BI dashboards.

The dataset used is the **Superstore Sales Dataset** from Kaggle, containing retail order and shipment records with customer, product, region, and sales details.

---

## 🧱 Architecture Overview


Each layer serves a specific purpose, enabling clean, maintainable, and auditable data transformation in line with **Medallion Architecture** best practices.

| Layer | Purpose | Output Type |
|-------|----------|-------------|
| **Landing** | Raw data ingestion from Kaggle (CSV/Excel) | Parquet files |
| **Bronze** | Basic data cleaning, date normalization, and schema alignment | Parquet (partitioned by Year/Month) |
| **Silver** | Deduplication, enrichment, and joining with external data (Wikidata API for state lat/long) | Parquet |
| **Silver+ (Normalization)** | Dimensional modeling — creates Dimension and Fact tables | Delta / Parquet tables |
| **Gold** | Analytical Materialized Lake Views for reporting | Lake views (SQL-based summaries) |

---

## ⚙️ Pipeline Flow and Notebooks

### 1️⃣ Kaggle → Landing
**Notebook:** `Kaggle_to_Landing.ipynb`

- Downloads dataset using **KaggleHub API**.  
- Detects encoding via **chardet** for robust file reading.  
- Cleans column names (handles spaces, special characters).  
- Adds audit metadata (`Year`, `Month`, `Processing_Time`).  
- Writes clean data to the **Landing zone** in OneLake as partitioned Parquet files.  
- Logs execution metrics to a `dbo.pipeline_log` table.

**Key Libraries:** `kagglehub`, `pandas`, `pyspark.sql`, `chardet`

---

### 2️⃣ Landing → Bronze
**Notebook:** `Landing_to_Bronze.ipynb`

- Reads partitioned data from Landing.  
- Converts date strings into Spark date types.  
- Adds standardized time partitions (`Year`, `Month`).  
- Writes to the **Bronze zone** as Parquet files.  
- Appends run statistics to `pipeline_log`.

**Enhancements:** Toggle parameter `RUN_LANDING_TO_BRONZE` allows selective pipeline execution.

---

### 3️⃣ Bronze → Silver
**Notebook:** `Bronze_to_Silver.ipynb`

- Reads from Bronze and removes duplicates/nulls across key columns.  
- Fetches **State-level latitude and longitude** dynamically from **Wikidata SPARQL API**.  
- Joins reference data to enrich the sales dataset with geo-coordinates.  
- Writes enriched, cleaned data to the **Silver zone** in partitioned Parquet format.  
- Logs execution to `pipeline_log`.

**Highlight:** Integration with external APIs (Wikidata) for real-time data enrichment.

---

### 4️⃣ Silver → Silver+ (Dimensional Modeling)
**Notebook:** `Silver_Plus_(Normalization).ipynb`

Implements **dimensional modeling** for analytical readiness:

| Dimension | Description |
|------------|-------------|
| `dim_customer` | Customer attributes: segment, region, state, city |
| `dim_product` | Product details: category, sub-category |
| `dim_state` | State attributes + geolocation |
| `dim_date` | Derived from order dates |
| **Fact Table:** `fact_sales` — links all dimensions and contains measures such as sales, profit, quantity, discount |

All tables are saved under `/Tables/SilverPlus` for consistency and reusability.

---

### 5️⃣ Silver+ → Gold
**Notebook:** `Gold_Layer.ipynb`

Creates **Materialized Lake Views** for advanced analytics and Power BI connectivity.

#### Gold Views Created
| View | Description |
|------|--------------|
| `mv_gold_sales_summary` | Sales/profit summary by category, region, month |
| `mv_gold_top_customers` | Top 50 customers by total sales |
| `mv_gold_product_summary` | Product-level profitability summary |
| `mv_gold_region_performance` | Regional KPIs (sales, profit, customers) |
| `mv_gold_monthly_trends` | Monthly sales and order trends |
| `mv_gold_customer_summary` | Customer segmentation metrics |
| `mv_gold_shipping_performance` | Shipping time efficiency and sales impact |

Each view is logged to the central `pipeline_log` table with start/end timestamps, runtime, and row counts.

---

## 📊 Data Logging & Monitoring

A unified log table (`dbo.pipeline_log`) captures metadata for every pipeline stage:

| Column | Description |
|---------|-------------|
| `Dataset` | Dataset or layer name |
| `Start_Timestamp`, `End_Timestamp` | Execution time window |
| `run_duration` | Duration (minutes) |
| `Stage` | Transformation stage (e.g., Bronze→Silver) |
| `Destination` | Lakehouse destination path |
| `Row_Count` | Total records processed |

This allows **end-to-end traceability** and enables **Fabric Data Activator or Power BI monitoring dashboards**.

---

## 🧩 Technologies & Tools

- **Microsoft Fabric Lakehouse**
- **PySpark**
- **Delta Lake / Parquet**
- **KaggleHub API**
- **Wikidata SPARQL API**
- **Azure OneLake (ABFSS Pathing)**
- **Fabric Notebooks & Pipelines**
- **Materialized Lake Views (Fabric SQL Endpoint)**

---

## 🚀 Key Features

✅ End-to-End ETL/ELT data flow  
✅ Automated logging with runtime metrics  
✅ External API integration (geo enrichment)  
✅ Modular notebook-based architecture  
✅ Schema evolution with `mergeSchema=true`  
✅ Dimensional model & Gold-level analytics  
✅ Power BI-ready Lakehouse tables/views  

---

## 🗂️ Folder Structure

## 📂 Project Folder Structure

```text
📁 Fabric_LH_Sales
│
├── 🧩 Notebooks
│   ├── Kaggle_to_Landing.ipynb
│   ├── Landing_to_Bronze.ipynb
│   ├── Bronze_to_Silver.ipynb
│   ├── Silver_Plus_(Normalization).ipynb
│   └── Gold_Layer.ipynb
│
├── 🪣 Lakehouse / OneLake Paths
│   ├── Files/Landing/
│   ├── Tables/dbo/Bronze/
│   ├── Tables/dbo/Silver/
│   ├── Tables/SilverPlus/
│   ├── Tables/Reference/
│   └── Tables/gold/
│
└── 🧾 dbo.pipeline_log  (execution log table)



## 🏁 Conclusion

This project exemplifies a **modern data engineering solution** built natively within Microsoft Fabric, showcasing skills in:
- **ETL/ELT pipeline design**
- **Spark optimization**
- **Cloud data architecture**
- **End-to-end data lifecycle management**

---

### 👤 Author
**Adewole Oyediran**  
Data Engineer | Microsoft Fabric | Azure | PySpark | ETL Automation  
📧 *Bensha2019@outlook.com*  


---

