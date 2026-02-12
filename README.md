# Regression-Based Network Load Forecasting for Sustainable Digital Infrastructure

Forecast **1-hour-ahead** network load using an hourly time series with engineered, leakage-safe features and walk-forward cross-validation.

## Project Goal
Build an interpretable forecasting baseline for operational planning:
- predict hourly peak throughput (`y = max(IN, OUT)`)
- evaluate performance **overall** and on **extreme peak hours (top 1%)**
- keep the workflow reproducible and time-series safe

---

## Data & Target
- **Granularity:** hourly
- **Time range:** 2025-08-19 → 2026-01-14 (UTC)
- **Rows / features:** ~3.3k rows after FE, ~20+ features
- **Target:** `y` = hourly peak IO traffic (Gbps), defined as `max(total_in_max_bps, total_out_max_bps)`
- **Missing hours:** 35-hour downtime window (planned maintenance / structured outage), documented as a limitation (not random missingness)

**Note:** Raw data is not included in this repo (privacy / availability). See `data/README.md`.

---

## Method Overview
### 1) EDA (time-series focused)
- target distribution (right-skew, rare extreme peaks)
- daily + weekly seasonality (hour-of-day and weekday effects)
- IN vs OUT decomposition (IN dominates most hours; OUT dominates many of the top peaks)
- data quality checks (low-quality hours are extremely rare, no overlap with peaks)
- monthly business proxies as step signals (baseline-level context)

### 2) Feature Engineering (leakage-safe)
All time-dependent features use **past-only information** via `.shift(1)`:
- calendar features + cyclic encoding (sin/cos for hour and weekday)
- lags: `t-1`, `t-24`
- rolling windows (24h): mean/max/std/min + range (volatility)
- CPU utilization (per host)
- monthly proxies (prev-month VMs/users)

---

## Evaluation Protocol
**Walk-forward cross-validation (3 folds)** with a **24h gap** between train and test windows to avoid leakage from rolling/lag boundary effects.

Metrics:
- **RMSE** (sensitive to large errors)
- **MAE** (more robust)
- **Peak slice:** RMSE/MAE on **top 1% hours** inside each test fold

Baselines:
- **Naive:** `ŷ[t] = y[t-1]`
- **Seasonal naive:** `ŷ[t] = y[t-24]` (performed poorly → indicates non-stationarity / daily shape alone is insufficient)

---

## Models
- **Ridge Regression (primary):** strong results on engineered features, stable coefficients across folds
- **XGBoost (explored):** did not outperform Ridge under current settings; results suggest the series is largely linear in the engineered feature space or requires more careful tuning/feature design

---

## Key Results (CV)
*(Fill with your final numbers)*

| Model | RMSE (Gbps) | MAE (Gbps) | R² |
|------|-------------:|-----------:|---:|
| Naive (lag-1) | ~0.30 | ~0.20 | ~0.58 |
| Seasonal naive (lag-24) | worse | worse | ~0.47 |
| **Ridge (best α)** | **~0.235** | **(your value)** | **~0.74** |

Peak hours (top 1%) remain challenging (systematic underprediction), consistent with rare-event behavior.

---

## Repository Contents
- `notebooks/01_eda_feature_engineering.ipynb` — EDA + leakage-safe feature engineering
- `notebooks/02_modeling_evaluation.ipynb` — baselines + walk-forward CV + model comparison
- `reports/figures/` — key plots used in the report/presentation

---
