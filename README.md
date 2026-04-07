# Retail Medallion Data Pipeline: End-to-End Capstone

Build a production-grade batch data pipeline that ingests raw retail CSV data, applies rigorous data engineering transformations using the Medallion Architecture, ensures high data quality, and synchronizes the final analytical models into Snowflake for business intelligence.

## 🏗️ Architecture Overview
The project follows the **Medallion Architecture** to ensure data reliability and quality as it moves through the pipeline:
* **Bronze (Raw Ingestion)**: Ingests raw CSV files from source, persisting them as Delta Tables with added ingestion metadata.
* **Silver (Cleaned & Validated)**: Filters, cleans, and validates data to ensure it is "Trustworthy".
* **Gold (Business Modeling)**: Implements a Star Schema (Dimensional Modeling) optimized for business intelligence.
* **Warehouse**: Syncs final Gold models to Snowflake for downstream BI and reporting.

## 🛠️ Tech Stack
* **Processing**: Databricks (Spark & Delta Lake)
* **Data Warehouse**: Snowflake
* **Orchestration**: Apache Airflow
* **Languages**: Python (PySpark), SQL

## 📁 Mandatory Project Structure
This project adheres to clean code principles with the following folder structure:

```Text
capstone-project/
├── data/raw/              # Input CSV files (Customers, Products, Orders, Items)
├── src/                   # PySpark Processing Logic
│   ├── ingestion.py       # Phase 1: Bronze layer logic
│   ├── silver_transform.py # Phase 2: Cleaning & Validation logic
│   ├── gold_transform.py   # Phase 3: Dimensional modeling
│   └── data_quality.py    # Phase 4: Custom DQ framework
├── sql/                   # SQL DDL Scripts
│   └── gold_tables.sql    # DDL for Snowflake synchronization
├── airflow/               # Orchestration Scripts
│   └── retail_dag.py      # Airflow DAG Definition
├── config/                # Configuration Files
│   └── config.yaml        # Environment & Table configurations
└── README.md              # Documentation & Setup Guide
```

## 🚀 Implementation Phases

### Phase 1: Bronze (RAW) - Ingestion
* **Task**: Read CSVs and persist them as Delta Tables.
* **Rule 1**: Zero transformations to keep data in its original state.
* **Rule 2**: Append metadata including `ingestion_timestamp` and `source_file_name`.
* **Output**: `bronze_customers`, `bronze_products`, `bronze_orders`, and `bronze_order_items`.

### Phase 2: Silver - Cleaning & Validation
* **Task**: Ensure the data is "Trustworthy" for downstream analysis.
* **Customer Validations**: Remove duplicate `customer_id` and validate email format (must contain `@`).
* **Product Validations**: Ensure `price` is strictly greater than 0.
* **Order Validations**: Filter for specific statuses and ensure `order_date` is not NULL.
* **Integrity**: Confirm that `order_id` in `order_items` exists in the `orders` table.

### Phase 3: Gold - Dimensional Modeling
* **Task**: Build an optimized Star Schema for business intelligence.
* **fact_sales**: A join of Orders, Items, and Products including a calculated `total_amount` ($quantity \times price$).
* **Filter**: The fact table includes ONLY `DELIVERED` orders.
* **dim_customer**: Flattened geography and name fields (first + last).
* **dim_product**: Standardized product catalog for reporting.

### Phase 4: Data Quality (DQ) Checks
* **Uniqueness**: Automated checks for Primary Key violations in Gold tables.
* **Completeness**: Verification of no NULL values in critical columns such as Date, ID, and Amount.
* **Reconciliation**: Record count checks comparing Bronze vs. Silver to ensure data consistency.

---

## ⚙️ Setup & Configuration

### 1. Databricks Secrets
* Store your Snowflake credentials and environment tokens in Databricks Secrets to avoid hardcoding sensitive information.

### 2. Snowflake Loading
* **Connector**: Uses the Spark-Snowflake Connector for data synchronization.
* **Idempotency**: Implements `OVERWRITE` mode or `MERGE` logic to prevent duplicate data on retries.

### 3. Orchestration Flow
The pipeline is managed via Apache Airflow with the following task sequence:
`ingest_raw` → `build_silver` → `dq_check_silver` → `build_gold` → `dq_check_gold` → `load_snowflake`

---

## 📈 Performance Optimizations
* **Partitioning**: Silver and Gold tables are partitioned by `order_date` to optimize query performance.
* **Broadcast Joins**: Utilized when joining the `dim_product` (small table) to `fact_sales` to reduce data shuffling.
* **Caching**: The Silver orders table is cached as it is accessed multiple times during Gold transformations.

---

## 👥 Contributors
* **Wasim**: Tasks assigned.
* **Anurag**: Tasks assigned.
* **Avranit**: Tasks assigned.

---

## ✅ Evaluation Criteria
This project is built to meet the following weights:
* **Spark Logic (25%)**: Vectorized operations only; no loops.
* **Data Modeling (15%)**: Proper Star Schema in the Gold layer.
* **Data Quality (15%)**: Automated and logged DQ checks.
* **Airflow DAG (15%)**: Modular tasks with retry logic.
* **Git Practice (10%)**: Professional versioning and multiple commits.
