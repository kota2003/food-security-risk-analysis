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
 Feature Engineering        ← coming in Phase 3
        ↓
       EDA                  ← coming in Phase 3
        ↓
    Modeling                ← coming in Phase 4
        ↓
    Insights                ← coming in Phase 5
```

### Phase Status

| Phase | Description | Status |
|---|---|---|
| Phase 1 | Data Collection & Understanding | ✅ Complete |
| Phase 2 | Data Cleaning & Integration | ✅ Complete |
| Phase 3 | EDA | 🔄 In Progress |
| Phase 4 | Modeling & Analysis | ⏳ Planned |
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

---

## 4. Results

> Results will be added upon completion of Phase 4 (Modeling & Analysis).

---

## 5. Insights

> Policy implications will be added upon completion of Phase 5 (Insights & Reporting).

---

## 6. Limitations

- **Missing data:** `poverty_rate` has 60.1% missingness; `undernourishment_pct` (target) has 22.6%
- **Proxy variables:** Undernourishment rate is a proxy for food insecurity, not a direct measure
- **Correlation ≠ causation:** Statistical relationships identified do not imply causal mechanisms
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
│   ├── 03_eda.ipynb             🔄 In progress
│   └── 04_modeling.ipynb        ⏳ Planned
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

**Key dependencies:** `pandas`, `numpy`, `matplotlib`, `seaborn`, `pycountry`

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
jupyter notebook notebooks/03_eda.ipynb        # coming soon
jupyter notebook notebooks/04_modeling.ipynb   # coming soon
```

---

## Tech Stack

`Python` · `Pandas` · `NumPy` · `Matplotlib` · `Seaborn` · `pycountry`
