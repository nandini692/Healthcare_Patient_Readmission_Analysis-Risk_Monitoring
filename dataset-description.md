# Dataset Description (aligned to readmission_db.sql)

This file documents the actual schema and coverage loaded by Dataset/readmission_db.sql. The SQL dump implements a reporting-style model (dimensions + facts) designed for 30-day readmission analysis and is ready for use with MySQL, Power BI, and Excel.

## Key tables and coverage

- `dim_date`: daily date dimension covering 2019-01-01 through 2022-12-31 (keys in `YYYYMMDD` int format).
- `dim_department`: 10 service lines / departments (Cardiology, Pulmonology, ..., Endocrinology).
- `dim_diagnosis`: ~40 ICD-10 diagnosis codes with description and diagnosis_group.
- `dim_insurance`: 9 payer entries (Medicare, Medicaid variants, major private insurers, Self-pay).
- `dim_patient`: synthetic patient master (approximately 7,000 patients; unique `medical_record_number`).
- `fact_admission`: admissions fact (primary row per inpatient encounter). The dump contains admissions through at least `admission_id` ~11066 (~11k admissions).
- `bridge_admission_diagnosis`: many-to-many bridge linking admissions to multiple diagnoses.
- `fact_readmission`: readmission links between an index admission and a later admission with `days_to_readmit`, `readmission_within_30` flag and a categorical `readmission_reason_category`.

## Important columns (mapping to the earlier recommended fields)

- PatientID: `dim_patient.patient_id` (also `medical_record_number` available as `medical_record_number`).
- AdmissionID: `fact_admission.admission_id`.
- AdmissionDate / DischargeDate: linked via `fact_admission.admission_date_key` and `fact_admission.discharge_date_key` -> `dim_date.date`.
- ReadmissionFlag: `fact_readmission.readmission_within_30` (TINYINT 0/1).
- ReadmissionType: `fact_readmission.readmission_type` (Planned / Unplanned).
- ReadmissionReasonCategory: `fact_readmission.readmission_reason_category` (enumerated list in SQL).
- PrimaryDiagnosisCode: `fact_admission.primary_diagnosis_code` (FK -> `dim_diagnosis.diagnosis_code`).
- SecondaryDiagnosisCodes / multiple diagnoses: `bridge_admission_diagnosis` (`admission_id`, `diagnosis_code`, `is_primary`).
- ComorbidityCount: `fact_admission.comorbidity_count`.
- LengthOfStay: `fact_admission.length_of_stay` (days).
- DischargeDisposition: `fact_admission.discharge_disposition` (Home, Skilled Nursing, Death, Other).
- InsuranceType: `fact_admission.insurance_id` -> `dim_insurance.insurance_name` / `payer_type`.
- AgeGroup / Gender: `dim_patient.age_group`, `dim_patient.gender`.
- ServiceLine / Department: `fact_admission.service_line_id` -> `dim_department.department_id` / `department_name`.
- SeverityLevel: `fact_admission.severity_level` (Low / Medium / High).
- FollowUpScheduled / FollowUpDays: `fact_admission.follow_up_scheduled`, `fact_admission.follow_up_days`.
- MedicationReconciliationDone: `fact_admission.medication_reconciliation_done`.
- SocialSupportFlag: `dim_patient.social_support_flag`.
- ZipCode / Region: `dim_patient.zip_code`, `dim_patient.region`.
- AdmissionSource: `fact_admission.admission_source` (ED, Transfer, Clinic, Outpatient).

## Enumerations and categories (examples from the SQL dump)

- `admission_source`: 'ED','Transfer','Clinic','Outpatient'.
- `discharge_disposition`: 'Home','Skilled Nursing','Death','Other'.
- `severity_level`: 'Low','Medium','High'.
- `payer_type` (in `dim_insurance`): 'Medicare','Medicaid','Private','Self-pay'.
- `readmission_reason_category`: includes values such as 'Heart Failure Exacerbation','Respiratory Distress','Infection / Sepsis','Renal Failure','GI Complication','Medication Non-compliance','Surgical Complication','Metabolic Imbalance','Neurological Event','Other'.

## Notes on suitability for Excel / Power BI / SQL

- The model is intentionally star-like and analytic-ready: facts use integer PK/FK columns and dimensions store human-readable labels — ideal for joins in Power BI or Excel via ODBC/CSV extracts.
- Date grain is daily and mapped to `dim_date`, enabling time-series slicing in Power BI using the date key.
- Indexes and foreign keys are present in the dump (useful for testing SQL performance and ensuring referential integrity when imported into MySQL).

## Suggested next steps when using this dump

- Load `Dataset/readmission_db.sql` into a local MySQL instance (InnoDB, utf8mb4) and confirm row counts for `fact_admission` and `fact_readmission`.
- Build a simple star-schema view in SQL that flattens `fact_admission` + patient + primary diagnosis + insurance + date for easy Power BI import.
- Create a readmission-level view joining `fact_readmission` -> `fact_admission` (index and readmit admissions) to analyze days-to-readmit and reason categories.

If you want, I can update the repository README with a short import and verification SQL script and create the two recommended views for Power BI import.