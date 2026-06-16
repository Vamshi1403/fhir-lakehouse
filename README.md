# FHIR API Data Ingestion & Analytics

## Architecture
Medallion Lakehouse on Microsoft Fabric

| Layer  | Location | Format |
|--------|----------|--------|
| Raw    | Files/raw/resource/date/ | JSON |
| Bronze | bronze.patient/encounter/observation/condition | Delta |
| Silver | silver.patient/encounter/observation/condition | Delta + SCD2 |
| Gold   | gold.dim_patient/fact_encounter/fact_observation/fact_condition | Views |

## Pipeline
`pl_master_fhir` — runs daily, orchestrates all 3 notebooks in sequence

## Notebooks
| Notebook | Purpose |
|----------|---------|
| 01_bronze_ingest_fhir | Fetch FHIR API → raw JSON → bronze Delta |
| 02_silver_transform_fhir | Parse → dedupe → SCD Type 2 merge |
| 03_gold_views_fhir | Reporting views on silver current records |

## Resources Ingested
- Patient · Encounter · Observation · Condition
- 3 days of data with pagination (50 records/page, 5 pages max)

## SCD Type 2
All silver tables track:
- `effective_from` / `effective_to` / `is_current` / `row_hash`
