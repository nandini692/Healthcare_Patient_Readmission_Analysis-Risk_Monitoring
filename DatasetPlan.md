# Dataset Plan

## Purpose
Define the ideal dataset structure for a hospital patient readmission analytics project. This dataset is intended for Excel-based collection and Power BI visualization.

## Dataset Goal
Capture patient admission, discharge, and follow-up data that supports analysis of 30-day unplanned readmissions.

## Recommended Dataset Structure
Use separate Excel sheets or tables for each primary data group:
- `Patient_Master`
- `Admissions`
- `Readmissions`
- `FollowUp_Tracking`

### 1. Patient Details
- Patient ID (de-identified or coded)
- Age
- Gender
- Insurance type / payer
- Zip code or service region
- Primary diagnosis group

### 2. Index Admission
- Admission date
- Discharge date
- Discharge department / service line
- Length of stay
- Admission type (elective / emergency)
- Discharge disposition (home, skilled nursing, home health)

### 3. Readmission Information
- Readmission flag (yes / no within 30 days)
- Readmission date
- Readmission reason or diagnosis
- Time to readmission (days)
- Planned vs unplanned status

### 4. Clinical / Risk Factors
- Comorbidities count or list
- Key conditions such as heart failure, COPD, pneumonia, diabetes, renal disease
- Number of prior admissions in the past 6–12 months
- Risk score proxy fields, such as:
  - number of comorbidities
  - prior hospital use
  - age group
  - discharge complexity

### 5. Follow-up / Process Measures
- Follow-up appointment scheduled (yes / no)
- Follow-up appointment date
- Medication reconciliation completed (yes / no)
- Discharge education documented (yes / no)

### 6. Operational Context
- Department / service line
- Attending physician or care team
- Admission source (ED, transfer, clinic)
- Readmission cost estimate or penalty flag (optional)

## Data Collection Reference
- Use a synthetic sample or hospital discharge registry-style dataset
- Model data after EHR admission/discharge logs and care transition tracking spreadsheets
- Keep the data structure simple and easy to maintain in Excel

## Why this dataset works
- Supports the core question: why are patients returning within 30 days?
- Keeps the project analytic rather than predictive
- Enables the following dashboard insights:
  - 30-day readmission rate
  - readmission counts by department and condition
  - high-risk patient counts
  - follow-up compliance metrics
