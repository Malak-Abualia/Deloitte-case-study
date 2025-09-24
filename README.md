# Deloitte-case-study
# Retail Orders Analytics — Data Warehouse MVP

This repository contains the core SQL DDL, staging/quarantine/audit design, and artifacts used to build a small end‑to‑end DW pipeline for the Retail Orders dataset.

## Repository Structure
```
.
├─ ddl/                      # SQL DDL for schemas/tables, indexes, constraints
├─ etl/                      # (placeholder) loaders/transformers (psql, Python, or dbt)
├─ profiling/                # (placeholder) data profiling / inconsistencies reports
├─ docs/                     # HLD, ERD, and README assets
└─ marts/                    # (placeholder) star-schema marts build scripts
```

Suggested initial files to commit:

- `ddl/01_create_schemas_and_tables.sql` — schemas (raw, stg, core, audit, marts, mdm) and key tables (stg.orders, stg.orders_quarantine, core.dim_date, core.fact_order_sales, audit.*).  
- `docs/HLD.pptx` — High-Level Design slides.  
- `docs/ERD.jpg` — Dimensional model (star schema) for Fact_Order_Sales and related dimensions.  
- `profiling/Task_5_Inconsistencies_Analysis.csv` — data quality findings (counts per inconsistency).  
- `reports/Task_6_2_Data_Marts_Rows.csv` — row counts & distinct PK checks per mart.  
- `Task_8_GitHub_Link.txt` — the requested deliverable with this repo link.

> Notes:  
> • HLD overview and the processing flow are defined in the slides you prepared.  
> • The consolidated DDL includes audit + quarantine tables, date dimension, and fact table with indexes.

## How to Reproduce / Execute

### 1) Create DB & Schemas
Run the DDL in order (psql example):
```bash
psql -h localhost -U postgres -d retail_orders -f ddl/01_create_schemas_and_tables.sql
```

This creates:
- Schemas: `raw`, `stg`, `core`, `audit`, `marts`, `mdm`.
- Staging table `stg.orders` with basic checks and audit columns (`load_ts`, `batch_id`).
- Quarantine table `stg.orders_quarantine` for failed DQ rows.
- Core `core.dim_date` and `core.fact_order_sales` with constraints and indexes.
- Audit tables: `audit.pipeline_execution_audit`, `audit.file_audit`, `audit.table_audit` to track runs and row counts.

### 2) Load RAW and STG
- Ingest monthly CSV files into `raw.orders_file` (columns kept as text to avoid parse failures).
- Clean & cast into `stg.orders` (fix dates, numerics, trim/case, etc.). Rows that fail rules go to `stg.orders_quarantine` with a `dq_type` and `file_name` for SME review.

### 3) Build CORE & MARTS
- Populate `core.dim_date` from the distinct dates found in staging (or generate a full calendar).  
- Transform and load `core.fact_order_sales` joining conformed dimensions (`dim_product`, `dim_customer`, `dim_geography`, `dim_shipmode`).  
- (Optional) Publish thin marts in the `marts` schema for BI consumption.

### 4) Auditing & Monitoring
- Start a record in `audit.pipeline_execution_audit` at the beginning of a run.  
- For each file ingested, write a record to `audit.file_audit` with start/end and loaded rows.  
- For each table step, write to `audit.table_audit` with `rows_in` / `rows_out`.  
- At the end, aggregate `rows_in` vs `rows_out` to validate completeness for the run.

### 5) Inconsistencies Report
- Provide a CSV of inconsistency types discovered (e.g., date format mismatch, numeric parsing issues, null key fields, categorical casing/whitespace).  
- Include *counts of affected rows* per inconsistency; use this to decide what can be auto‑fixed vs. what needs SME input.

## Assumptions
- Source files are pipe-delimited (`|`) and encoded Win1252/CP1252.  
- File names carry a timestamp and are stored in `raw.orders_file.source_file` and `source_file_dt`.  
- Discount is in `[0..1]`. Sales/Quantity are non-negative.  
- Date parsing supports `DD-MM-YYYY` and `MM/DD/YYYY` variants.
- Surrogate keys (`*_sk`) are numeric, natural IDs kept for lineage.

## Suggestions / Next Steps
- Add dbt models for transformations + tests (unique/not null, accepted values).  
- Add Airflow/cron to orchestrate ingestion and publish row-count dashboards.  
- Extend `dim_*` builders (product/customer/geography/shipmode) from staging/MDM.  
- Generate a full calendar `dim_date` spanning data ranges (not only from staging).  
- Enrich quarantine with `review_status`, SME notes, and reprocessing hooks.

## How to Push This Repo
1. Create a new GitHub repo (public or private).  
2. On your machine:
```bash
git init
git branch -M main
git add .
git commit -m "Initial commit: DDL, HLD/ERD, audit/quarantine scaffolding, README"
git remote add origin https://github.com/<your-username>/retail-orders-analytics.git
git push -u origin main
```
3. Update `Task_8_GitHub_Link.txt` with the final GitHub URL and commit again:
```bash
echo "https://github.com/<your-username>/retail-orders-analytics" > Task_8_GitHub_Link.txt
git add Task_8_GitHub_Link.txt
git commit -m "Add Deliverable 7 link file"
git push
```

## Credits
- HLD & flow summary based on the project slides.  
- SQL DDL consolidated from your working scripts (schemas, staging, quarantine, audit, dims/facts).
