# Proton Pump Inhibitors vs H2 Receptor Antagonists and All-Cause Mortality: A Survival Analysis of Simulated Electronic Health Records

[![R](https://img.shields.io/badge/R-%3E%3D4.0-blue?logo=r)](https://www.r-project.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## Overview

This project investigates the effect of **proton pump inhibitor (PPI)** prescription, compared to **H2 receptor antagonist (H2RA)** prescription, on subsequent **all-cause mortality** using simulated electronic health record (EHR) data. The analysis employs Cox proportional hazards regression with careful model building, diagnostics, and stratification.

This work was completed as part of the **Analysis of Electronic Health Records** module during an MSc in Health Data Science at the **London School of Hygiene & Tropical Medicine (LSHTM)**.

> **Note:** All data used in this project are **simulated** and do not represent real patients. The simulated dataset mimics the structure of UK primary care electronic health records.

## Study Design

| Component | Description |
|---|---|
| **Study type** | Retrospective cohort study |
| **Data source** | Simulated UK primary care EHR database |
| **Study period** | 17 April 1997 – 17 April 2017 |
| **Population** | Patients with a first prescription for a PPI or H2RA, registered ≥1 year at their GP practice |
| **Exposure** | First PPI prescription (exposed) vs first H2RA prescription (comparator) |
| **Outcome** | All-cause mortality |
| **Follow-up** | From first prescription to death, transfer out, or end of study (17 April 2017) |

### Confounders Adjusted For

- Age (modelled with natural cubic splines, 3 df)
- Sex
- Socioeconomic status (IMD quintile)
- Ethnicity
- Calendar period of first prescription
- BMI
- Number of healthcare contacts in the prior year
- Prior gastric cancer diagnosis
- Recent GERD (within 6 months)
- Recent peptic ulcer (within 6 months)

## Repository Structure

```
├── 3_run_pipeline.R                  # Master script — runs the full pipeline
├── 1_data_extraction_code.Rmd        # Script 1: Data extraction & cohort creation
├── 2_data_analysis_code.Rmd          # Script 2: Statistical analysis & visualisation
├── Codelists/                        # Medical code lists (CSV) for identifying
│   ├── ppi_codes.csv                 #   conditions and prescriptions
│   ├── h2ra_codes.csv
│   ├── gastric_cancer_codes.csv
│   ├── gerd_codes.csv
│   ├── peptic_ulcer_codes.csv
│   └── ... (33 codelist files)
├── 2. Figures/                       # Output figures and tables
│   ├── plot_kaplan_meier.png         #   Kaplan-Meier survival curves
│   ├── plot_cox_predicted_survivor_curves.png # Estimated survivor curves
│   ├── table_baseline_characteristics.png # Table 1 cohort characteristics
│   ├── table_cox_regression_combined.png # Cox regression results table
│   ├── plot_martingale_residuals_age.png # Martingale residual plots
│   ├── plot_schoenfeld_residuals_ppi.png # Schoenfeld residual plots
│   └── ...
├── .gitignore
└── README.md
```

> **Data not included:** The simulated patient-level CSV files (`Assessment_sample/`) and the derived analysis dataset (`analysis_data.Rds`) are excluded from this repository due to file size (~1.2 GB total). See [Data](#data) below.

## Analysis Pipeline

The analysis is executed via a single master script and consists of two sequential stages:

### Script 1: Data Extraction (`1_data_extraction_code.Rmd`)

1. **Data import** — Loads 7 patient-level CSV files and 30+ medical codelists
2. **Eligibility criteria** — Identifies first PPI/H2RA prescription per patient; applies date and registration eligibility windows
3. **Confounder extraction** — Derives comorbidities from clinical records (gastric cancer, GERD, peptic ulcer); calculates BMI from recorded and weight/height-derived sources; extracts demographics (age, sex, SES, ethnicity, calendar period); counts healthcare contacts in the prior year
4. **Exposure investigation** — Exploratory analysis of prescription patterns, treatment persistence, and treatment switching between PPI and H2RA groups
5. **Follow-up & outcome** — Defines follow-up end date and death indicator
6. **Dataset export** — Merges all variables and saves `analysis_data.Rds`

### Script 2: Statistical Analysis (`2_data_analysis_code.Rmd`)

1. **Descriptive statistics** — Table 1 of cohort characteristics by exposure group
2. **Time-to-event summary** — Deaths, censoring, median follow-up, and mortality rates
3. **Kaplan-Meier analysis** — Survival curves by exposure group with log-rank test
4. **Univariable Cox regression** — Crude hazard ratio for PPI vs H2RA
5. **Multivariable Cox regression** — Model building with:
   - Martingale residual plots to assess functional form of continuous covariates
   - AIC comparison of linear, quadratic, and spline models for age
6. **Proportional hazards assessment** — Log-minus-log plots and Schoenfeld residual tests; identification and stratification of variables violating the PH assumption (gender, SES, calendar period)
7. **Extended Cox model** — Time-varying effect of PPI to address residual non-proportionality
8. **Estimated survivor curves** — Predicted survival for specific patient profiles (varying sex, deprivation, and peptic ulcer status)

## Sample Outputs

<p align="center">
  <img src="2. Figures/plot_kaplan_meier.png" width="600" alt="Kaplan-Meier survival curves by PPI vs H2RA exposure">
  <br>
  <em>Kaplan-Meier survival curves by exposure status with number at risk table</em>
</p>

<p align="center">
  <img src="2. Figures/plot_cox_predicted_survivor_curves.png" width="600" alt="Estimated survivor curves for individual profiles">
  <br>
  <em>Estimated survivor curves from the stratified Cox model for four patient profiles</em>
</p>

## Data

The simulated dataset consists of 7 CSV files mimicking UK primary care EHR data:

| File | Description |
|---|---|
| `Sim_Patient` | Patient demographics (ID, practice, sex, ethnicity, year of birth, registration/leave dates, death date) |
| `Sim_Clinic` | Clinical events with medical codes and entity types |
| `Sim_Contactdates` | Healthcare contact dates and types |
| `Sim_Prescription` | Prescriptions with drug codes, strength, quantity, and daily dose |
| `Sim_Other` | Additional clinical measurements (BMI, weight, height, smoking, alcohol) |
| `Sim_Death` | Death records with cause-of-death flags |
| `Sim_SES` | Socioeconomic deprivation status (IMD quintile) |

**These files are not included in the repository** due to their size (~1.2 GB). The codelists required to replicate the analysis are included in `Codelists/`.

## Requirements

### R Packages

```r
pacman::p_load(
  tidyverse, dplyr, readr,       # Data wrangling
  gt, gtsummary,                  # Summary tables
  ggplot2, survival, survminer,   # Survival analysis & visualisation
  broom, splines                  # Model tidying & spline basis functions
)
```

> `mexhaz` is also loaded in Script 2 but is not actively used in the final analysis.

### System

- **R** ≥ 4.0
- The `pacman` package is used for dependency management — install it first with `install.packages("pacman")` if not already available.

## How to Run

1. Clone this repository
2. Place the simulated data CSV files in an `Assessment_sample/` directory at the project root
3. Open `3_run_pipeline.R` and update the `setwd()` path to your local project directory (or use `here::here()`)
4. Run `3_run_pipeline.R` — this will:
   - Execute `1_data_extraction_code.Rmd` (produces `analysis_data.Rds`)
   - Execute `2_data_analysis_code.Rmd` (produces all figures and tables in `2. Figures/`)

## Key Findings

> These results are based on **simulated data** and are for methodological demonstration only.

- The PPI group showed lower unadjusted survival compared to H2RA (Kaplan-Meier analysis)
- Multivariable Cox regression adjusted for demographic and clinical confounders
- The proportional hazards assumption was violated for gender, SES, and calendar period — addressed via stratification
- The final model incorporated a time-varying effect for PPI exposure
- Estimated survivor curves were produced for specific patient profiles varying by sex, deprivation, and peptic ulcer status

## Acknowledgements

This project was completed as part of the MSc in Health Data Science at the [London School of Hygiene & Tropical Medicine](https://www.lshtm.ac.uk/). The simulated dataset and assessment framework were provided by the LSHTM teaching team for the **Analysis of Electronic Health Records** module.

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
