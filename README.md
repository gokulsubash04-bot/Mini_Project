# Climate Change Analysis: Global Monthly Temperature Anomalies (1850–2024)

[![Power BI](https://img.shields.io/badge/Power_BI-Desktop_2026-F2C811?style=flat&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![Dataset](https://img.shields.io/badge/Dataset-Berkeley_Earth-176B87?style=flat)](https://berkeleyearth.org/)
[![Status](https://img.shields.io/badge/Status-Completed-22C55E?style=flat)]()

An executive Power BI analytics report investigating 175 years of global surface temperature anomalies (1850 to 2024), tracking long-term climate trajectories, decadal warming escalation, and seasonal variations.

---

## 📌 Table of Contents
1. [Project Overview](#-project-overview)
2. [Dataset Description](#-dataset-description)
3. [Data Cleaning & ETL Pipeline](#-data-cleaning--etl-pipeline)
4. [Data Model & Semantic Schema](#-data-model--semantic-schema)
5. [DAX Measures Reference](#-dax-measures-reference)
6. [Dashboard Architecture & Layout](#-dashboard-architecture--layout)
7. [Key Analytical Insights & Findings](#-key-analytical-insights--findings)
8. [Project File Inventory](#-project-file-inventory)
9. [How to Run & Export](#-how-to-run--export)

---

## 🌍 Project Overview

### Problem Statement
Global surface temperatures have demonstrated notable variability and accelerated warming since the Industrial Revolution. This project analyzes historical monthly temperature anomalies across 175 continuous years (1850–2024) to provide clear answers to critical environmental and policy questions:
- What is the long-term trend of Earth's temperature anomaly compared to the mid-20th century baseline?
- At what rate has global warming accelerated across recent decades?
- Are global temperature anomalies uniform across calendar months, or is warming concentrated in specific seasons?
- What are the historic high and low extreme anomalies recorded in modern history?

---

## 📊 Dataset Description

The analysis uses the **Berkeley Earth Land/Ocean Temperature Record**, synthesized by combining Berkeley Earth land-surface observations with the HadSST ocean temperature field.

- **Temporal Coverage:** January 1850 to December 2024 (175 Years / 2,100 unique monthly observations).
- **Baseline Reference:** 1951–1980 global mean surface temperature (anomalies represent deviations above or below this baseline in °C).
- **Key Columns:**
  - `Year`, `Month`, `Date`
  - `Monthly Anomaly (°C)`, `Monthly Uncertainty (°C)`
  - `Annual Anomaly (°C)`, `Annual Uncertainty (°C)`
  - `Five-Year`, `Ten-Year`, and `Twenty-Year` smoothed anomalies and uncertainty bounds.

---

## 🧹 Data Cleaning & ETL Pipeline

Data ingestion and transformation were performed using **Power Query (M)**:

1. **Deduplication of Appended Records:**
   - Raw source records contained duplicate blocks (4,200 rows).
   - Applied deduplication on `[Year]` and `[Month]` to isolate exactly **2,100 unique monthly records**.
2. **Missing Value Integrity:**
   - Missing anomaly values were preserved as strict `null` values and **never imputed as zero**, preventing artificial skewing of mean calculations.
3. **Data Type Standardization:**
   - `Year` and `Month`: Whole Number (`Int64`).
   - Temperature anomaly and uncertainty metrics: Decimal Number (`Double`).
   - `Date`: Formatted as standard date sequence starting on the first day of each month.
4. **Calculated Grouping Attributes:**
   - Created `Decade = Number.RoundDown([Year] / 10) * 10` for decadal aggregations (e.g., 1850s, 1980s, 2020s).
   - Configured custom 0.1°C bins on `Monthly Anomaly (°C)` for the distribution histogram.

---

## 📐 Data Model & Semantic Schema

The data model follows an optimized tabular structure built within `mini_project_final.SemanticModel`:

| Column / Measure | Type | Role |
| :--- | :--- | :--- |
| `Year` | Dimension (Int64) | Temporal filter & annual trend x-axis |
| `Month` | Dimension (Int64) | Seasonal cycle axis (1 to 12) |
| `Date` | Dimension (String/Date) | Continuous chronological timeline |
| `Monthly Anomaly (°C)` | Metric (Double) | Core temperature deviation variable |
| `Monthly Uncertainty (°C)`| Metric (Double) | 95% confidence interval bound |
| `Decade` | Dimension (String) | Decadal cohort grouping |
| `Monthly Anomaly (bins)` | Calculated (Double)| 0.1°C frequency distribution bins |

---

## 🧮 DAX Measures Reference

The semantic model includes core DAX measures powering KPI cards and dynamic aggregations:

### 1. Average Temperature Anomaly (°C)
```dax
Average Temperature Anomaly =
AVERAGE('Cleaned_Temperature_Data'[Monthly Anomaly (°C)])
```
*Calculates the overall average monthly temperature deviation.*

### 2. Highest Temperature Anomaly (°C)
```dax
Highest Temperature Anomaly =
MAX('Cleaned_Temperature_Data'[Monthly Anomaly (°C)])
```
*Identifies the single hottest month anomaly in historical records (+1.548°C in Sep 2023).*

### 3. Lowest Temperature Anomaly (°C)
```dax
Lowest Temperature Anomaly =
MIN('Cleaned_Temperature_Data'[Monthly Anomaly (°C)])
```
*Identifies the coldest month anomaly in recorded history (-1.004°C in Jan 1861).*

### 4. Total Monthly Records
```dax
Monthly Records =
COUNT('Cleaned_Temperature_Data'[Monthly Anomaly (°C)])
```
*Ensures 100% data completeness (2,100 verified records).*

### 5. Annual Average Anomaly (°C)
```dax
Annual Average Anomaly =
AVERAGEX(
    VALUES('Cleaned_Temperature_Data'[Year]),
    CALCULATE(AVERAGE('Cleaned_Temperature_Data'[Monthly Anomaly (°C)]))
)
```

---

## 🖥️ Dashboard Architecture & Layout

The dashboard is structured into a **1920 × 1080 (16:9 Full HD)** grid, eliminating visual overlaps and adhering to Microsoft Fluent 2 design principles:

```
┌─────────────────────────────────────────────────────────────┬──────────────────┐
│  CLIMATE CHANGE ANALYSIS (Executive Banner)                 │ Filter By Year   │ y: 20–105
│  Global Monthly Temperature Anomalies (1850 – 2024)         │ [ Dropdown ]     │
├───────────────────┬───────────────────┬─────────────────────┼──────────────────┤
│ Avg Temp Anomaly  │ Max Temp Anomaly  │ Min Temp Anomaly    │ Monthly Records  │ y: 120–235
│   +0.01 °C        │   +1.55 °C        │   -1.00 °C          │     2,100        │
├───────────────────┴───────────────────┴─────────────────────┴──────────────────┤
│                                                                                │
│  Global Temperature Anomaly Trend (1850 – 2024) [ Full Width Hero Line Chart ] │ y: 255–645
│                                                                                │
├─────────────────────────┬───────────────────────────┬──────────────────────────┤
│ Monthly Distribution    │ Monthly Anomaly Pattern   │ Decadal Warming Trend    │ y: 665–1055
│ (Histogram / Bins)      │ (Seasonal Jan – Dec)      │ (Decade Progression)     │
└─────────────────────────┴───────────────────────────┴──────────────────────────┘
```

### Visual Specifications
- **Canvas Background:** Soft, clean canvas `#F5F8FA` with 0% transparency.
- **Card Containers:** Pure white `#FFFFFF` cards with `10px` rounded corners, subtle `#E2E8F0` borders, and soft ambient drop shadows.
- **Header Bar:** Deep Navy (`#17324D`) command bar with bold white title and teal (`#64CCC5`) subtitle.
- **Color Palette:**
  - Warming Anomaly / Heat Accent: `#E76F51` (Coral) / `#D90429` (Crimson)
  - Baseline & Ocean Data: `#176B87` (Deep Ocean Teal)
  - Secondary Accents: `#F4A261` (Amber), `#64CCC5` (Mint)

---

## 📈 Key Analytical Insights & Findings

### 1. Decadal Warming Shift
Global mean temperatures by decade illustrate a significant departure from historical baseline levels:

| Decade Cohort | Avg Monthly Anomaly | Variance vs Baseline | Climate Regime |
| :--- | :---: | :---: | :--- |
| **1850s** | -0.340 °C | -0.34 °C | Pre-Industrial Baseline |
| **1880s** | -0.338 °C | -0.34 °C | Early Industrial Baseline |
| **1910s** | -0.364 °C | -0.36 °C | Cool Historic Baseline |
| **1940s** | +0.025 °C | +0.03 °C | Neutral Transition Period |
| **1970s** | +0.045 °C | +0.05 °C | Moderate Initial Rise |
| **1980s** | +0.267 °C | +0.27 °C | Accelerated Warming Begins |
| **1990s** | +0.439 °C | +0.44 °C | Strong Decadal Shift |
| **2000s** | +0.631 °C | +0.63 °C | Continued Thermal Escalation |
| **2010s** | +0.847 °C | +0.85 °C | Historic Warm Record |
| **2020s (to 2024)** | **+1.084 °C** | **+1.08 °C** | **Extreme Heat Era (>+1.0°C)** |

### 2. Post-1980 Acceleration
From 1850 to 1979, the global anomaly remained stable within -0.4°C and 0.0°C. Since 1980, the rate of increase accelerated to **+0.18°C per decade**, culminating in an average anomaly of **+1.08°C** for the current decade.

### 3. Record Extremes
- **All-Time Warmest Month:** **September 2023 (+1.548°C)**, followed closely by consecutive months exceeding +1.2°C through 2024.
- **All-Time Coldest Month:** **January 1861 (-1.004°C)**.

### 4. Seasonal Uniformity
Examining monthly averages across the full 175-year timeline reveals that global anomalies are nearly identical across calendar months (varying by merely ±0.03°C from Jan to Dec). This confirms that warming is a continuous, planetary phenomenon rather than a localized seasonal fluctuation.

---

## 📁 Project File Inventory

```
D:\Entri\mini_pjt\
├── mini_project_final.pbip                 <- Main Power BI Developer Project
├── mini_project_final.Report\              <- PBIR report definitions, pages, and theme
├── mini_project_final.SemanticModel\       <- Semantic model, TMDL tables, and DAX measures
├── Climate_Change_Analysis_Report.pdf      <- 3-page publication-ready PDF report
├── README.md                               <- Full project documentation
│
└── Climate_Change_PowerBI_Starter_Kit\
    └── Climate_Change_PowerBI_Kit\
        ├── Climate_Temperature.xlsx        <- Cleaned source workbook
        ├── Climate_Theme.json              <- Custom color theme file
        ├── DAX_Measures.txt                <- Reference DAX measure definitions
        └── PowerBI_Setup_Guide.pdf         <- Original lab guide
```

---

## 🚀 How to Run & Export

### Opening the Project in Power BI Desktop
1. Ensure **Power BI Desktop** (October 2024 release or newer) is installed.
2. Navigate to `D:\Entri\mini_pjt\` and double-click:
   ```
   mini_project_final.pbip
   ```
3. Power BI Desktop will automatically load the semantic model and the redesigned layout.

### Exporting to PDF
- **Pre-generated PDF:** Open `D:\Entri\mini_pjt\Climate_Change_Analysis_Report.pdf` for the complete 3-page executive report.
- **Native Power BI Desktop PDF Export:**
  1. Open `mini_project_final.pbip` in Power BI Desktop.
  2. Click **File > Export > Export to PDF**.
  3. Power BI Desktop will generate a vector PDF matching your active report view.
