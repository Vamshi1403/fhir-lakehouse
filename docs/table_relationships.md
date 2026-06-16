# Table Relationships

## Flow
```
FHIR API
   │
   ├── /Patient     → Files/raw/patient/{date}/     → bronze.tbl_patient     → silver.tbl_patient     → gold.vw_dim_patient
   ├── /Encounter   → Files/raw/encounter/{date}/   → bronze.tbl_encounter   → silver.tbl_encounter   → gold.vw_fact_encounter
   ├── /Observation → Files/raw/observation/{date}/ → bronze.tbl_observation → silver.tbl_observation → gold.vw_fact_observation
   └── /Condition   → Files/raw/condition/{date}/   → bronze.tbl_condition   → silver.tbl_condition   → gold.vw_fact_condition
```

## Bronze Schema (all tables)
| Column | Type | Description |
|--------|------|-------------|
| resource_type | string | FHIR resource name |
| resource_id | string | Unique FHIR ID |
| raw_json | string | Full response as JSON string |
| extraction_timestamp | string | When API was called |
| api_url | string | Endpoint hit |
| load_date | string | Partition date |

## Silver Schema (all tables)
| Column | Type | Description |
|--------|------|-------------|
| resource_id | string | Business key |
| ...parsed columns... | string | Resource specific fields |
| extraction_timestamp | string | Carried from bronze |
| load_date | string | Carried from bronze |
| effective_from | string | SCD2 — row valid from |
| effective_to | string | SCD2 — row valid to |
| is_current | boolean | SCD2 — latest version flag |
| row_hash | string | MD5 hash for change detection |

## Gold Views
| View | Source | Filter |
|------|--------|--------|
| gold.vw_dim_patient | silver.tbl_patient | is_current = true |
| gold.vw_fact_encounter | silver.tbl_encounter | is_current = true |
| gold.vw_fact_observation | silver.tbl_observation | is_current = true |
| gold.vw_fact_condition | silver.tbl_condition | is_current = true |

## Resource Specific Fields

### gold.vw_dim_patient
| Column | Source |
|--------|--------|
| patient_id | resource_id |
| gender | p.gender |
| birth_date | p.birthDate |
| deceased | p.deceasedBoolean |
| valid_from | effective_from |

### gold.vw_fact_encounter
| Column | Source |
|--------|--------|
| encounter_id | resource_id |
| status | e.status |
| encounter_class | e.class |
| encounter_date | load_date |
| valid_from | effective_from |

### gold.vw_fact_observation
| Column | Source |
|--------|--------|
| observation_id | resource_id |
| status | o.status |
| effective_date | o.effectiveDateTime |
| subject_ref | o.subject |
| valid_from | effective_from |

### gold.vw_fact_condition
| Column | Source |
|--------|--------|
| condition_id | resource_id |
| clinical_status | c.clinicalStatus |
| verification_status | c.verificationStatus |
| recorded_date | c.recordedDate |
| subject_ref | c.subject |
| valid_from | effective_from |
