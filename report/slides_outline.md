# Presentation Slide Outline
## Employment Status Prediction & Unemployment Risk Profiling

**~12 slides, ~15 minutes**

---

### Slide 1 — Title
- Employment Status Prediction & Unemployment Risk Profiling
- Yeol Ban, Charbel Dawlabani | CMPS 262 | Spring 2026

### Slide 2 — The Problem
- Aggregate unemployment rate hides group-level disparities
- Two questions:
  1. Can we predict individual employment status from demographics?
  2. Which groups face elevated risk, and why?
- Stakeholders: workforce dev, policy, research

### Slide 3 — Data
- CPS December 2025 Basic Monthly microdata (~121K records)
- Filtered to labor-force participants: 44,544 records
- **3.8% unemployed** — severe class imbalance
- 10 demographic features, survey-weighted

### Slide 4 — EDA: Who Is Unemployed?
- 4-panel chart: unemployment rate by age / education / race / region
- Key finding: youth + low education = compounding risk
- Age × Education heatmap (the headline visual)

### Slide 5 — Modeling Approach
- Primary: Logistic Regression (interpretable)
- Comparison: Random Forest
- `class_weight='balanced'` to handle imbalance
- 80/20 stratified split + 5-fold CV

### Slide 6 — Performance
- Model comparison table (F1, ROC-AUC, PR-AUC)
- LR and RF essentially tied → signal is near-linear
- ROC and PR curves side-by-side
- Population-weighted metrics confirm representativeness

### Slide 7 — Calibration
- Problem: balanced-weight training inflates probabilities (mean 0.42 vs. true 0.04)
- Solution: isotonic calibration
- Reliability diagram: before vs. after
- Result: mean predicted = 0.038 ≈ actual 0.038

### Slide 8 — Threshold Tuning
- Default 0.5 threshold is wrong at 4% base rate
- F1-optimal threshold: 0.119 → F1 = 0.19 (up from 0.14)
- Three operating points: F1-opt, high-recall, high-precision

### Slide 9 — Risk Factors
- Horizontal bar chart: standardized LR coefficients
- Top ↑: low education, Black race, unknown marital status
- Top ↓: family income, prime-age, graduate degree
- RF feature importance agrees

### Slide 10 — SHAP Explanations
- Global: mean |SHAP| per feature (confirms LR coefficients)
- Individual waterfall: high-risk 18yo example
- Auditable, per-person explanations

### Slide 11 — High-Risk Segments
- Table/chart of top 10 age × education × sex cells
- All top cells: 16–24 + low education
- Bottom cells: prime-age + graduate degree
- Calibration plot (predicted vs. actual by segment)

### Slide 12 — Conclusions & Limitations
**Takeaways**
- Interpretable, calibrated model with F1 = 0.19, AUC = 0.72
- Risk rankings match labor economics
- Good for stratification, not high-precision individual targeting

**Limitations**
- Cross-sectional (one month)
- No industry/occupation features
- Correlation ≠ causation

**Future work:** multi-month pooling, industry features, fairness analysis

---

### Backup slides (if asked)
- Confusion matrix at tuned threshold
- Full classification report
- Missingness analysis
- Survey-weight methodology
