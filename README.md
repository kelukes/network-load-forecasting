# Regression-Based Network Load Forecasting for Sustainable Digital Infrastructure

Forecast **1-hour-ahead** network load from an hourly time series using **leakage-safe feature engineering** and **walk-forward cross-validation**.

---

## Problem
- **Task:** 1-step-ahead forecasting — predict traffic at hour *t* using information available up to *t−1*
- **Target:** hourly **peak** traffic on the upstream link: `y[t] = max(IN[t], OUT[t])` (reported in **Gbps**)
- **Why it matters:** capacity planning and operational resilience — rare peaks are high-risk events

---

## Data 
- **Granularity:** hourly aggregates only (no user/content-level data)
- **Time range:** 2025-08-19 → 2026-01-14 (UTC)
- **Size:** ~3.3k rows after feature engineering, ~20+ features
- **Known gap:** 35-hour downtime window (planned maintenance / structured outage) — treated as a limitation (not random missingness)

> **Data availability:** raw data is **not included** in this repository (confidential operational telemetry).  
> See `data/README.md` for the expected schema and how to plug in a compatible dataset.

---

## Method
### Time-series safe evaluation
- **Walk-forward CV (expanding window)** with **3 folds**
- **24-hour gap** between train and test to reduce boundary leakage
- Metrics reported:
  - **RMSE / MAE** 
  - **Peak RMSE / MAE** on **top 1% hours** within each test fold (extreme load slice)

### Models
- **Baselines**
  - Naive persistence: `ŷ[t] = y[t−1]` (lag_1)
  - Seasonal naive: `ŷ[t] = y[t−24]` (lag_24)
- **Main model**
  - **Ridge Regression** + `StandardScaler` (interpretable + stable coefficients)
- **Nonlinear check**
  - **XGBoost** (tested; did not outperform Ridge under current setup)

---

## Key Results Walk-forward CV
| Model | RMSE (Mbps) | MAE (Mbps) | R² |
|------|------------:|-----------:|---:|
| Naive (lag_1) | ~297 | ~200 | ~0.58 |
| Seasonal naive (lag_24) | ~333 | ~220 | ~0.47 |
| **Ridge (α≈0.001–0.01)** | **~235** | **~153** | **~0.74** |


**Peak hours (top 1%)** remain challenging (systematic underprediction), consistent with rare-event behavior and regime shifts.

---

## Repository Contents
- `notebooks/` — EDA/FE + modeling notebooks (walk-forward CV, baselines, Ridge, XGBoost)
- `artifacts/` — exported figures, tables, and model outputs used in reporting
- `docs/` — submitted deliverables (stakeholder report / project report / presentation)
- `data/` — **no raw data**, only schema + instructions (`data/README.md`)

---


