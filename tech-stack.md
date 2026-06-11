# Project Tech Stack

This project is a pure data analytics solution for patient readmission analysis and risk monitoring. It uses only the following technologies:

- **Microsoft Excel**
- **Power BI**
- **SQL**

## Purpose of Each Technology

### Excel
- Raw data staging and source data templates
- Manual data cleaning, review, and validation
- Data dictionaries and metadata documentation
- Quick pivot table checks for dataset consistency

### Power BI
- Main analytics and reporting environment
- Data modeling using Power Query and Power Pivot
- Creation of interactive dashboards and drill-through reports
- KPI cards, line charts, bar charts, and trend analysis

### SQL
- Data extraction and transformation from source systems
- Building reusable views for:
  - admissions and discharges
  - readmission windows
  - patient cohorts
  - rate and trend aggregations
- Supporting the dataset for Power BI ingestion

## Supporting Environment

- `Excel` workbook as the raw staging and collection vehicle
- `Power BI Desktop` for report development
- `SQL Server`, `Azure SQL`, or another relational database for structured dataset storage
- `Power Query` in Excel and Power BI for ETL work
- `DAX` measures in Power BI for computed KPI calculations

## Explicit Constraints

This project is intentionally limited to descriptive analytics and monitoring. It must avoid:

- `Python`
- `Machine Learning`
- any AI modeling or predictive modeling tools

The focus is on dashboard-driven analysis, not algorithmic prediction.
