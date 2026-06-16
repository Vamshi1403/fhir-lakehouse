# Table Relationships

## Pipeline Flow
```
FHIR API
   │
   ├── /Patient     → Files/raw/patient/{date}/response.json     → bronze.tbl_patient     → silver.tbl_patient     → gold.vw_dim_patient
   ├── /Encounter   → Files/raw/encounter/{date}/response.json   → bronze.tbl_encounter   → silver.tbl_encounter   → gold.vw_fact_encounter
   ├── /Observation → Files/raw/observation/{date}/response.json → bronze.tbl_observation → silver.tbl_observation → gold.vw_fact_observation
   └── /Condition   → Files/raw/condition/{date}/response.json   → bronze.tbl_condition   → silver.tbl_condition   → gold.vw_fact_condition
```

## Layer Descriptions

| Layer | Schema | Format | Purpose |
|-------|--------|--------|---------|
| Raw | Files/raw/ | JSON | Store full API response as-is, partitioned by date |
| Bronze | bronze | Delta, append-only | Preserve raw_json as string, add metadata columns |
| Silver | silver | Delta, SCD Type 2 | Parse and clean fields, track historical changes |
| Gold | gold | Views | Reporting-ready, current records only |

---

## Bronze Schema (all tables)

| Column | Type | Description |
|--------|------|-------------|
| resource_type | string | FHIR resource name (Patient, Encounter etc.) |
| resource_id | string | Unique FHIR resource ID |
| raw_json | string | Full API response stored as JSON string |
| extraction_timestamp | string | When the API was called |
| api_url | string | FHIR endpoint that was hit |
| load_date | string | Date of ingestion — partition key |

---

## Silver Schema (all tables)

| Column | Type | Description |
|--------|------|-------------|
| resource_id | string | Business key — unique FHIR resource ID |
| ...resource columns... | string | Parsed fields specific to each resource |
| extraction_timestamp | string | Carried from bronze |
| load_date | string | Carried from bronze |
| effective_from | string | SCD2 — when this version became active |
| effective_to | string | SCD2 — when closed (9999-12-31 = still current) |
| is_current | boolean | SCD2 — true means latest active version |
| row_hash | string | MD5 hash of all non-key columns for change detection |

---

## Gold Views

| View | Source Table | Filter Applied |
|------|-------------|----------------|
| gold.vw_dim_patient | silver.tbl_patient | is_current = true |
| gold.vw_fact_encounter | silver.tbl_encounter | is_current = true |
| gold.vw_fact_observation | silver.tbl_observation | is_current = true |
| gold.vw_fact_condition | silver.tbl_condition | is_current = true |

---

## Resource Specific Fields

### gold.vw_dim_patient
| Column | Source Field in FHIR | Description |
|--------|---------------------|-------------|
| patient_id | resource_id | Unique patient identifier |
| gender | gender | Patient gender |
| birth_date | birthDate | Date of birth |
| deceased | deceasedBoolean | Whether patient is deceased |
| valid_from | effective_from | SCD2 valid from timestamp |
| is_current | is_current | Always true in gold view |

### gold.vw_fact_encounter
| Column | Source Field in FHIR | Description |
|--------|---------------------|-------------|
| encounter_id | resource_id | Unique encounter identifier |
| status | status | Encounter status (finished, in-progress etc.) |
| encounter_class | class | Type of encounter (inpatient, outpatient etc.) |
| encounter_date | load_date | Date of encounter load |
| valid_from | effective_from | SCD2 valid from timestamp |

### gold.vw_fact_observation
| Column | Source Field in FHIR | Description |
|--------|---------------------|-------------|
| observation_id | resource_id | Unique observation identifier |
| status | status | Observation status (final, preliminary etc.) |
| effective_date | effectiveDateTime | When observation was taken |
| subject_ref | subject | Reference to the patient |
| load_date | load_date | Date of ingestion |
| valid_from | effective_from | SCD2 valid from timestamp |

### gold.vw_fact_condition
| Column | Source Field in FHIR | Description |
|--------|---------------------|-------------|
| condition_id | resource_id | Unique condition identifier |
| clinical_status | clinicalStatus | Active, resolved, or inactive |
| verification_status | verificationStatus | Confirmed or unconfirmed |
| recorded_date | recordedDate | When condition was recorded |
| subject_ref | subject | Reference to the patient |
| load_date | load_date | Date of ingestion |
| valid_from | effective_from | SCD2 valid from timestamp |

---

## SCD Type 2 Logic

Silver tables implement Slowly Changing Dimension Type 2 to track changes over time.

| Scenario | Action |
|----------|--------|
| New resource_id | Insert as current (is_current = true) |
| Same resource_id, same data | Skip — no change detected |
| Same resource_id, data changed | Close old row (is_current = false, effective_to = now), insert new row as current |

Change detection uses `row_hash` — MD5 of all non-key columns. If hash differs between runs, the record has changed.

---

## Naming Conventions

| Object | Prefix | Example |
|--------|--------|---------|
| Bronze tables | tbl_ | bronze.tbl_patient |
| Silver tables | tbl_ | silver.tbl_patient |
| Gold views | vw_ | gold.vw_dim_patient |

---

## Config Reference (00_config)

All table and view names are generated by helper functions in `00_config` — no hardcoding in any notebook.

| Function | Returns | Example |
|----------|---------|---------|
| bronze_table(resource) | bronze schema table name | bronze.tbl_patient |
| silver_table(resource) | silver schema table name | silver.tbl_patient |
| gold_view(resource) | gold schema view name | gold.vw_dim_patient |
| raw_path(resource, date) | raw file path | Files/raw/patient/2026-06-16/response.json |
