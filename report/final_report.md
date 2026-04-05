# Employment Status Prediction & Unemployment Risk Profiling
## CMPS 262 Capstone — Final Report

**Authors:** Yeol Ban, Charbel Dawlabani
**Course:** CMPS 262
**Date:** Spring 2026
**Dataset:** December 2025 Current Population Survey (CPS) Basic Monthly microdata

---

## 1. Executive Summary

We built a binary classifier that predicts whether a U.S. labor-force participant is employed or unemployed using demographic and household attributes from the December 2025 CPS. The classifier identifies high-risk demographic segments and provides individual-level risk explanations. Our primary model (isotonic-calibrated logistic regression) achieves an **F1 of 0.19** and **ROC-AUC of 0.72** for the minority (unemployed) class at the tuned operating threshold — a substantial lift over the 3.8% base rate. Risk drivers identified match established labor economics: youth, low education, and low family income are the strongest predictors of unemployment.

---

## 2. Problem Statement & Motivation

The unemployment rate is a headline economic indicator, but the aggregate number masks enormous disparities across demographic groups. Policymakers, workforce development agencies, and researchers need to understand **who** is unemployed, not just **how many**. We address two questions:

1. **Prediction:** Given an individual's age, education, sex, race, marital status, region, citizenship, and household income, can we predict their employment status?
2. **Profiling:** Which demographic segments face elevated unemployment risk, and what are the underlying drivers?

This work is relevant to any stakeholder designing targeted labor-market interventions (e.g., re-training programs, unemployment insurance outreach).

---

## 3. Data

**Source:** December 2025 CPS Basic Monthly microdata (U.S. Census Bureau / BLS), ~121,000 individual records.

**Target:** `PEMLR` (Monthly Labor Force Recode), collapsed to a binary indicator:
- **Employed** (`PEMLR` ∈ {1, 2}): at work, or with a job but not at work
- **Unemployed** (`PEMLR` ∈ {3, 4}): on layoff, or actively looking

We exclude individuals not in the labor force (`PEMLR` ∈ {5, 6, 7}), armed forces personnel, and those under 16. The final modeling dataset contains **44,544 records** with a **3.82% unemployment rate** — a severely imbalanced classification problem.

**Features used:**

| Feature | CPS Variable | Type |
|---|---|---|
| Age (continuous + banded) | `PRTAGE` | Numeric + categorical |
| Education (6 groups) | `PEEDUCA` | Categorical |
| Sex | `PESEX` | Binary |
| Race (4 groups) | `PTDTRACE` | Categorical |
| Hispanic origin | `PEHSPNON` | Binary |
| Marital status (3 groups) | `PRMARSTA` | Categorical |
| Census region | `GEREG` | Categorical |
| Citizenship (3 groups) | `PRCITSHP` | Categorical |
| Family income bracket | `HEFAMINC` | Ordinal |
| Survey weight | `PWSSWGT` | Used for evaluation only |

After one-hot encoding, the design matrix has **26 features**.

---

## 4. Exploratory Data Analysis

Key findings from `01_eda.ipynb`:

- **Severe class imbalance.** 96.2% employed, 3.8% unemployed. Accuracy is a useless metric here; we rely on precision, recall, and F1 for the minority class.
- **Age.** Workers aged 16–19 face unemployment rates roughly 3× the prime-age average. Rates decline monotonically until age 65+.
- **Education.** A clean inverse relationship: less-than-high-school workers face unemployment rates ~3× higher than graduate-degree holders.
- **Race/ethnicity.** Black and Hispanic workers show notably higher unemployment than White and Asian workers.
- **Interaction effects.** Age × Education heatmap reveals that young workers *with low education* face dramatically elevated rates — these compound rather than simply add.
- **Missingness.** Minimal after filtering; family income has the most "not in universe" codes (recoded to median).

---

## 5. Methodology

### 5.1 Modeling Approach

We trained two models to cross-check findings:

1. **Logistic Regression** (primary). Chosen for interpretability — standardized coefficients map directly to log-odds effects, which is essential for the risk-profiling objective. Trained with `class_weight='balanced'` to handle imbalance.
2. **Random Forest** (comparison). Captures non-linear interactions the LR may miss. 200 trees, max depth 12, `class_weight='balanced'`.

We used an **80/20 stratified train/test split** (seed 42) and **5-fold stratified cross-validation** to confirm stability.

### 5.2 Calibration

Training with `class_weight='balanced'` produces correctly-*ranked* but inflated predicted probabilities (mean predicted prob ≈ 0.42 vs. actual 0.04). We refit the LR without class weights and applied **isotonic calibration via 5-fold `CalibratedClassifierCV`**. After calibration, the mean predicted probability (0.038) matches the observed unemployment rate (0.038) almost exactly — the reliability diagram shows near-perfect alignment with the diagonal.

### 5.3 Threshold Tuning

The default 0.5 threshold is suboptimal under class imbalance. We swept thresholds on the calibrated model and report three operating points:

| Operating point | Threshold | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| F1-optimal | 0.119 | 0.181 | 0.194 | **0.187** |
| High-recall (≥80%) | 0.021 | 0.058 | 0.806 | 0.108 |
| High-precision (≥20%) | 0.182 | 0.201 | 0.097 | 0.131 |

The F1-optimal threshold improves F1 by 38% over the 0.5 default.

### 5.4 Evaluation Metrics

We report precision, recall, F1, ROC-AUC, and PR-AUC. All metrics are computed both on the raw test sample and re-weighted by `PWSSWGT` (CPS survey weights) to produce population-representative estimates.

---

## 6. Results

### 6.1 Model Performance

| Metric | Logistic Regression | Random Forest |
|---|---:|---:|
| Precision (unemployed) | 0.077 | 0.082 |
| Recall (unemployed) | 0.559 | 0.541 |
| F1 (unemployed) | 0.136 | 0.143 |
| ROC-AUC | 0.718 | 0.713 |
| PR-AUC | 0.113 | 0.114 |

*(Default 0.5 threshold, balanced-weight training)*

**With tuned threshold (calibrated LR):** F1 = 0.187, Precision = 0.181, Recall = 0.194.

**Population-weighted metrics** (using survey weights on test set) closely match unweighted results, confirming the sample is representative.

**5-fold CV confirms stability:** F1 = 0.145 ± 0.005, ROC-AUC = 0.724 ± 0.013.

Logistic regression and random forest perform essentially identically on this tabular, largely-categorical data — a strong indication that the underlying signal is close to linear-additive. We select **logistic regression as the primary model** given interpretability.

### 6.2 Risk Factors (Standardized LR Coefficients)

**Top risk-increasing factors** (positive coefficients → higher unemployment log-odds):

| Feature | Coefficient | Odds Ratio |
|---|---:|---:|
| `marital_Unknown` | +0.199 | 1.22 |
| `region_West` | +0.135 | 1.14 |
| `race_Black` | +0.116 | 1.12 |
| `region_Northeast` | +0.105 | 1.11 |
| `education_LessThanHS` | +0.102 | 1.11 |
| `marital_PrevMarried` | +0.068 | 1.07 |
| `marital_NeverMarried` | +0.066 | 1.07 |

**Top risk-decreasing factors:**

| Feature | Coefficient | Odds Ratio |
|---|---:|---:|
| `family_income` | -0.494 | 0.61 |
| `age_band_35-44` | -0.425 | 0.65 |
| `age_band_25-34` | -0.382 | 0.68 |
| `age_band_45-54` | -0.343 | 0.71 |
| `age_band_55-64` | -0.268 | 0.76 |
| `age_band_65+` | -0.252 | 0.78 |

Family income dominates. Prime-age bands are protective relative to the reference (16–19). Random Forest feature importance confirms the same ordering: family income, age, marital status, and education top the list.

### 6.3 SHAP Explanations

SHAP confirms the global importance ranking and provides individual-level explanations. Two example cases from the test set:

**High-risk individual (predicted p = 0.96):**
- 18-year-old male, less-than-HS education, low family-income bracket (4/16)
- Top contributions: `family_income` (+1.53), `education_LessThanHS` (+0.38), `marital_Unknown` (+0.29)

**Low-risk individual (predicted p = 0.18):**
- 85-year-old male, graduate degree, highest income bracket (16/16)
- Top contributions: `age_band_65+` (-0.81), `age` (-0.46), `family_income` (-0.40)

### 6.4 High-Risk Segments

Aggregating calibrated predictions by age × education × sex identifies the highest-risk demographic cells. The top segments are consistently **young workers (16–24) with less-than-HS or high-school education**, regardless of sex. The lowest-risk segments are **prime-age (35–54) workers with bachelor's or graduate degrees**. The calibration plot shows predicted segment rates align well with observed rates.

---

## 7. Discussion

### 7.1 What the Model Can and Can't Do

**Strengths:**
- Risk **rankings** are well-calibrated and match established labor-economics findings.
- Individual-level explanations (SHAP) make predictions auditable and actionable.
- Survey-weighted metrics confirm population-representative performance.

**Limitations:**
- **Low absolute precision.** Even at the F1-optimal threshold, precision is ~18%. With a 3.8% base rate, the model flags about 4% of the sample but catches only ~20% of unemployed individuals at that precision. The model is better for **risk stratification** than for **individual targeting**.
- **Cross-sectional snapshot.** December 2025 only — no temporal dynamics, no duration of unemployment, no transitions.
- **Feature ceiling.** We rely on demographic attributes only. Industry, occupation, local labor market conditions, and job-search behavior would likely improve predictive power but are beyond scope.
- **Correlation ≠ causation.** Coefficients describe associations, not causal effects. The `race_Black` coefficient, for example, captures residual disparities after controlling for other features — it does not imply race *causes* unemployment.

### 7.2 Policy Implications

The model's strongest use is as a **targeting tool for interventions**:
- Workforce development programs should prioritize young workers with less than a high-school credential.
- Income-support programs (e.g., EITC expansion) address the strongest single predictor (family income).
- Racial/ethnic disparities persist after controlling for age, education, income, and region — indicating structural factors not captured by our features.

---

## 8. Conclusion

We built an interpretable, calibrated unemployment-risk classifier on CPS microdata with competitive performance (F1 = 0.19, ROC-AUC = 0.72) for a severely imbalanced problem. The risk factors we identified (age, education, family income) match established labor economics, and SHAP explanations make individual predictions auditable. The model is most useful for demographic risk profiling and policy targeting rather than high-precision individual classification.

### Future Work

- **Multi-month extension.** Pool several months of CPS data or compare against Dec 2024 to capture temporal trends.
- **Add industry/occupation features.** `PEIO1COW`, `PRDTIND1`, `PRDTOCC1` likely improve predictive power.
- **Multinomial extension.** Distinguish short-term vs. long-term unemployment, or predict labor-force transitions.
- **Fairness analysis.** Compute equal-opportunity and equalized-odds metrics across demographic groups.

---

## 9. Artifacts

- **EDA notebook:** `notebooks/01_eda.ipynb` — 51 cells, all outputs
- **Modeling notebook:** `notebooks/02_modeling.ipynb` — 43 cells, all outputs (data prep, LR, RF, calibration, threshold tuning, SHAP)
- **Data:** `dec25pub.csv` (CPS December 2025 basic monthly, public)
- **Repository:** https://github.com/Dawlabani/cmps262_project

---

## 10. References

- U.S. Census Bureau / BLS. *Current Population Survey, December 2025 Basic Monthly.*
- U.S. Census Bureau. *CPS Monthly Microdata Technical Documentation.*
- Lundberg, S. M., & Lee, S. I. (2017). *A unified approach to interpreting model predictions.* NeurIPS.
- Pedregosa et al. (2011). *Scikit-learn: Machine Learning in Python.* JMLR 12, 2825–2830.
