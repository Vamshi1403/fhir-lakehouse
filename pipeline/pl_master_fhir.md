# Pipeline — pl_master_fhir

## Overview
Orchestrates all 3 notebooks daily in sequence.

## Activities
```
Bronze_Ingest →(on success)→ Silver_Transform →(on success)→ Gold_Views
```

## Activity Details
| Activity | Notebook | Purpose |
|----------|----------|---------|
| Bronze_Ingest | 01_bronze_ingest_fhir | Fetch FHIR API, save raw JSON, load bronze Delta tables |
| Silver_Transform | 02_silver_transform_fhir | Parse, deduplicate, SCD Type 2 merge into silver |
| Gold_Views | 03_gold_views_fhir | Create reporting views from silver current records |

## Schedule
- Frequency: Daily
- Trigger: Scheduled

## Execution Order
Patient → Encounter → Observation → Condition
(controlled by RESOURCES list in 00_config)

## Error Handling
- Each activity only runs if previous succeeds (on success dependency)
- Per-resource try/except in bronze notebook logs errors without stopping pipeline
