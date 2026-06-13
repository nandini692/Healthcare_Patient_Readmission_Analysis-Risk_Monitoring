# End-to-End Implementation Guide — Patient 30‑Day Readmission Analytics

This guide maps the project from data understanding through delivery of stakeholder-ready dashboards using MySQL, Power BI, and Excel. It is written for BI analysts, data engineers, and care-quality stakeholders.

1. Project Understanding

- Business problem: Reduce avoidable 30‑day hospital readmissions by identifying clinical, demographic, and process drivers and recommending operational interventions.
- Objectives: measure baseline readmit rate, identify high-risk cohorts, quantify process gaps (follow-up, med reconciliation), deliver dashboards for execs and operations, and recommend prioritized actions.
- Stakeholders: Quality & Case Mgmt, Care Coordination, Clinical Leads, Finance, Operations, Executive Sponsor.
- Success criteria: reproducible data import from `Dataset/readmission_db.sql`, validated KPIs in Power BI, actionable prioritized recommendations with owners and expected impact.

2. Dataset Assessment

- Files to verify: `Dataset/readmission_db.sql`; reference doc: `dataset-description.md`.
- Schema summary (from SQL dump): `dim_date`, `dim_patient` (~7k), `dim_diagnosis` (~40 codes), `dim_department` (10), `dim_insurance` (9), `fact_admission` (~11k rows), `fact_readmission`, `bridge_admission_diagnosis`.
- Immediate checks after import:
  - Row counts: `SELECT COUNT(*) FROM fact_admission; SELECT COUNT(*) FROM fact_readmission;`
  - Referential integrity: find orphan facts with LEFT JOIN to dims.
  - Date range: `SELECT MIN(admission_date_key), MAX(admission_date_key) FROM fact_admission;`
- Data quality risks to expect: NULLs in optional follow-up fields, duplicate admissions, outlier LOS / days_to_readmit, diagnosis code formatting.

3. Data Cleaning & Transformation Plan

- Principles: keep raw load immutable; implement all cleaning/derivations as SQL views in a `vw_*` pattern so Power BI imports stable flattened views.
- Staging objects to create:
  - `stg_invalid_admissions` — records failing critical validation (missing patient_id, invalid date keys, discharge < admission).
  - `vw_admission_flat` — single-row flattened admission view for reporting.
  - `vw_readmission_summary` — readmission-level view linking index→readmit admissions with metrics.
- Key cleaning steps (SQL):
  - De-duplicate:
    - Keep latest `updated_at` per (`admission_id`) or per (`patient_id`, `admission_date_key`).
  - Date validation:
    - Move admissions where `admission_date_key > discharge_date_key` to `stg_invalid_admissions`.
  - FK checks:
    - LEFT JOIN admissions to dims; insert `Unknown` rows into dims or flag for ETL fix.
  - Normalize codes:
    - `UPPER(TRIM(primary_diagnosis_code))` and validate against `dim_diagnosis`.
  - Outlier flagging:
    - `length_of_stay > 120` or `days_to_readmit > 365` → flagged for review.
- Derived columns (in `vw_admission_flat`):
  - `admission_date`, `discharge_date` (from `dim_date`);
  - `age_at_admission` = TIMESTAMPDIFF(YEAR, dim_patient.dob, admission_date);
  - `is_readmit_30` (from `fact_readmission` or computed);
  - `primary_dx_group` (from `dim_diagnosis.diagnosis_group`);
  - `follow_up_within_7` = follow_up_scheduled=1 AND follow_up_days <=7;
  - `risk_score_proxy` = weighted rule-based score (example: comorbidity_count + CASE severity_level WHEN 'High' THEN 2 WHEN 'Medium' THEN 1 ELSE 0 END).

Example view skeleton:

```sql
CREATE OR REPLACE VIEW vw_admission_flat AS
SELECT fa.admission_id,
       fa.patient_id,
       dp.medical_record_number,
       dd_admit.date AS admission_date,
       dd_disc.date AS discharge_date,
       TIMESTAMPDIFF(YEAR, dp.dob, dd_admit.date) AS age_at_admission,
       dp.age_group,
       dp.gender,
       fa.primary_diagnosis_code,
       d.diagnosis_group AS primary_dx_group,
       fa.length_of_stay,
       fa.comorbidity_count,
       fa.severity_level,
       fa.follow_up_scheduled,
       fa.follow_up_days,
       (fa.follow_up_scheduled = 1 AND fa.follow_up_days <= 7) AS follow_up_within_7,
       fa.insurance_id,
       i.payer_type,
       fa.service_line_id,
       dept.department_name
FROM fact_admission fa
JOIN dim_patient dp ON fa.patient_id = dp.patient_id
JOIN dim_date dd_admit ON fa.admission_date_key = dd_admit.date_key
JOIN dim_date dd_disc  ON fa.discharge_date_key = dd_disc.date_key
LEFT JOIN dim_diagnosis d ON fa.primary_diagnosis_code = d.diagnosis_code
LEFT JOIN dim_insurance i ON fa.insurance_id = i.insurance_id
LEFT JOIN dim_department dept ON fa.service_line_id = dept.department_id;
```

4. Exploratory Data Analysis (10–12 Business Questions)

For each question below run an initial SQL metric, then visualize in Power BI. Example SQL snippets can be provided on request.

Q1 — What is the baseline 30‑day readmission rate and its monthly trend?
Q2 — Which departments have the highest readmission rates and volumes?
Q3 — Which primary diagnosis groups account for most readmissions?
Q4 — How does readmission rate vary by discharge disposition?
Q5 — Does payer type correlate with readmission risk after age-adjustment?
Q6 — What is the impact of scheduling follow-up within 7 days on readmit risk?
Q7 — Which ZIP codes / regions have elevated readmission rates?
Q8 — Who are repeat utilizers (patients with multiple admissions and readmits)?
Q9 — What is the distribution of days-to-readmit and top reason categories within 30 days?
Q10 — Are there seasonal patterns in readmissions (month-of-year effects)?
Q11 — How does comorbidity_count relate to readmit probability?
Q12 — Operational: average LOS and readmit risk by admission source (ED vs Outpatient).

For every question produce:
- Objective, SQL approach, visualization type, interpretation, and next-action recommendation.

5. Visualizations & Analytical Approach

- Build a visualization catalog mapping each question to chart type (line, bar, box, histogram, heatmap, KPI card). Prioritize pre-aggregated SQL views for heavy joins.
- Power BI performance tips:
  - Import `vw_admission_flat` and `vw_readmission_summary` rather than raw joins.
  - Create a small `Dim_Date` table from `dim_date` and mark it as date table in Power BI.
  - Use incremental refresh when moving from dev to production (requires Power BI Premium or dataset partitioning).

6. Key Insights (deliver with supporting evidence)

- Structure each insight into Observation → Why → Business Meaning → Recommended Action → Owner → Est. Impact.
- Example template row in findings deliverable:
  - Observation: 30‑day readmit rate = X% (Y admissions) concentrated in Cardiology and Heart Failure.
  - Why: low `follow_up_within_7` and elevated comorbidity_count.
  - Action: implement automatic 7-day follow-up scheduling for HF discharges; owner: Case Mgmt; priority: High.

7. Hypothesis Testing (Optional)

- H1: Early follow-up reduces 30‑day readmission (two-proportion z-test).
- H2: LOS differs between readmitted and non-readmitted index admissions (t-test or Mann-Whitney if skewed).
- Provide SQL to extract contingency tables and Python/R snippets only if allowed — otherwise use Excel (CHI-SQ) or Power BI (Decomposition tree + percentile tests).

8. KPI Framework

- Strategic KPIs: 30‑Day Readmission Rate, Readmission Cost Estimate.
- Operational KPIs: % Follow-up within 7 days, Medication Reconciliation Rate, % Discharged to SNF.
- Performance KPIs: Avg LOS, Readmit Rate by Department, Top High-risk ZIPs.
- Calculation examples included as SQL and as DAX-ready measure pseudo-code (available on request).

9. Multi-Page Report Design

- Executive Summary: KPI row, 90‑day trend, top 3 drivers, prioritized actions.
- Data Overview: row counts, date range, data quality exceptions.
- KPI Overview: KPIs with trend and period-over-period deltas.
- Trend Analysis: monthly and seasonal patterns.
- Segmentation: by age group, payer, diagnosis group, region.
- Performance: LOS, throughput, admission source analysis.
- Root Cause: deep-dive into top diagnoses and cohorts.
- Insights & Recommendations: prioritized actions, owners, and timeline.
- Appendix: Data dictionary and SQL snippets.

10. Dashboard Design & Wireframes

- I will prepare wireframes when requested. Minimal wireframe guidance here:
  - Top: report title + period selector + export.
  - Left: global slicers (Date, Dept, Payer, Age Group, Region).
  - Body: KPI row → trend visuals → top drivers (bar/treemap) → table of top N patients/zip codes.
  - Drill-through: patient-level timeline page showing index → readmit visits.

11. Insight → Impact → Recommendation Mapping

- Deliver a prioritized table mapping each insight to expected impact (qualitative or numeric), required owners, and estimated effort (Low/Med/High). This becomes the project action register.

12. Executive Summary

- One-page snapshot: baseline readmit rate, top 2 drivers, 3 recommended pilots, ask for resources and owners.

13. Final Business Recommendations

- Short-term (0–3 mo): import and validate dump; create `vw_admission_flat` and `vw_readmission_summary`; build executive snapshot.
- Medium (3–6 mo): pilot 7-day follow-up automation for top cohort; run case reviews.
- Long-term (6–12 mo): operationalize monitoring, integrate alerts, measure pilot ROI.

14. End-to-End Implementation Roadmap (high level)

- Phase 0: Import & Verify (0–1 day)
  - Commands:
  ```powershell
  mysql -u root -p < Dataset/readmission_db.sql
  mysql -u root -p -e "USE readmission_db; SELECT COUNT(*) FROM fact_admission;"
  ```
- Phase 1: Staging & Views (1–3 days)
  - Create `stg_invalid_admissions`, `vw_admission_flat`, `vw_readmission_summary`.
- Phase 2: EDA & Prototyping (2–4 days)
  - Run queries for the 12 questions, prototype visuals in Power BI desktop.
- Phase 3: Build & Validate Dashboards (3–5 days)
  - Implement KPIs, deploy PBIX to Power BI Service, configure refresh and access.
- Phase 4: Pilot & Iterate (4–12 weeks)
  - Run interventions, measure effect, iterate.

---

If you want I will:
- create the two SQL views (`vw_admission_flat`, `vw_readmission_summary`) and an `import_verify.sql` script now, or
- scaffold Power BI wireframe images / a starter PBIX structure.

Which of the above should I do next? (I can implement the SQL views now.)
