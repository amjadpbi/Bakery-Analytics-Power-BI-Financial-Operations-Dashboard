# Bakery Analytics | Automated ETL Pipeline & Financial Dashboard

Power BI project built for a retail bakery franchise using a legacy POS system that exported messy CSV files. The goal was to replace manual cleanup with a repeatable Power Query pipeline and a financial dashboard that could be refreshed without rework.



## Project background

The case study behind this project is **Power BI Bakery Project Pipeline**. It shows a full end-to-end workflow:

1. ingestion folder
2. dynamic folder path
3. cleaning function in Power Query
4. staging and fact tables
5. data modeling
6. dashboard

The source files were inconsistent, duplicated, and difficult to use for reporting. The model was built to handle that kind of file drift while keeping the business layer clean.

## What this project does

This solution automates the path from raw CSV files to a financial dashboard for bakery operations.

It supports:
- revenue tracking
- expense analysis
- cash flow reporting
- profit margin analysis
- payment reconciliation
- payment aging analysis
- month-over-month and year-over-year reporting

## What is included in the repo

- `Model.bim` — semantic model with tables, relationships, and measures
- `DAX_Library.md` — extracted measure library from the model
- `DAX_Measure_Index.md` — measure index grouped by folder
- `Power BI Bakery Project Pipeline.pdf` — case study reference
- `screenshots/` — dashboard and model images
- `README.md` — project summary and GitHub guide

## Model summary

From the provided model:

- Model name: `Bakery_Analytics`
- Compatibility level: `1600`
- Tables: `12`
- Relationships: `11`
- Measures: `50`

## Key features

### Data ingestion and cleaning
- Folder-based ingestion for new CSV files
- Dynamic folder path parameter
- Reusable Power Query cleaning functions
- Header cleanup and footer removal
- Staging layer design to separate raw shaping from business logic

### Data model
- Star schema design
- Dimension and fact table split
- Deterministic surrogate key approach
- One-to-many relationships
- Time intelligence setup with a dedicated date table

### Dashboard layer
- Revenue, expenses, cash flow, and profit margin KPIs
- Payment reconciliation views
- Collection rate and outstanding payment analysis
- Drill-through transaction details
- Month-over-month and year-over-year comparisons
- Interactive filtering by date and business category

## Data model structure

### Fact tables
- `FactSalesHeader` — order-level sales
- `FactSalesLine` — line-level sales detail
- `FactPayment` — payment transactions and status
- `FactExpense` — expense transactions

### Dimension and helper tables
- `DimDate`
- `DimCustomer`
- `DimProduct`
- `DimLocation`
- `DimPaymentMethod`
- `DimExpCategory`
- `_Measures`
- `Payment Aging Categories`

## DAX library

The measure library extracted from the model is included in `DAX_Library.md`. It covers:
- core financial measures
- breakdown measures
- time-series measures
- drill-through measures
- formatting and color helper measures

The measure list is indexed in `DAX_Measure_Index.md`.

## Technical approach

The pipeline is built in layers:

**1. Parameters**  
Dynamic file path management for source data.

**2. Functions**  
Reusable M functions for cleaning, standardizing, and preparing files.

**3. Staging queries**  
Intermediate queries that combine and shape raw files.

**4. Dimension tables**  
Business entities used for slicing and filtering.

**5. Fact tables**  
Transaction tables that feed the report and measures.

This layout keeps refresh logic separate from reporting logic and makes the model easier to maintain.

## Data quality handling

The project handles common issues found in legacy exports:
- inconsistent column names
- junk header rows
- footer totals
- blank rows
- mixed date formats
- currency formatting differences
- missing or invalid values

## How to use the project

1. Open the model in Power BI Desktop.
2. Update the folder parameter to the local source data path.
3. Refresh the model.
4. Review the report pages and drill-through views.

## Recommended GitHub upload structure

```text
Bakery-Analytics/
├─ README.md
├─ Model.bim
├─ DAX_Library.md
├─ DAX_Measure_Index.md
├─ .gitignore
├─ screenshots/
│  ├─ dashboard-main.png
│  ├─ model-view.png
│  └─ drill-through-page.png
└─ docs/
   └─ Power BI Bakery Project Pipeline.pdf
```

## What to keep out of the repo

Do not upload raw source data if it contains customer, payment, or transaction details. If you need sample files for demonstration, sanitize them first.

## Suggested GitHub title

**Bakery Analytics | Automated ETL Pipeline & Financial Dashboard**

That title reads cleaner than a raw file dump and matches the story of the project.
