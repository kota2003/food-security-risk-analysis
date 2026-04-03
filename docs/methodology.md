# Methodology — Global Food Security Risk Analysis

**Project:** Global Food Security Risk Analysis  
**Repository:** https://github.com/kota2003/food-security-risk-analysis  
**Data Coverage:** 200 countries, 2010–2022  
**Status:** Phases 1–4 complete

---

## Overview

This document describes the full analytical pipeline from data acquisition through modeling. The goal is reproducibility — a researcher following these steps should arrive at equivalent results.

```
Phase 1: Data Collection & Understanding
Phase 2: Data Cleaning & Integration
Phase 3: Exploratory Data Analysis (EDA)
Phase 4: Modeling & Analysis
Phase 5: Insights & Reporting   ← current
```

---

## Phase 1 — Data Collection

### Data Sources

| Dataset | Source | Coverage | Key Variables |
|---------|--------|----------|---------------|
| FAOSTAT Food Security Indicators | FAO | 200 countries, 2010–2022 | undernourishment_pct, dietary_energy_supply_kcal, food_supply_kcal, protein_supply_g, fat_supply_g, cereal_import_dependency_pct, food_supply_variability_kcal |
| FAOSTAT Production Indices | FAO | 200 countries, 2010–2022 | agri_production_index, food_production_index |
| World Development Indicators (WDI) | World Bank | 200 countries, 2010–2022 | gdp_per_capita, poverty_rate, population, population_growth_pct, precipitation_mm, political_stability_index |

### Target Variable

`undernourishment_pct` — the percentage of a country's population that is undernourished (i.e., with habitual food consumption below minimum dietary energy requirements). Sourced from FAOSTAT.

**FAOSTAT threshold note:** Values below 2.5% are recorded as `<2.5` (a detection limit). These were recoded to `2.5` (upper bound). This affects approximately **25% of all rows** and creates an artificial floor for low-risk countries. It does not affect high-risk country analysis.

---

## Phase 2 — Data Cleaning & Integration

Notebook: `notebooks/02_cleaning.ipynb`  
Output: `data/processed/merged_final.csv`

### Final Dataset

| Property | Value |
|----------|-------|
| Shape | (2,599 rows × 18 columns) |
| Countries | 200 |
| Years | 2010–2022 |
| Duplicate rows | 0 |

### Columns

```python
# Keys
"iso3", "Area", "Year"

# Target variable
"undernourishment_pct"

# Food supply
"dietary_energy_supply_kcal", "food_supply_kcal", "protein_supply_g",
"fat_supply_g", "food_supply_variability_kcal", "cereal_import_dependency_pct"

# Agricultural production
"agri_production_index", "food_production_index"

# Economic
"gdp_per_capita", "poverty_rate"

# Demographic
"population", "population_growth_pct"

# Climate
"precipitation_mm"

# Political
"political_stability_index"
```

### Issues Encountered and Resolved

| Issue | Root Cause | Resolution |
|-------|-----------|------------|
| BOM character in column names | FAOSTAT UTF-8-BOM encoding | `encoding='utf-8-sig'` on file read |
| `Year` column stored as string | 3-year average indicators use range format (e.g. `'2009-2011'`) | `parse_year()` function extracts end year as integer |
| `Value` column non-numeric | `'<2.5'` detection limit stored as string | Replaced with `2.5` (upper bound) |
| Duplicate rows for China | M49=159 (aggregate) and M49=156 (mainland) both map to ISO3=CHN | Excluded M49=159 from all FAOSTAT datasets |
| 4 countries missing ISO3 | Small territories absent from `pycountry` library | Manually mapped: COK, FRO, NIU, TKL |
| Production Index missing years 2015, 2018 | Absent from source data | Confirmed as source-level gap; no imputation applied |

---

## Phase 3 — Exploratory Data Analysis

Notebook: `notebooks/03_eda.ipynb`

### Missing Value Summary (before modeling)

| Feature | Missing % |
|---------|-----------|
| poverty_rate | 60.1% |
| undernourishment_pct | 22.6% |
| cereal_import_dependency_pct | 22.4% |
| food_production_index | 22.0% |
| agri_production_index | 22.0% |
| food_supply_variability_kcal | 16.4% |
| dietary_energy_supply_kcal | 16.2% |
| food_supply_kcal | 14.0% |
| protein_supply_g | 14.0% |
| fat_supply_g | 14.0% |
| precipitation_mm | 9.2% |
| political_stability_index | 2.5% |
| gdp_per_capita | 1.9% |
| population | 0.5% |
| population_growth_pct | 0.5% |

### Target Variable Distribution

`undernourishment_pct` is right-skewed (Skewness = 2.037), with most countries clustered at low values and a long tail toward high-risk countries. Log transformation (`log(1 + x)`) was applied for modeling.

### Correlation with Target Variable

| Feature | Pearson r |
|---------|-----------|
| dietary_energy_supply_kcal | −0.84 |
| food_supply_kcal | −0.83 |
| poverty_rate | +0.80 |
| protein_supply_g | −0.78 |
| fat_supply_g | −0.67 |
| gdp_per_capita | −0.62 |
| agri_production_index | ~0.00 |
| food_production_index | ~0.00 |

### Notable Multicollinearity

| Pair | Correlation |
|------|-------------|
| food_supply_kcal ↔ dietary_energy_supply_kcal | r = 0.99 |
| food_production_index ↔ agri_production_index | r = 0.99 |
| gdp_per_capita ↔ dietary_energy_supply_kcal | r = 0.80 |

### Clustering (k=4)

Countries were grouped into 4 clusters using k-means on standardized features:

| Cluster | Risk Level | Characteristics |
|---------|-----------|----------------|
| 0 | High Risk | Low caloric supply, low GDP, high poverty |
| 1 | Medium Risk | Moderate food access, developing economies |
| 2 | Low Risk | Adequate supply, middle-income |
| 3 | Very Low Risk | Strong food systems, high income |

---

## Phase 4 — Modeling

Notebook: `notebooks/04_modeling.ipynb`

### Model Selection

**Linear Regression** was chosen over more complex models (e.g., Random Forest, XGBoost) because the primary goal is **interpretability** — producing actionable policy insights requires understanding the *direction* and *magnitude* of each driver's effect.

### Modeling Dataset

| Property | Value |
|----------|-------|
| Rows (after dropping missing) | 1,313 |
| Countries | 148 |
| Years | 2010–2022 |
| Train/test split | 80/20 (random state = 42) |

### Step 1 — Feature Engineering

Two types of transformations were applied:

**Log transformations** (to linearize non-linear relationships and reduce skew):
```python
df['log_gdp_per_capita'] = np.log(df['gdp_per_capita'])
df['log_target'] = np.log(1 + df['undernourishment_pct'])
```

**Dropped due to multicollinearity:**
- `food_supply_kcal` (r = 0.99 with `dietary_energy_supply_kcal`)
- `food_production_index` (r = 0.99 with `agri_production_index`)
- `poverty_rate` (60.1% missing — would reduce dataset to unusable size)
- `fat_supply_g`, `food_supply_variability_kcal` (lower predictive value, overlap with retained features)

### Step 2 — Model Training

```python
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

features = [
    'dietary_energy_supply_kcal',
    'protein_supply_g',
    'cereal_import_dependency_pct',
    'population_growth_pct',
    'precipitation_mm',
    'log_gdp_per_capita',
    'agri_production_index',
    'political_stability_index'
]

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LinearRegression())
])

pipeline.fit(X_train, y_train)
```

**Standardization** (via `StandardScaler`) was applied so that coefficients are directly comparable across features with different units.

### Step 3 — Model Evaluation

| Metric | Train | Test |
|--------|-------|------|
| R² | 0.85 | 0.85 |
| RMSE | — | 0.28 |

Train ≈ Test → no overfitting. The model explains 85% of variance in log-transformed undernourishment rates.

**RMSE is on the log-transformed scale.** Interpreting in original units: predictions are typically within ~2–4 percentage points of actual values across the range of observed undernourishment rates.

### Step 4 — Feature Importance (Standardized Coefficients)

| Rank | Feature | Coefficient | Direction |
|------|---------|-------------|-----------|
| 1 | dietary_energy_supply_kcal | −0.560 | ↓ reduces risk |
| 2 | protein_supply_g | −0.143 | ↓ reduces risk |
| 3 | cereal_import_dependency_pct | +0.055 | ↑ increases risk |
| 4 | population_growth_pct | +0.043 | ↑ increases risk |
| 5 | precipitation_mm | −0.037 | ↓ reduces risk |
| 6 | log_gdp_per_capita | −0.031 | ↓ reduces risk |
| 7 | agri_production_index | −0.010 | ↓ reduces risk |
| 8 | political_stability_index | +0.001 | ≈ negligible |

**Key interpretation note on GDP:** The GDP coefficient (−0.031) appears small relative to its known importance. This is a multicollinearity effect: GDP and dietary energy supply are correlated at r = 0.80. The model distributes explanatory power between them, understating each individually. Both are important in practice.

### Predictions vs. Actual Values

Predictions were back-transformed from log scale:
```python
predictions_pct = np.exp(y_pred) - 1
```

---

## Reproducibility

### Environment

- Python 3.11.5
- pandas, numpy, matplotlib, seaborn, scikit-learn, pycountry

### Execution Order

```
01_data_loading.ipynb   → inspects raw FAOSTAT and WDI files
02_cleaning.ipynb       → produces data/processed/merged_final.csv
03_eda.ipynb            → produces outputs/figures/step*.png
04_modeling.ipynb       → produces outputs/figures/step4_*.png
```

### Randomness

- Train/test split: `random_state=42`
- K-means clustering: `random_state=42`

All other steps are deterministic.

---

## Limitations

| Limitation | Technical Detail |
|------------|----------------|
| FAOSTAT 2.5% floor | 25% of observations are right-censored at the lower bound |
| `poverty_rate` excluded | 60.1% missing; listwise deletion would reduce n to ~500 |
| Linear model | Assumes additive, linear relationships between features and log(target) |
| No country fixed effects | Unobserved country-level heterogeneity (e.g., geography, governance history) not controlled |
| Cross-sectional pooling | Treats country-years as independent; ignores autocorrelation within countries over time |
| Correlation ≠ causation | All coefficients describe statistical association, not causal effect |

---

*For non-technical summary of findings, see `docs/findings.md`.*
