# CMPS 262 Capstone Project — Employment Status Prediction & Risk Profiling

**Authors:** Yeol Ban, Charbel Dawlabani
**Course:** CMPS 262
**Date:** Spring 2026

## Overview

This project uses **December 2025 Current Population Survey (CPS)** microdata from the U.S. Census Bureau / Bureau of Labor Statistics to:

1. **Predict** an individual's employment status (employed vs. unemployed) based on demographic and household attributes.
2. **Profile** demographic segments facing elevated unemployment risk.

## Dataset

- **Source:** [CPS Basic Monthly — December 2025](https://www.census.gov/data/datasets/time-series/demo/cps/cps-basic.2025.html)
- **Records:** ~121,000 individual respondents
- **Target variable:** `PEMLR` (Monthly Labor Force Recode)

> The raw CSV (`dec25pub.csv`) is excluded from version control due to size. Download it from the Census Bureau link above.

## Repository Structure

```
cmps262_project/
├── data/                     # Data files (gitignored)
├── notebooks/
│   ├── 01_eda.ipynb          # Exploratory Data Analysis
│   └── 02_modeling.ipynb     # Modeling, calibration, SHAP
├── report/
│   ├── final_report.md       # Final written report
│   └── slides_outline.md     # Presentation outline
├── src/                      # Helper scripts (if needed)
├── .gitignore
├── README.md
└── requirements.txt
```

## Setup

```bash
pip install -r requirements.txt
```

## Milestones

### Milestone 1 — EDA
- Data cleaning & preprocessing (decode CPS codes, filter working-age population, handle missing values)
- Exploratory Data Analysis (class balance, unemployment by demographics, correlations)
- Feature engineering (age bands, education groups)

### Milestone 2 — Modeling
- Logistic Regression (primary) + Random Forest (comparison)
- 5-fold cross-validation
- Isotonic probability calibration
- F1-optimal threshold tuning
- SHAP individual-level explanations

### Milestone 3 — Report
- Final written report: [`report/final_report.md`](report/final_report.md)
- Presentation outline: [`report/slides_outline.md`](report/slides_outline.md)

## Key Results

| Metric | Value |
|---|---:|
| F1 (unemployed class, tuned threshold) | 0.187 |
| ROC-AUC | 0.718 |
| PR-AUC | 0.113 |
| Calibration (mean predicted vs. actual) | 0.038 vs. 0.038 |

Top risk factors: low family income, less-than-HS education, youth (16–24), unknown/non-married status.
