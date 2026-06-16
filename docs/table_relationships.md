## Resource Specific Fields

### gold.vw_dim_patient
| Column | Source Field in FHIR | Description |
|--------|---------------------|-------------|
| patient_id | resource_id | Unique patient identifier |
| gender | gender | Patient gender |
| birth_date | birthDate | Date of birth |
| deceased | deceasedBoolean | Whether patient is deceased |
| valid_from | effective_from | SCD2 valid from timestamp |

### gold.vw_fact_encounter
| Column | Source Field in FHIR | Description |
|--------|---------------------|-------------|
| encounter_id | resource_id | Unique encounter identifier |
| status | status | Encounter status |
| encounter_class | class | Type of encounter |
| encounter_date | load_date | Date of encounter load |
| valid_from | effective_from | SCD2 valid from timestamp |

### gold.vw_fact_observation
| Column | Source Field in FHIR | Description |
|--------|---------------------|-------------|
| observation_id | resource_id | Unique observation identifier |
| status | status | Observation status |
| effective_date | effectiveDateTime | When observation was taken |
| subject_ref | subject | Reference to patient |
| valid_from | effective_from | SCD2 valid from timestamp |

### gold.vw_fact_condition
| Column | Source Field in FHIR | Description |
|--------|---------------------|-------------|
| condition_id | resource_id | Unique condition identifier |
| clinical_status | clinicalStatus | Active/resolved/inactive |
| verification_status | verificationStatus | Confirmed/unconfirmed |
| recorded_date | recordedDate | When condition was recorded |
| subject_ref | subject | Reference to patient |
| valid_from | effective_from | SCD2 valid from timestamp |
