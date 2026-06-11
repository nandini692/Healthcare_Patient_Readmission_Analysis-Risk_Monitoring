# Dataset Description

This dataset is designed to support a patient readmission analytics and risk monitoring dashboard. The goal is to capture the data needed to understand why patients return within 30 days and provide actionable insights using Excel, SQL, and Power BI.

## Recommended Dataset Structure

- `PatientID`
- `AdmissionID`
- `AdmissionDate`
- `DischargeDate`
- `ReadmissionDate` (if within 30 days)
- `ReadmissionFlag` (yes/no within 30 days)
- `ReadmissionType` (planned / unplanned)
- `PrimaryDiagnosisCode`
- `SecondaryDiagnosisCodes`
- `ComorbidityCount`
- `LengthOfStay`
- `DischargeDisposition`
- `InsuranceType`
- `AgeGroup`
- `Gender`
- `ServiceLine` / `Department`
- `SeverityLevel`
- `FollowUpAppointmentScheduled` (yes/no)
- `FollowUpDays` (days to first post-discharge follow-up)
- `MedicationReconciliationDone` (yes/no)
- `SocialSupportFlag` (if available)
- `ZipCode` / `Region`
- `AdmissionSource` (ED, transfer, clinic)
- `ReadmissionReasonCategory`

## Why This Dataset Works

- It aligns directly with the problem statement: identifying factors associated with 30-day readmissions.
- It supports analysis by demographics, diagnosis, department, and care transition processes.
- It enables KPI calculations in Power BI without requiring predictive modeling.
- It facilitates process insights, such as the relationship between follow-up scheduling and readmission outcomes.

## Best Data Sources

- Hospital Admission, Discharge, and Transfer (ADT) system
- Electronic Health Record (EHR) admission and discharge logs
- Clinical documentation for diagnosis and severity details
- Scheduling systems for follow-up appointments
- Billing or claims data for insurance type and planned/unplanned status
- Quality or case management records for care-transition metrics

## Reference Ideas

Use concepts from established readmission analytics sources such as:

- CMS Hospital Readmission Reduction Program (HRRP) metrics
- AHRQ readmission quality measures
- LACE score component definitions for risk-factor reference
- Hospital dashboards built from admission/discharge tables, patient demographics, and transition process metrics

## Dataset Use Case

This dataset should be used as a raw collection or staging reference. It is intended for:

- data collection planning
- dataset design and preparation
- SQL-based aggregation and transformation
- Power BI dashboarding and reporting

This approach ensures the project remains a focused data analytics effort, not a machine learning or AI implementation.