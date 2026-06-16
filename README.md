# FHIR API Data Ingestion & Analytics
Medallion Lakehouse pipeline on Microsoft Fabric — config-driven, incremental, SCD Type 2

## Architecture
```
FHIR API (hapi.fhir.org)
     ↓
Raw Layer     → Files/raw/{resource}/{date}/response.json
     ↓
Bronze Layer  → bronze.tbl_{resource}     (Delta, append-only)
     ↓
Silver Layer  → silver.tbl_{resource}     (Delta, SCD Type 2)
     ↓
Gold Layer    → gold.vw_{dim/fact}_{resource}  (Views)
```

## Naming Conventions
| Object | Prefix | Example |
|--------|--------|---------|
| Tables | `tbl_` | `bronze.tbl_patient` |
| Views  | `vw_`  | `gold.vw_dim_patient` |

## Notebooks
| Notebook | Purpose |
|----------|---------|
| `00_config` | Central config — all settings, naming helper functions |
| `01_bronze_ingest_fhir` | Fetch FHIR API → save raw JSON → append to bronze Delta |
| `02_silver_transform_fhir` | Parse raw_json → deduplicate → SCD Type 2 merge |
| `03_gold_views_fhir` | Create reporting views from silver current records |

## Pipeline — `pl_master_fhir`
Runs daily, orchestrates notebooks in this order:
```
Bronze_Ingest →(success)→ Silver_Transform →(success)→ Gold_Views
```
Each notebook runs `%run 00_config` at start — no hardcoding anywhere.

## Table Inventory
| Bronze | Silver | Gold |
|--------|--------|------|
| `bronze.tbl_patient` | `silver.tbl_patient` | `gold.vw_dim_patient` |
| `bronze.tbl_encounter` | `silver.tbl_encounter` | `gold.vw_fact_encounter` |
| `bronze.tbl_observation` | `silver.tbl_observation` | `gold.vw_fact_observation` |
| `bronze.tbl_condition` | `silver.tbl_condition` | `gold.vw_fact_condition` |

## Metadata Columns — Bronze (all tables)
| Column | Description |
|--------|-------------|
| `resource_type` | FHIR resource name
