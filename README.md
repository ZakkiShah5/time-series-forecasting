# Store Item Demand Forecasting — Time Series Project

> Comparing classical statistical models (ARIMA, SARIMA, walk-forward SARIMA) with machine learning approaches (Random Forest, XGBoost) on 5 years of daily retail sales data.

**Result:** XGBoost with engineered lag features wins at **2.09% MAPE**, beating walk-forward SARIMA on every metric.

---

## 📊 Final Scoreboard

| Rank | Model | RMSE | MAE | MAPE |
|------|-------|------|-----|------|
| 🥇 | **XGBoost (lag features)** | **993.75** | **611.66** | **2.09%** |
| 🥈 | Random Forest (lag features) | 1061.39 | 622.71 | 2.19% |
| 🥉 | SARIMA Walk-Forward | 1315.68 | 696.06 | 2.48% |
| 4 | Seasonal Naive | 1868.41 | 902.56 | 3.25% |
| 5 | Naive | 8593.58 | 7019.64 | 22.15% |
| 6 | SARIMA single-shot | 8979.64 | 7593.77 | 23.62% |
| 7 | ARIMA single-shot | 10039.96 | 8344.24 | 25.84% |

---

## 🎯 Dataset

- **Source:** Kaggle Store Item Demand Forecasting Challenge
- **Size:** 913,000 rows across 500 store-item series
- **Period:** 5 years of daily data (2013-2017)
- **Granularity:** Daily total sales aggregated across all 500 series

---

## 🔍 Methodology

### Split Strategy
Chronological train/test split:
- **Train:** 2013-01-01 to 2016-12-31 (4 years)
- **Test:** 2017-01-01 to 2017-12-31 (1 year)

Never used `train_test_split` from scikit-learn — shuffling causes future data to leak into training in time series problems.

### Baselines
- **Naive:** Repeat the last training value (22.15% MAPE — the dumb baseline)
- **Seasonal Naive:** today = same day last week (3.25% MAPE — the real benchmark)

### Classical Statistical Models
- **ARIMA(1,1,1):** Failed on long horizon — single-shot prediction collapses to mean (25.84% MAPE)
- **SARIMA(1,1,1)(1,1,1,7):** Marginal improvement (23.62% MAPE) but still suffers long-horizon collapse
- **Walk-Forward SARIMA:** Refit weekly on observed data, forecast 7 days ahead. Dropped MAPE to **2.48%** — the strongest classical result.

### Machine Learning Models

**Feature engineering:**
- Lag features: `lag_1`, `lag_7`, `lag_14`, `lag_30`
- Leak-safe rolling means: `.shift(1).rolling(window=N).mean()` to prevent target leakage
- Calendar features: `day_of_week`, `month`, `day_of_month`

**Models trained:**
- Random Forest Regressor (200 trees, max_depth=10) — 2.19% MAPE
- XGBoost Regressor (200 trees, max_depth=6, learning_rate=0.05) — **2.09% MAPE 🥇**

---

## 💡 Key Findings

### 1. ML beat the best classical method
XGBoost outperformed walk-forward SARIMA across RMSE, MAE, and MAPE — without needing weekly refitting.

### 2. Single-shot ARIMA/SARIMA fail at long horizons
Both single-shot classical models drift toward the mean over 365 days, performing **worse than naive**. This is the long-horizon collapse problem.

### 3. Walk-forward saves SARIMA
Refitting weekly on observed data reduced SARIMA's MAPE from 23.62% to 2.48% — a 10x improvement. Deployment evaluation matters as much as algorithm choice.

### 4. lag_7 dominates the XGBoost forecast
Feature importance analysis revealed `lag_7` alone explained **89%** of XGBoost's predictions, with `lag_1` (5.5%) and `month` (2.9%) providing small corrections. Rolling means contributed less than 1% combined.

**Interpretation:** XGBoost essentially refines seasonal naive's "same day last week" logic by learning small adjustments via calendar features that classical models cannot access.

### 5. Feature engineering wins
The ML advantage over classical models came from feature design — not algorithmic novelty. Calendar features were the structural advantage.

---

## 🛠️ Stack

- **Language:** Python 3.10+
- **Time Series:** `statsmodels` (ARIMA, SARIMA, ADF, KPSS)
- **Machine Learning:** `scikit-learn`, `xgboost`
- **Data:** `pandas`, `numpy`
- **Visualization:** `matplotlib`, `seaborn`

---

## 📁 Repository Structure

store-item-demand-forecasting/
├── README.md
├── notebooks/
│   └── time_series_forecasting.ipynb
├── results/
│   ├── model_comparison.csv
│   ├── forecast_comparison.png
│   └── feature_importance.png
├── data/
│   └── train.csv  (Kaggle source data)


---

## 🚀 Reproduction

```bash
git clone https://github.com/ZakkiShah5/store-item-demand-forecasting.git
cd store-item-demand-forecasting
pip install -r requirements.txt
jupyter notebook notebooks/time_series_forecasting.ipynb
```

---

## 🎓 Context

This project previews methodology from my MSc dissertation at Federal University of Santa Maria (UFSM): *"Hybrid Modelling of Count Time Series via GLARMA and Machine Learning Methods."* The dissertation applies similar hybrid statistical + ML approaches to respiratory-mortality count data, implemented in R.

---

## 👤 Author

**Zakee Ul Hassan**
MSc Mathematics candidate — UFSM, Brazil
[LinkedIn](https://www.linkedin.com/in/zakee-ul-hassan-813b201ab/) · [GitHub](https://github.com/ZakkiShah5)