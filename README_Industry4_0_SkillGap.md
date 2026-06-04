# Bridging the Industry 4.0 Skill Gap: A Predictive Analytics Framework

A six-module Python pipeline that quantifies workforce readiness gaps across countries using World Bank data (2010–2024), calibrated against WEF 2025 benchmarks. Produces a custom **Skill Gap Index (SGI)**, country risk clusters, regression models, and employment forecasts through 2030.

---

## Overview

Industry 4.0 technologies — AI/ML, Cloud, Cybersecurity, Blockchain, Digital Twins, IoT — are outpacing workforce skill development globally. This project builds a data-driven pipeline to:

- Measure skill gaps by country using a composite SGI
- Map gaps across 8 technology domains with WEF-calibrated multipliers
- Cluster countries by workforce readiness level
- Model the relationship between education investment and employment outcomes
- Forecast employment rates and skill gaps through 2030 under three policy scenarios

---

## Datasets

All datasets sourced from the **World Bank Open Data** portal:

| File | Indicator | Code |
|------|-----------|------|
| `API_SL.UEM.TOTL.ZS_*.csv` | Unemployment rate (% of labour force) | SL.UEM.TOTL.ZS |
| `API_SE.TER.ENRR_*.csv` | Gross tertiary enrolment ratio (%) | SE.TER.ENRR |
| `API_SE.XPD.TOTL.GD.ZS_*.csv` | Government education expenditure (% of GDP) | SE.XPD.TOTL.GD.ZS |
| `Metadata_Country_*.csv` | Country metadata (region, income group) | — |

Coverage: **2010–2024**, sovereign countries only (aggregates excluded).
Calibration: **WEF Future of Jobs Report 2025** (global mean SGI = 22.4 pp).

---

## Project Structure

```
Industry4_0_SkillGap_Pipeline.ipynb   ← Main analysis notebook
Dataset/                               ← World Bank CSV/XLSX inputs
Fig1_Global_Trends.png                ← Unemployment, education spend, enrolment trends
Fig2_Skill_Gap_Heatmap.png            ← Technology domain gap heatmap (top 30 countries)
Fig3_Correlation_Matrix.png           ← Correlation matrix (2018–2024)
Fig4_KMeans_Clusters.png              ← PCA cluster plot
Fig5_OLS_Coefficients.png             ← OLS regression coefficient chart
Fig6_XGBoost_Results.png              ← XGBoost feature importance plot
```

---

## Pipeline Modules

### Module 1 — Data Loading & Cleaning
- Loads three World Bank CSVs, skipping metadata header rows
- Melts wide-format year columns into long format (one row per country-year)
- Merges unemployment, tertiary enrolment, and education expenditure on `Country Code` and `Year`
- Derives `Employment_Rate = 100 − Unemployment`
- Filters to 2010–2024 and sovereign nations only
- Forward-fills missing values within each country group

### Module 2 — Skill Gap Index (SGI)
- Computes percentile ranks (0–100) for tertiary enrolment, education expenditure, and employment rate
- Constructs a **Skill Readiness Score (SRS)** with weighted percentiles:
  - Tertiary Enrolment: 40%
  - Education Expenditure: 30%
  - Employment Rate: 30%
- Inverts SRS to get raw SGI (`SGI_raw = 100 − SRS`)
- Linearly calibrates to WEF 2025 global mean of **22.4 percentage points**

### Module 3 — Technology Domain Heatmap (Fig 2)
- Applies WEF 2025 domain-specific multipliers to SGI for 8 technology areas:

| Domain | Multiplier |
|--------|-----------|
| Digital Twins | 1.55× |
| Blockchain | 1.50× |
| AI / ML | 1.45× |
| Cybersecurity | 1.40× |
| IoT & Edge | 1.40× |
| Data Analytics | 1.35× |
| RPA | 1.30× |
| Cloud Computing | 1.25× |

- Visualises gaps for the 30 highest-gap countries as a heatmap

### Module 4 — K-Means Clustering (Fig 4)
- Features: tertiary enrolment, education expenditure, employment rate, SGI
- Optimal K determined via elbow method (K=4 selected)
- PCA reduces to 2 components for visualisation
- Countries labelled and coloured by cluster in a scatter plot
- Cluster means printed for interpretation

### Module 5 — Regression Modelling (Figs 5 & 6)
- **OLS Regression** (`statsmodels`): Employment Rate ~ Tertiary Enrolment + Education Expenditure + SGI
  - Full summary with p-values, R², confidence intervals
  - Coefficient bar chart with standard error bars
- **XGBoost Regressor**: same features, 75/25 train-test split
  - Hyperparameters: 200 estimators, depth 4, learning rate 0.05, subsample 0.8
  - Outputs R², RMSE, and feature importance chart

### Module 6 — 2030 Forecasting
- Cluster-level means used as starting points
- Three policy scenarios simulated year-by-year (2025–2030):

| Scenario | Education Expenditure Growth | SGI Trajectory |
|----------|------------------------------|----------------|
| Optimistic | +5% per year | −0.5 pp/year |
| Baseline | +2% per year | Flat |
| Pessimistic | −1% per year | −0.3 pp/year |

- Employment rate projected using OLS-derived education beta coefficient
- Results stored per cluster and scenario for downstream reporting

---

## Requirements

```bash
pip install pandas numpy matplotlib seaborn scikit-learn statsmodels xgboost openpyxl
```

### Key Libraries

| Library | Purpose |
|---------|---------|
| `pandas` / `numpy` | Data wrangling and numerical operations |
| `matplotlib` / `seaborn` | All figures |
| `scikit-learn` | StandardScaler, KMeans, PCA, train_test_split |
| `statsmodels` | OLS regression with full statistical output |
| `xgboost` | Gradient boosted regression |
| `openpyxl` | Reading Excel files if needed |

---

## Setup

1. Download the four World Bank datasets and place them in a `Dataset/` subfolder.
2. Update `BASE_PATH` in the notebook to match your local directory:
   ```python
   BASE_PATH = r'path/to/your/project/folder'
   DATA_PATH = f"{BASE_PATH}\\Dataset"
   ```
3. Install dependencies (see above).
4. Run all cells top to bottom — figures are saved as 300 dpi PNGs.

---

## Key Results

- **SGI** calibrated to WEF 2025 global mean of 22.4 pp across all sovereign nations
- **Digital Twins** and **Blockchain** show the largest projected skill gaps (1.55× and 1.50× SGI)
- **OLS and XGBoost** models quantify how tertiary enrolment and education spending drive employment outcomes
- **Optimistic scenario** projects meaningful SGI reduction and employment gains by 2030; pessimistic scenario shows widening gaps without policy intervention

---

## Research Context

This pipeline was developed to support an academic paper on Industry 4.0 workforce readiness, targeting the gap between emerging technology adoption and available human capital. The SGI framework and scenario forecasts are designed to inform policymakers, HR strategists, and ESG investors on where to prioritise education and upskilling investment.
