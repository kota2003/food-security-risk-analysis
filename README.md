# Global Food Security Risk Analysis

Analyzing global food security risk using multi-source public data to identify high-risk regions and key drivers such as economic, climate, and agricultural factors.

---

## Overview

This project identifies countries at high risk of food insecurity and quantifies key drivers (economic, climate, and agricultural factors) to support policy and strategic decision-making.

**Target Users:** Policy makers · Thinktank analysts · NGOs (food / agriculture sector)

---

## Key Questions

- Which countries are at highest risk of food insecurity?
- What factors contribute most to that risk?
- How do economic vs climate factors compare?
- Can we cluster countries by risk profile?

---

## 1. Introduction

Food insecurity is a global issue driven by complex, interacting factors including climate variability, economic conditions, and agricultural productivity. Despite the availability of public data, these relationships are rarely structured or quantified in a way that directly supports policy decisions.

This project builds an end-to-end analytical pipeline — from raw data collection to modeling and interpretation — using publicly available datasets from FAOSTAT and the World Bank. The goal is to produce interpretable, reproducible insights for decision-makers working on food security.

---

## 2. Data

### Sources

| Source | Dataset | Access |
|---|---|---|
| [FAOSTAT](https://www.fao.org/faostat/) | Suite of Food Security Indicators | Public |
| [FAOSTAT](https://www.fao.org/faostat/) | Food Balances | Public |
| [FAOSTAT](https://www.fao.org/faostat/) | Production Indices | Public |
| [World Bank](https://databank.worldbank.org/) | World Development Indicators | Public |

### Variables

**Target Variable**
| Variable | Description | Source |
|---|---|---|
| `undernourishment_pct` | Prevalence of undernourishment (%) | FAOSTAT |

**Features**
| Variable | Category | Source |
|---|---|---|
| `dietary_energy_supply_kcal` | Food Supply | FAOSTAT |
| `food_supply_kcal` | Food Supply | FAOSTAT |
| `protein_supply_g` | Food Supply | FAOSTAT |
| `fat_supply_g` | Food Supply | FAOSTAT |
| `food_supply_variability_kcal` | Food Supply | FAOSTAT |
| `cereal_import_dependency_pct` | Food Supply | FAOSTAT |
| `agri_production_index` | Agriculture | FAOSTAT |
| `food_production_index` | Agriculture | FAOSTAT |
| `gdp_per_capita` | Economic | World Bank |
| `poverty_rate` | Economic | World Bank |
| `population` | Demographic | World Bank |
| `population_growth_pct` | Demographic | World Bank |
| `precipitation_mm` | Climate | World Bank |
| `political_stability_index` | Political | FAOSTAT |

### Final Dataset

| Item | Value |
|---|---|
| File | `data/processed/merged_final.csv` |
| Shape | 2,599 rows × 18 columns |
| Countries | 200 |
| Years | 2010–2022 |

> **Note:** Raw and processed data files are excluded from this repository via `.gitignore` due to file size. Download instructions are provided below.

---

## 3. Methodology

### Pipeline
```
Raw Data (FAOSTAT + World Bank)
        ↓
    Cleaning
  (type fixes, year parsing, value standardization)
        ↓
     Merge
  (M49 → ISO3 country code standardization, left join on iso3 + Year)
        ↓
       EDA
  (distributions, correlations, clustering, COVID impact analysis)
        ↓
    Modeling
  (log transforms, linear regression, feature importance)
        ↓
    Insights                ← coming in Phase 5
```

### Phase Status

| Phase | Description | Status |
|---|---|---|
| Phase 1 | Data Collection & Understanding | ✅ Complete |
| Phase 2 | Data Cleaning & Integration | ✅ Complete |
| Phase 3 | EDA | ✅ Complete |
| Phase 4 | Modeling & Analysis | ✅ Complete |
| Phase 5 | Insights & Reporting | ⏳ Planned |

### Key Cleaning Decisions (Phase 2)

| Issue | Root Cause | Resolution |
|---|---|---|
| BOM character in column names | FAOSTAT UTF-8-BOM encoding | `encoding='utf-8-sig'` |
| `Year` as string object | 3-year average indicators use range format `'2009-2011'` | Extracted end year (`parse_year()`) |
| `'<2.5'` in Value column | Detection limit in undernourishment data | Replaced with `2.5` (upper bound) |
| Duplicate rows for China | M49=159 (aggregate) and M49=156 (mainland) both map to ISO3=CHN | Excluded M49=159 |
| 4 countries missing ISO3 | Small territories absent from pycountry | Manually mapped COK, FRO, NIU, TKL |
| PI missing years 2015, 2018 | Absent from source data | Confirmed as source-level gap |

### Modeling Decisions (Phase 4)

| Decision | Rationale |
|---|---|
| Log-transform target | `undernourishment_pct` has Skewness = 2.037 → violates normality assumption |
| Log-transform GDP | Non-linear (exponential) relationship with target observed in EDA |
| Drop `food_supply_kcal` | r = 0.99 with `dietary_energy_supply_kcal` → multicollinearity |
| Drop `food_production_index` | r = 0.99 with `agri_production_index` → multicollinearity |
| Drop `poverty_rate` | 60.1% missing → would reduce modeling dataset by more than half |
| StandardScaler before fitting | Features have different units (kcal, USD, %) → standardization enables fair coefficient comparison |
| Linear Regression | Project goal is interpretability, not maximum accuracy — coefficients map directly to policy questions |

---

## 4. Results

### EDA Findings (Phase 3)

#### Highest-Risk Countries (2022, Observed)

| Rank | Country | Undernourishment |
|------|---------|-----------------|
| 1 | Somalia | 52.1% |
| 2 | Haiti | 47.8% |
| 3 | Madagascar | 39.9% |
| 4 | Liberia | 37.7% |
| 5 | DR Congo | 36.6% |

#### Strongest Predictors (EDA Correlation)

| Feature | Correlation with Target | Direction |
|---------|------------------------|-----------|
| `dietary_energy_supply_kcal` | -0.84 | More food supply → less risk |
| `food_supply_kcal` | -0.83 | (highly correlated with above) |
| `poverty_rate` | +0.80 | More poverty → more risk |
| `protein_supply_g` | -0.77 | More protein → less risk |
| `political_stability_index` | -0.42 | More stability → less risk |
| `gdp_per_capita` | -0.41 | Higher GDP → less risk (non-linear) |

#### Country Risk Clusters (Phase 3)

| Cluster | Countries | Avg Undernourishment | Profile |
|---------|-----------|---------------------|---------|
| 🔴 High Risk | 46 | 21.7% | Low GDP, political instability, high population growth |
| 🟠 Medium Risk | 39 | 9.3% | Middle-income, tropical regions |
| 🔵 Low Risk | 58 | 3.8% | Middle-to-high income, stable |
| 🟢 Very Low Risk | 25 | 2.6% | Advanced economies, Gulf states |

#### Regional Trends (2010–2022)

| Region | Avg Undernourishment | Trend |
|--------|---------------------|-------|
| Africa | 19.2% | ➡ Flat — no meaningful improvement |
| Oceania | 12.6% | ↘ Slight improvement |
| Asia | 9.1% | ↘ Improving, slight uptick post-COVID |
| Americas | 9.0% | ↘ Slight improvement |
| Europe | 3.4% | ↘ Consistently improving |

#### COVID-19 Impact

- Global undernourishment temporarily improved in 2020, then worsened in 2021–2022
- GDP recovered (+11% by 2022) but undernourishment still rose — recovery benefits did not reach the poorest populations
- Most affected countries: Syria (+10.5pp), Kenya (+8.5pp), Guinea-Bissau (+7.0pp)

---

### Modeling Results (Phase 4)

#### Model Performance

| Metric | Train | Test |
|--------|-------|------|
| RMSE | 0.2936 | 0.2831 |
| R² | 0.8541 | 0.8529 |

The model explains **85% of the variance** in log-undernourishment rates using 10 features. Near-identical Train and Test R² confirms no overfitting.

#### Feature Importance (Standardized Coefficients)

| Rank | Feature | Coefficient | Direction |
|------|---------|-------------|-----------|
| 1 | `dietary_energy_supply_kcal` | -0.560 | ↓ reduces risk |
| 2 | `protein_supply_g` | -0.143 | ↓ reduces risk |
| 3 | `cereal_import_dependency_pct` | +0.055 | ↑ increases risk |
| 4 | `population_growth_pct` | +0.043 | ↑ increases risk |
| 5 | `precipitation_mm` | -0.037 | ↓ reduces risk |
| 6 | `log_gdp_per_capita` | -0.031 | ↓ reduces risk |
| 7–10 | others | < ±0.030 | — |

> **Note on GDP:** `log_gdp_per_capita` shows a small coefficient (-0.031) despite being a strong EDA correlate (r = -0.84). This is because GDP and `dietary_energy_supply_kcal` share r = 0.80 — the model cannot fully separate these two effects. GDP's influence is effectively captured through the food supply variable.

#### High-Risk Country Ranking (2022, Model-Based)

| Rank | Country | Predicted Undernourishment |
|------|---------|--------------------------|
| 1 | Haiti | 35.3% |
| 2 | Madagascar | 32.8% |
| 3 | DR Congo | 31.5% |
| 4 | Zambia | 27.1% |
| 5 | Liberia | 26.2% |

> Somalia ranked #1 in EDA (52.1%) but is absent from the 2022 model ranking due to missing data in that year.

#### Regional Average Predicted Risk (2022)

| Region | Avg Predicted Risk |
|--------|--------------------|
| Africa | 15.7% |
| Asia | 8.5% |
| Americas | 7.7% |
| Oceania | 6.9% |
| Europe | 2.9% |

Africa's average predicted risk is **1.8× higher than Asia** and **5.4× higher than Europe**.

---

## 5. Insights

### Key Finding 1 — Dietary energy supply is the dominant policy lever

Caloric supply is by far the strongest predictor of undernourishment risk (standardized coefficient = -0.560, nearly 4× the next feature). Direct food supply interventions — food aid, agricultural subsidies, supply chain improvements — offer the highest measurable leverage for risk reduction.

### Key Finding 2 — Economic development and food security must be addressed jointly

GDP per capita and dietary energy supply are highly correlated (r = 0.80). Wealthier countries tend to have higher food supply; the two cannot be meaningfully separated. This means interventions targeting food supply alone without addressing economic capacity — or vice versa — face structural limits. Integrated approaches that combine economic development with food system investment are likely more effective.

### Key Finding 3 — Africa requires targeted, region-specific attention

Africa's average predicted undernourishment (15.7%) is nearly twice that of the next highest region (Asia, 8.5%) and shows no improvement trend over 2010–2022. This contrasts with all other regions, which showed measurable improvement. Generic global food security strategies are insufficient; Africa-specific policy investment is needed at scale.

### Key Finding 4 — High-risk countries share a common structural profile

The highest-risk countries (Haiti, Madagascar, DR Congo) share three characteristics: low caloric supply, low GDP per capita, and high cereal import dependency. This combination — low domestic production, high import reliance, low economic capacity — defines the most vulnerable structural configuration and can serve as an early-warning profile for targeting interventions.

---

## 6. Limitations

- **Missing data:** `poverty_rate` has 60.1% missingness and was excluded from modeling; `undernourishment_pct` (target) has 22.6% missingness
- **FAOSTAT 2.5% data floor:** Countries with undernourishment below 2.5% are reported as exactly 2.5% — this creates a data cluster affecting 25% of records, primarily for low-risk countries
- **Somalia excluded from 2022 model ranking:** Missing 2022 data removed the highest-risk EDA country from the model-based ranking
- **Proxy variables:** Undernourishment rate is a proxy for food insecurity, not a direct measure
- **Correlation ≠ causation:** Statistical relationships identified do not imply causal mechanisms
- **Linear assumption:** The model assumes linear relationships after log-transformations; some residual non-linearity may remain
- **Country coverage:** 200 countries after merging; some small territories have limited data
- **Temporal gaps:** Production indices missing for 2015 and 2018 in source data

---

## 7. Repository Structure

```
food-security-risk-analysis/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/                        ← excluded from GitHub (see below)
│   ├── raw/
│   └── processed/
│
├── notebooks/
│   ├── 01_data_loading.ipynb    ✅ Phase 1 complete
│   ├── 02_cleaning.ipynb        ✅ Phase 2 complete
│   ├── 03_eda.ipynb             ✅ Phase 3 complete
│   └── 04_modeling.ipynb        ✅ Phase 4 complete
│
├── src/
│   ├── data_loader.py
│   ├── data_cleaning.py
│   ├── feature_engineering.py
│   └── modeling.py
│
├── outputs/
│   ├── figures/
│   └── tables/
│
└── docs/
    ├── methodology.md
    └── findings.md
```

---

## 8. Getting Started

### Requirements

```bash
pip install -r requirements.txt
```

**Key dependencies:** `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`, `plotly`, `missingno`, `pycountry`

### Download Data

**FAOSTAT:**
1. Go to [https://www.fao.org/faostat/en/#data](https://www.fao.org/faostat/en/#data)
2. Download the following datasets and place them in `data/raw/`:
   - Suite of Food Security Indicators → `Suite_of_Food_Security_Indicators.csv`
   - Food Balances → `FoodBalances.csv`
   - Production Indices → `ProductionIndices.csv`

**World Bank:**
1. Go to [https://databank.worldbank.org/source/world-development-indicators](https://databank.worldbank.org/source/world-development-indicators)
2. Download → `WorldDevelopmentIndicators.csv`

### Run Notebooks

```bash
# Run in order
jupyter notebook notebooks/01_data_loading.ipynb
jupyter notebook notebooks/02_cleaning.ipynb
jupyter notebook notebooks/03_eda.ipynb
jupyter notebook notebooks/04_modeling.ipynb
```

---

## Tech Stack

`Python` · `Pandas` · `NumPy` · `Matplotlib` · `Seaborn` · `Scikit-learn` · `Plotly` · `Missingno` · `pycountry`

