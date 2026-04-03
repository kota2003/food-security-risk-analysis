# 🌍 Global Food Security Risk Analysis

> End-to-end data analysis pipeline identifying countries at highest risk of food insecurity and quantifying key economic, agricultural, and climatic drivers — built for policy makers, NGOs, and think-tank analysts.

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2.x-150458?logo=pandas)
![Scikit--learn](https://img.shields.io/badge/Scikit--learn-1.x-f7931e?logo=scikit-learn&logoColor=white)
![R²](https://img.shields.io/badge/R²-0.853-2e7d32)
![Countries](https://img.shields.io/badge/Countries-200-1565c0)
![Years](https://img.shields.io/badge/Years-2010--2022-6a1b9a)

---

## 📋 Table of Contents

1. [Project Overview](#-project-overview)
2. [Data Sources](#-data-sources)
3. [Pipeline & Methodology](#️-pipeline--methodology)
4. [Exploratory Data Analysis](#-exploratory-data-analysis)
5. [Modeling & Results](#-modeling--results)
6. [Key Findings](#-key-findings)
7. [Repository Structure](#-repository-structure)
8. [How to Run](#-how-to-run)
9. [Limitations](#️-limitations)

---

## 📌 Project Overview

Food insecurity affects hundreds of millions of people globally — driven by interacting economic, agricultural, and climatic factors. This project builds a **reproducible end-to-end analytical pipeline** to identify high-risk countries and quantify key drivers using publicly available international datasets.

> **Goal:** The emphasis is on *interpretability over complexity*. A linear model that policy makers can understand and act on outperforms a black-box model that cannot be explained.

### Target Users
- 🏛️ **Policy makers** — country-level risk rankings and driver breakdowns
- 📊 **Think-tank analysts** — interpretable regression coefficients and regional comparisons
- 🌱 **NGOs (food / agriculture)** — risk cluster profiles and COVID-19 impact analysis

### Key Questions Addressed
- Which countries are at highest risk of food insecurity?
- What factors contribute most to that risk?
- How do economic vs. climate vs. agricultural factors compare?
- Can we cluster countries by risk profile?
- What was the measurable impact of COVID-19?

### Dataset at a Glance

| | |
|---|---|
| **Countries** | 200 |
| **Years** | 2010–2022 (13 years) |
| **Observations** | 2,599 rows × 18 columns |
| **Features** | 15 |
| **Model R² (test)** | **0.853** |

---

## 📦 Data Sources

Four datasets were downloaded, cleaned, and merged into a single analysis-ready file (`data/processed/merged_final.csv`).

| File | Source | Shape (raw) | Countries | Key Variables |
|---|---|---|---|---|
| `Suite_of_Food_Security_Indicators.csv` | FAOSTAT | 20,471 × 15 | 204 | undernourishment %, dietary energy, cereal import dep., political stability |
| `FoodBalances.csv` | FAOSTAT | 7,275 × 15 | 179 | protein g, fat g, kcal supply per capita |
| `ProductionIndices.csv` | FAOSTAT | 4,506 × 14 | 200 | agri & food production index (per capita) |
| `WorldDevelopmentIndicators.csv` | World Bank | 1,601 × 17 | 268 | GDP per capita, population, precipitation, poverty rate |
| **`merged_final.csv`** | processed | **2,599 × 18** | **200** | All features combined · 2010–2022 |

> ⚠️ **Missing data note:** `poverty_rate` had 60.1% missing values and was excluded from the main model. `undernourishment_pct` (target variable) had 22.6% missing — rows were dropped for modeling only.

---

## ⚙️ Pipeline & Methodology

```
Phase 1          Phase 2           Phase 3     Phase 4      Phase 5
Data Loading  →  Cleaning & Merge  →  EDA  →  Modeling  →  Insights
```

### Key Cleaning Decisions (Phase 2)

| Issue | Root Cause | Resolution |
|---|---|---|
| BOM character in column names | FAOSTAT UTF-8-BOM encoding | `encoding='utf-8-sig'` |
| `Year` as string object | 3-yr averages use `'2009-2011'` format | Custom `parse_year()` extracts end year |
| Non-numeric `Value` column | `'<2.5'` detection-limit strings | Replaced with `2.5` (conservative upper bound) |
| Duplicate rows for China | M49=159 (aggregate) and M49=156 both map to ISO3=CHN | Excluded M49=159 from all FAOSTAT datasets |
| 4 countries missing ISO3 | Small territories absent from pycountry | Manually mapped COK, FRO, NIU, TKL |
| PI missing years 2015, 2018 | Absent from source data | Confirmed as source-level gap, not a bug |

### Modeling Decisions (Phase 4)

| Decision | Rationale |
|---|---|
| Target: `log(1 + undernourishment_pct)` | Right skew (Skewness = 2.037) — violates linear regression normality assumption |
| Feature: `log(gdp_per_capita)` | EDA showed non-linear (exponential) relationship with target |
| Drop `food_supply_kcal` | r = 0.99 with `dietary_energy_supply_kcal` — severe multicollinearity |
| Drop `food_production_index` | r = 0.99 with `agri_production_index` — severe multicollinearity |
| Drop `poverty_rate` | 60.1% missing — reduces modeling dataset to unusable size |

---

## 🔍 Exploratory Data Analysis

### Target Variable — Undernourishment Rate Distribution

![Target variable distribution](outputs/figures/01_target_distribution.png)

*Figure 1 — Distribution of `undernourishment_pct` (raw and log-transformed). Strong right skew (Skewness = 2.037) necessitates log transformation for linear regression.*

### Feature Correlation Matrix

![Correlation heatmap](outputs/figures/02_correlation_heatmap.png)

*Figure 2 — Pearson correlation matrix across all numeric features. Strongest predictors: `dietary_energy_supply_kcal` (r = −0.84), `poverty_rate` (r = +0.80), `protein_supply_g` (r = −0.77).*

> 💡 **Multicollinearity detected:** `food_supply_kcal` and `dietary_energy_supply_kcal` have r = 0.99. Similarly `food_production_index` and `agri_production_index` have r = 0.99. One from each pair was dropped before modeling.

### Regional Trends (2010–2022)

![Regional undernourishment trends](outputs/figures/03_regional_trends.png)

*Figure 3 — Average undernourishment rate by region over time. Africa showed no improvement across the entire 13-year period. COVID-19 (2020–2022) reversed progress in all regions.*

| Region | Avg Undernourishment | Trend (2010–2022) |
|---|---|---|
| Africa | 19.2% | → Flat — no progress |
| Oceania | 12.6% | ↘ Slight improvement |
| Asia | 9.1% | ↘ Improving (slight COVID reversal) |
| Americas | 9.0% | ↘ Slight improvement |
| Europe | 3.4% | ↘ Consistently improving |

### Highest-Risk Countries (2022)

![Top risk countries 2022](outputs/figures/04_top_risk_countries.png)

*Figure 4 — Top 15 countries by undernourishment rate (2022). **18 of the top 20 are in Africa.***

| Rank | Country | Undernourishment | Region |
|---|---|---|---|
| 1 | Somalia | 52.1% | Africa |
| 2 | Haiti | 47.8% | Americas |
| 3 | Madagascar | 39.9% | Africa |
| 4 | Liberia | 37.7% | Africa |
| 5 | DR Congo | 36.6% | Africa |

### COVID-19 Impact (2019→2022)

![COVID-19 impact analysis](outputs/figures/08_covid_impact.png)

*Figure 5 — Countries with the largest deterioration post-COVID. Global average rose +3.2pp despite GDP recovering +11% — economic recovery did not reach the poorest populations.*

### Country Risk Clusters (K-Means, k=4, 2022)

| Cluster | Countries | Avg Undernourishment | Key Characteristics |
|---|---|---|---|
| 🔴 High Risk | 46 | 21.7% | Low GDP, political instability, high population growth |
| 🟠 Medium Risk | 39 | 9.3% | Middle income, tropical climate, moderate stability |
| 🔵 Low Risk | 58 | 3.8% | Middle-to-high income, stable governance |
| 🟢 Very Low Risk | 25 | 2.6% | Advanced economies, high GDP, political stability |

---

## 🤖 Modeling & Results

A **linear regression model** was built on 1,313 complete-case observations (148 countries, 2010–2022) after log-transforming skewed variables and removing multicollinear features.

### Model Performance

| Metric | Train | Test | Assessment |
|---|---|---|---|
| R² | 0.8541 | **0.8529** | Explains 85% of variance |
| RMSE | 0.2936 | **0.2831** | Low error on log scale |
| Overfitting | — | Train ≈ Test | None detected |

### Actual vs Predicted & Residuals

![Model performance](outputs/figures/07_model_performance.png)

*Figure 6 — Actual vs. Predicted (left) and Residual Plot (right) on the test set. No systematic bias detected.*

### Feature Importance

![Feature importance](outputs/figures/05_feature_importance.png)

*Figure 7 — Standardized regression coefficients. `dietary_energy_supply_kcal` dominates at −0.560 — approximately 4× stronger than the next variable.*

| Feature | Coefficient | Direction |
|---|---|---|
| `dietary_energy_supply_kcal` | **−0.560** | More food → less risk |
| `protein_supply_g` | −0.143 | More protein → less risk |
| `food_supply_variability_kcal` | +0.103 | More variability → more risk |
| `cereal_import_dependency_pct` | +0.060 | More import dependency → more risk |
| `population_growth_pct` | +0.050 | Faster growth → more risk |
| `log_gdp_per_capita` | −0.031 | Wealthier → less risk |
| `political_stability_index` | +0.001 | Near-zero direct effect |

### GDP × Food Supply Structure

![GDP vs food supply risk structure](outputs/figures/06_gdp_food_structure.png)

*Figure 8 — Joint effect of GDP and food supply on undernourishment risk. The diagonal gradient (red bottom-left → green top-right) shows both economic strength and food access must improve together. Improving only one dimension has structurally limited impact.*

---

## 💡 Key Findings

### Finding 1 — Dietary Energy Supply Is the Dominant Driver
**Coefficient: −0.560** (by far the largest in absolute value)

A 1 standard deviation increase in caloric food supply is associated with a 0.56-unit decrease in log-undernourishment — approximately **4× stronger** than the next variable (protein supply at −0.143). Direct food supply interventions — food aid, caloric subsidies, agricultural yield programs — offer the highest short-term leverage.

---

### Finding 2 — GDP and Food Access Are Structurally Inseparable
`log_gdp_per_capita` ranked only 6th (coefficient: −0.031) despite a −0.84 EDA correlation, because it shares ~80% of its information with food supply. Both variables carry heavily overlapping information — the model consolidates shared variance into food supply, which has the cleaner direct relationship.

**Policy implication:** Economic development and food access are two sides of the same coin. Any intervention strategy addressing only one dimension will be structurally limited.

---

### Finding 3 — Africa Requires Sustained, Targeted Attention
Africa's average predicted undernourishment (15.7%) is **2× Asia's (8.5%)** and **5× Europe's (2.9%)**. It is the **only region showing zero improvement** from 2010–2022. 11 of the top 15 highest-risk countries in 2022 are in Sub-Saharan Africa.

---

### Finding 4 — Political Stability Operates Through Indirect Channels
`political_stability_index` ranked last (coefficient: +0.001). This does **not** mean stability is irrelevant — political instability manifests through its downstream effects on GDP and food supply, which are already captured by other features. The impact is real but mediated.

---

## 📁 Repository Structure

```
project_food_security/
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── raw/                  # FAOSTAT & World Bank source files
│   └── processed/
│       └── merged_final.csv  # (2,599 × 18) final analysis-ready dataset
│
├── notebooks/
│   ├── 01_data_loading.ipynb
│   ├── 02_cleaning.ipynb
│   ├── 03_eda.ipynb
│   └── 04_modeling.ipynb
│
├── src/
│   ├── data_loader.py
│   ├── data_cleaning.py
│   ├── feature_engineering.py
│   └── modeling.py
│
├── outputs/
│   └── figures/              # all saved visualizations
│
└── docs/
    ├── methodology.md
    └── findings.md
```

---

## 🚀 How to Run

### 1. Clone & install

```bash
# Clone the repository
git clone https://github.com/your-username/food-security-risk-analysis.git
cd food-security-risk-analysis

# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Download data

Download the following files and place them in `data/raw/`:

| File | Source URL |
|---|---|
| Suite of Food Security Indicators | [FAOSTAT → Food Security](https://www.fao.org/faostat/en/#data/FS) |
| Food Balances | [FAOSTAT → Food Balances](https://www.fao.org/faostat/en/#data/FBS) |
| Production Indices | [FAOSTAT → Production](https://www.fao.org/faostat/en/#data/QI) |
| World Development Indicators | [World Bank WDI](https://databank.worldbank.org/source/world-development-indicators) |

### 3. Run notebooks in order

```bash
jupyter notebook notebooks/01_data_loading.ipynb
# → outputs: data inspection summary

jupyter notebook notebooks/02_cleaning.ipynb
# → outputs: data/processed/merged_final.csv

jupyter notebook notebooks/03_eda.ipynb
# → outputs: outputs/figures/*.png

jupyter notebook notebooks/04_modeling.ipynb
# → outputs: model metrics, feature importance, country rankings
```

### Tech Stack

| Library | Version | Purpose |
|---|---|---|
| `pandas` | 2.x | Data loading, cleaning, merging |
| `numpy` | 1.x | Numerical operations |
| `matplotlib` | 3.x | Visualization |
| `seaborn` | 0.x | Statistical plots |
| `scikit-learn` | 1.x | Linear regression, StandardScaler, train/test split |
| `pycountry` | — | M49 → ISO3 country code mapping |

---

## ⚠️ Limitations

| Limitation | Impact |
|---|---|
| FAOSTAT minimum threshold (2.5%) | Creates artificial floor for low-risk countries; does not affect high-risk analysis |
| `poverty_rate` excluded (60.1% missing) | A key socioeconomic driver is absent; EDA confirmed r = +0.80 with target |
| Linear model assumption | May miss complex non-linear interactions; chosen deliberately for interpretability |
| No country fixed effects | Cross-country structural differences (geography, governance history) are not controlled for |
| Correlation ≠ causation | Coefficients describe statistical association, not causal mechanisms |

> ✅ **Reproducibility:** All notebooks are fully self-contained with inline documentation of every decision, transformation, and finding. Run them in order on any machine with the dependencies installed.

---

*Data: FAOSTAT & World Bank WDI · Python 3.11 · 2024*  
*Figures saved to `outputs/figures/` · Processed data in `data/processed/merged_final.csv`*
