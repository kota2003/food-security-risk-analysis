# Global Food Security Risk Analysis

Analyzing global food security risk using multi-source public data to identify high-risk regions and key drivers such as economic and climate factors.

---

## Overview

This project identifies countries at high risk of food insecurity and quantifies key drivers (economic, climate, and agricultural factors) to support policy and strategic decision-making.

**Target Users:** Policy makers, Thinktank analysts, NGOs (food / agriculture sector)

---

## Data Sources

| Source | Dataset | Description |
|---|---|---|
| FAOSTAT | Suite of Food Security Indicators | Undernourishment, food insecurity, energy supply |
| FAOSTAT | Food Balances (2010-) | Protein & fat supply per capita |
| FAOSTAT | Production Indices | Agricultural & food production index per capita |
| World Bank | World Development Indicators | GDP, population, precipitation |

**Year Range:** 2010 – 2022  
**Coverage:** ~180 countries (after merging)

---

## Project Structure

```
food-security-risk-analysis/
├── data/
│   ├── raw/          # Original downloaded datasets
│   └── processed/    # Cleaned & merged data
├── notebooks/
│   ├── 01_data_loading.ipynb
│   ├── 02_cleaning.ipynb
│   ├── 03_eda.ipynb
│   └── 04_modeling.ipynb
├── src/              # Python scripts
├── outputs/          # Figures and tables
├── docs/             # Methodology and findings
├── README.md
└── requirements.txt
```

---

## Progress

| Phase | Description | Status |
|---|---|---|
| Phase 1 | Data Collection & Understanding | ✅ Complete |
| Phase 2 | Data Cleaning & Integration | 🔄 In Progress |
| Phase 3 | EDA | ⬜ Not Started |
| Phase 4 | Modeling & Analysis | ⬜ Not Started |
| Phase 5 | Insights & Reporting | ⬜ Not Started |

---

## Tech Stack

- Python, Pandas, NumPy, Matplotlib, Seaborn
