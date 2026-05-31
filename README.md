# Predicting a Country's Banking Z-Score
### Using Financial Institution Access Indicators to Model Banking System Stability

---

## Overview

The World Bank publishes a **Bank Z-Score** (series code `GFDD.SI.01`) for over 200 countries — a measure of commercial banking system stability that compares the buffer of equity and asset returns against the volatility of those returns. A higher Z-score indicates a more stable banking system with a lower probability of default.

This project asks whether a country's Z-score can be predicted from how much access its citizens have to financial products — rather than relying solely on the accounting data the Z-score is traditionally derived from. Using the World Bank's **Financial Institutions: Access** indicators (`GFDD.AI.*`), a Random Forest model was trained on a country-year panel dataset and evaluated on countries it had never seen during training.

The broader motivation is the "banking desert" problem: areas where individuals lack access to basic financial services and are instead preyed upon by high-fee alternatives like check-cashing centers. If financial access indicators predict banking stability, that is evidence that expanding services to underserved communities strengthens the broader financial system — not just the individuals it serves.

---

## Analysis Pipeline

| Stage | Description | Method |
|---|---|---|
| Data Collection | World Bank Z-score (GFDD.SI.01) + Financial Institutions: Access indicators (GFDD.AI.*), 200+ countries, 1997–2021 | World Development Indicators DataBank |
| EDA | Scatterplots of each access indicator vs. Z-score to assess visual relationships and select feature set | matplotlib |
| Data Preparation | Replace ".." with NaN; linear interpolation between sporadic survey years; country-scoped backfill; reshape to one row per country-year; create missingness dummy features | pandas |
| Imputation | Experimented with KNN Imputation vs. mean fill; corrected backfill + mean fill produced best results | scikit-learn |
| Modeling | Random Forest regression; initial random split, then corrected to country-alphabetical split to eliminate data leakage | scikit-learn |
| Feature Selection | Recursive Feature Elimination (RFE) to identify the 10 strongest access predictors of Z-score | scikit-learn |

---

## Key Findings

### Data Leakage — and How It Was Corrected

An initial model scored R² = **0.926** on a random train/test split — suspiciously high. Because the dataset has one row per country per year, adjacent years for the same country are nearly identical. With random splitting, the model could trivially predict "France 2007" by memorizing "France 2006."

The fix: a **country-based alphabetical split**. Countries with codes from "SGP" onward were reserved for testing only, ensuring the model evaluated on countries it had never encountered during training.

### Model Performance

| Model | Test R² | Notes |
|---|---|---|
| Random Forest, random split | 0.926 | Inflated — data leakage from country-year similarity |
| Random Forest, country split, all features | 0.016 | Barely above mean — too many weak features |
| Random Forest, country split, top 10 features (RFE) | ~0.15 | Best honest estimate |

### Top 10 Features by Importance

| Rank | Series Code | Indicator | Approx. Importance |
|---|---|---|---|
| 1 | GFDD.AI.02 | Bank branches per 100,000 adults | ~18% |
| 2 | GFDD.AI.16 | Loan from an employer in the past year (% age 15+) | ~10% |
| 3 | GFDD.AI.08 | Account used for business purposes (% age 15+) | ~10% |
| 4 | GFDD.AI.26 | Depositing/withdrawing at least once in a typical month (% age 15+) | ~10% |
| 5 | GFDD.AI.15 | Loan from a private lender in the past year (% age 15+) | ~9% |
| 6 | GFDD.AI.14 | Loan in the past year (% age 15+) | ~9% |
| 7 | GFDD.AI.20 | Credit card (% age 15+) | ~9% |
| 8 | GFDD.AI.27 | Firms with a checking or savings account (%) | ~8% |
| 9 | GFDD.AI.28 | Firms using banks to finance investments (%) | ~8% |
| 10 | GFDD.AI.12 | Saved any money in the past year (% age 15+) | ~8% |

Physical branch density (AI.02) is the single strongest predictor of banking stability. Loan access features (AI.14, AI.15, AI.16) collectively contribute ~25%, showing that lending access matters even when it originates outside traditional brick-and-mortar institutions.

### Interpretation

Financial institution access indicators explain roughly **15% of Z-score variability** across unseen countries. The Z-score is fundamentally an accounting metric, and access data alone cannot fully capture it. However, these 10 features provide a validated starting point for a larger model incorporating additional World Bank indicator categories — assets, interest rates, and economic policy — to build a more comprehensive picture of what drives banking system stability.

---

## Project Structure

```
bank-z-score-prediction/
│
├── bank_z_score_prediction.ipynb                            # Full analysis notebook: EDA → data prep → modeling → RFE (Milestones 1–4)
├── bank_z_score_prediction.pdf                              # Written analysis and findings
│
├── Sample_bank_Country_data.csv                             # Analysis-ready dataset: country Z-scores + access indicators (merged)
├── Country Survey Data.csv                                  # World Bank Financial Institutions: Access indicators, by country/year
├── Country Survey Data.xlsx                                 # Same — Excel format
├── DataBank Data 2-1-25.csv                                 # Full World Development Indicators export (Feb 2025)
├── 8303f5a8-..._Data.csv                                    # Bank Z-score source data (World Bank export, GFDD.SI.01)
├── 8303f5a8-..._Series - Metadata.csv                       # Series metadata for Z-score data
├── 700160f5-..._Series - Metadata.csv                       # Series metadata for access indicators
├── DataDetails countries.xls                                # Country-level reference data
├── Indicator and Topic Names.xls                            # Full indicator codebook: topic and series names
├── Question key.xlsx                                        # Field reference: maps series codes to full descriptions
└── P_Data_Extract_From_Global_Financial_Development.zip     # Raw World Bank Global Financial Development extract
```

---

## Requirements

```
pandas
numpy
matplotlib
seaborn
scikit-learn
```

Install with:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn
```

---

## Technologies

| Tool | Use |
|---|---|
| Python (pandas) | Data wrangling: NaN handling, linear interpolation, backfilling, country-year pivot |
| Python (scikit-learn) | Random Forest regression, KNN Imputation, Recursive Feature Elimination |
| Python (matplotlib) | EDA scatterplots, feature importance chart |
| World Bank DataBank | Source data: Bank Z-score (GFDD.SI.01) and Financial Institutions: Access indicators (GFDD.AI.*) |

---

## Data Source

All data retrieved from the **World Bank Development Indicators DataBank**:
[https://databank.worldbank.org/source/world-development-indicators#](https://databank.worldbank.org/source/world-development-indicators#)

Series codes used:
- **GFDD.SI.01** — Bank Z-score (target variable)
- **GFDD.AI.*** — Financial Institutions: Access indicators (features)

---

## Academic Context

This project was developed as a **DSC 550** (Data Mining, master's level) final project, completed across four milestones. It demonstrates an end-to-end machine learning workflow applied to a real-world economic dataset: feature selection from a large indicator library, iterative model refinement, recognition and correction of a data leakage problem, and honest interpretation of a model with modest but meaningful predictive power.

---

*Data source: [World Bank Development Indicators DataBank](https://databank.worldbank.org/source/world-development-indicators#) — publicly available, World Bank Group.*
