# Financial Fraud Detection Analytics Platform

An end-to-end fraud detection analytics platform built on 590,540 real-world financial transactions from the IEEE-CIS dataset. The platform covers the full data pipeline — from raw data ingestion and SQL analytics to machine learning fraud classification with explainable AI and a 3-page Power BI executive dashboard.

---

## Project Summary

| Metric | Value |
|--------|-------|
| Dataset size | 590,540 transactions |
| Total transaction value | $79.74M |
| Fraud amount identified | $3.08M |
| Fraud rate | 3.5% (20,663 fraudulent transactions) |
| XGBoost AUC | 0.81 |
| Baseline AUC (Logistic Regression) | 0.59 |
| Fraud recall | 76% |
| Fraud precision | 72% |
| Class imbalance corrected | 27.6:1 → 1:1 (SMOTE) |

---

## Power BI Dashboard

### Page 1 — Fraud Overview

![Fraud Overview Dashboard](dashboard/dashboard_1.png)

**Key insights from this page:**
- **590.54K transactions** analyzed with **$79.74M** total transaction value
- **20.66K fraudulent transactions** — 3.5% overall fraud rate
- **Discover cards** have the highest fraud rate at **7.7%** — more than double Visa (3.5%) and Mastercard (3.4%)
- **Hour 7 UTC peaks at 10.5% fraud rate** — hours 5–9 UTC form a sustained high-risk window nearly 3x the baseline rate

---

### Page 2 — Risk Drill-Down

![Risk Drill-Down Dashboard](dashboard/dashboard_2.png)

**Key insights from this page:**
- **Credit cards** show **6.7% fraud rate** vs debit cards at 2.4% — nearly 3x higher risk
- **Mobile devices** have a higher fraud rate than desktop — a previously undiscovered signal from the device dimension
- **Discover credit** at 8% and **Visa credit** at 7% are the highest risk card network + type combinations
- Risk segmentation table shows fraud rate by every card network and card type combination — actionable for fraud rule targeting

---

### Page 3 — Model Performance

![Model Performance Dashboard](dashboard/dashboard_3.png)

**Key insights from this page:**
- **XGBoost AUC 0.81** vs Logistic Regression baseline 0.59 — 37% improvement
- **76% of all fraud cases caught** (86,222 true positives) with 72% precision
- **27,753 fraud cases missed** (false negatives) — documented precision-recall tradeoff
- **34,092 legitimate transactions flagged** (false positives) — key metric for customer experience teams
- XGBoost bar clearly longer than Logistic Regression — visual proof of model improvement

---

## Key Findings

- **Discover cards** have the highest fraud rate at **7.7%** — more than double Visa (3.5%) and Mastercard (3.4%)
- **Credit cards** show **6.7% fraud rate** vs debit cards at 2.4% — nearly 3x higher risk
- **Hours 5–9 UTC** represent a sustained high-risk window peaking at **10.5% fraud rate at hour 7** — nearly 3x the baseline
- **Highest risk segment**: Discover credit cards — **8% fraud rate**
- **Visa** carries the most dollar exposure at **$1.99M** despite lower fraud rate — purely due to transaction volume
- **Mobile devices** show higher fraud rate than desktop — additional risk signal from device dimension
- Chi-square test confirms card network is a statistically significant fraud predictor (**χ²=368.90, p≈0**)

---

## Architecture

```
Raw Data (CSV)
     ↓
Phase 1: EDA — fraud pattern discovery across 590K transactions
     ↓
Phase 2: ETL Pipeline — cleaning, feature engineering, star schema, SMOTE
     ↓
Phase 3: SQL Analytics — CTEs, window functions, 3 views, $ at risk
     ↓
Phase 4: ML Model — XGBoost classifier + SHAP explainability
     ↓
Phase 5: Power BI Dashboard — 3-page fraud KPI reporting
```

---

## Star Schema

```
                    dim_card (14,885 rows)
                         |
dim_date (168 rows) — fact_transactions (590,540 rows) — dim_email (743 rows)
                         |
                   dim_device (1,943 rows)
```

**fact_transactions** is the core table — TransactionID, TransactionAmt, isFraud, all dimension keys, and engineered features.

---

## Engineered Features

| Feature | Description |
|---------|-------------|
| `hour` | Transaction hour of day (0–23) derived from TransactionDT |
| `day` | Transaction day of week derived from TransactionDT |
| `amt_zscore` | Transaction amount z-score — flags statistical outliers |
| `amt_log` | Log-transformed amount — reduces right skew for model |
| `high_amt_flag` | Binary flag for top 10% transaction amounts |
| `card_velocity` | Transaction count per card — high velocity is a fraud signal |

---

## SQL Analytics

Three production views created on the star schema:

**vw_fraud_summary** — fraud rate and dollar amount at risk by card network and card type

**vw_high_risk_segments** — ranked fraud risk by card network, card type, and transaction hour

**vw_transaction_velocity** — per-card transaction counts and fraud rates for velocity analysis

Window functions used:
- `AVG() OVER (ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING)` — rolling hourly fraud rate
- `RANK() OVER (ORDER BY fraud_rate_pct DESC)` — high risk segment ranking

---

## Model Performance

| Model | AUC | Accuracy | Fraud Recall | Fraud Precision |
|-------|-----|----------|--------------|-----------------|
| Logistic Regression (baseline) | 0.59 | 56% | 56% | 56% |
| XGBoost (final) | 0.81 | 73% | 76% | 72% |

**Confusion Matrix (XGBoost):**

| | Predicted Legitimate | Predicted Fraud |
|--|--|--|
| Actual Legitimate | 79,884 | 34,092 |
| Actual Fraud | 27,753 | 86,222 |

---

## SHAP Feature Importance

Top fraud signal features identified by SHAP:

1. **TransactionAmt** — high amounts are the strongest single fraud predictor
2. **amt_log** — log-transformed amount confirms amount dominance
3. **card_velocity** — low velocity cards (infrequent usage) are higher risk
4. **hour** — late night and early morning hours push toward fraud
5. **day** — day of week carries meaningful fraud signal

---

## Project Structure

```
Financial-Fraud-Detection-Analytics-Platform/
├── data/
│   ├── raw/                            ← Download from Kaggle (not included)
│   ├── fraud_detection.db              ← SQLite star schema database
│   ├── smote_resampled.pkl             ← SMOTE balanced dataset
│   ├── xgb_fraud_model.pkl             ← Trained XGBoost model
│   ├── eda_finding1_class_imbalance.png
│   ├── eda_finding2_transaction_amount.png
│   ├── eda_finding3_card_type.png
│   ├── eda_finding4_hour.png
│   ├── eda_finding5_missing.png
│   ├── eda_finding6_product_type.png
│   ├── model_roc_confusion.png
│   ├── shap_summary.png
│   └── shap_force_plot.png
├── notebooks/
│   ├── 01_eda.ipynb                    ← Phase 1: EDA — 6 findings
│   ├── 02_etl_pipeline.ipynb           ← Phase 2: ETL + star schema + SMOTE
│   ├── 03_sql_analytics.ipynb          ← Phase 3: SQL CTEs + views + chi-square
│   └── 04_model.ipynb                  ← Phase 4: XGBoost + SHAP
├── dashboard/
│   ├── FraudDetection.pbix             ← Power BI dashboard file
│   ├── dashboard_1.png                 ← Page 1: Fraud Overview
│   ├── dashboard_2.png                 ← Page 2: Risk Drill-Down
│   └── dashboard_3.png                 ← Page 3: Model Performance
├── .gitignore
├── LICENSE
└── README.md
```

---

## How to Run

**1. Clone the repo and set up virtual environment**

```bash
git clone https://github.com/prasadpatil0103/Financial-Fraud-Detection-Analytics-.git
cd Financial-Fraud-Detection-Analytics-
python -m venv venv
source venv/bin/activate        # Mac/Linux
venv\Scripts\activate           # Windows
```

**2. Install dependencies**

```bash
pip install -r requirements.txt
```

**3. Download dataset**

Download the IEEE-CIS Fraud Detection dataset from:
https://www.kaggle.com/competitions/ieee-fraud-detection/data

Place `train_transaction.csv` and `train_identity.csv` in `data/raw/`

**4. Run notebooks in order**

```
01_eda.ipynb           → EDA and fraud pattern discovery
02_etl_pipeline.ipynb  → ETL pipeline, star schema, SMOTE
03_sql_analytics.ipynb → SQL analytics and views
04_model.ipynb         → XGBoost model and SHAP explainability
```

**5. Open Power BI dashboard**

Open `dashboard/FraudDetection.pbix` in Power BI Desktop and connect to `data/fraud_detection.db` via ODBC.

---

## Requirements

```
pandas
numpy
matplotlib
seaborn
scikit-learn
imbalanced-learn
xgboost
shap
jupyter
ipykernel
scipy
```

---

## Dataset

**IEEE-CIS Fraud Detection** — Kaggle competition dataset provided by Vesta Corporation.

- 590,540 real-world e-commerce transactions
- 434 features across transaction and identity files
- Binary target: `isFraud` (0 = legitimate, 1 = fraud)
- ~3.5% fraud rate representing severe class imbalance

---

## Role Signal

| Role | What this project demonstrates |
|------|-------------------------------|
| Data Analyst | SQL CTEs, window functions, statistical validation, segmentation analysis, Power BI dashboarding |
| Data Scientist | XGBoost, SMOTE, SHAP explainability, baseline comparison, AUC evaluation |
| Data Engineer | ETL pipeline, star schema design, feature engineering, SQLite data warehouse, ODBC connection |
| Business Analyst | $3.08M fraud exposure quantified, risk segment identification, executive KPI dashboard |

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

**Dataset notice:** The IEEE-CIS Fraud Detection dataset is sourced from Kaggle and is subject to the competition's terms of use. The raw data files are not included in this repository. Download the dataset directly from [Kaggle](https://www.kaggle.com/competitions/ieee-fraud-detection/data) after accepting the competition rules.