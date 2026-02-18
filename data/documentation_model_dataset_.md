# Dataset Documentation - model_dataset_24h.csv / model_dataset_168.csv

## Overview
These datasets are **feature-engineered, leakage-safe** tables for **1-step-ahead forecasting** of hourly network traffic load.

- **Task:** predict `y[t]` (next hour traffic load) using features available up to `t-1`
- **Granularity:** 1 row = 1 hour (UTC)
- **Sorting:** sorted by `datetime_utc` ascending
- **Evaluation protocol used in modeling:** walk-forward CV with an optional **gap=24 hours** between train and test to reduce boundary leakage

### Files / objects in this project
- **24h feature set:** `df_model`  → shape **(3353, 22)**
- **168h feature set:** `df_model_week` → shape **(3353, 25)**

> Note: These are **model datasets**, not raw monitoring exports.  
> Raw per-uplink columns are present in data_merged.csv.

---

## Target
- **`y`** *(float)*  
  Hourly network load target used for modeling.  
  **Units:** consistent within the notebook (we reported metrics in the same scale).

---

## Core time columns
- **`datetime_utc`** *(datetime, UTC)*  
  Timestamp of the hour (start of hour).

---

## Calendar features (time context)
These capture stable periodic structure (daily + weekly patterns).

- **`hour`** *(int)* — hour of day (0–23)
- **`weekday`** *(int)* — day of week (0=Mon … 6=Sun)
- **`is_weekend`** *(bool/int)* — weekend indicator

### Cyclic encoding (prevents boundary artifacts)
- **`sin_hour`**, **`cos_hour`** *(float)* — cyclic encoding of `hour`
- **`sin_wd`**, **`cos_wd`** *(float)* — cyclic encoding of `weekday`

---

## Lag features (past-only)
Lags are computed from previous values of `y` (leakage-safe).

- **`lag_1`** *(float)* — `y[t−1]`
- **`lag_24`** *(float)* — `y[t−24]` (same hour yesterday)

**Additional weekly lag (168h dataset only):**
- **`lag_168`** *(float)* — `y[t−168]` (same hour last week)

---

## Rolling window features (past-only summary statistics)
Rolling features describe local level and volatility.  
Important: they are computed with **past-only logic** (e.g., shifting before rolling).

### 24-hour window (both datasets)
- **`roll_mean_24`** *(float)* — rolling mean over last 24 hours
- **`roll_max_24`** *(float)* — rolling max over last 24 hours
- **`roll_min_24`** *(float)* — rolling min over last 24 hours
- **`roll_std_24`** *(float)* — rolling std over last 24 hours
- **`traffic_range_24h`** *(float)* — `roll_max_24 - roll_min_24` (24h range proxy)

### 168-hour window (168h dataset only)
- **`roll_mean_168`** *(float)* — rolling mean over last 168 hours
- **`roll_std_168`** *(float)* — rolling std over last 168 hours

---

## Host CPU utilization features
Hourly average CPU utilization from three hosts behind the router.
average hourly CPU utilization:
- **`xen200_cpu_util_avg`** *(float)* 
- **`xen201_cpu_util_avg`** *(float)*
- **`xen203_cpu_util_avg`** *(float)*

---

## Monthly business proxy features
Monthly features are joined to each hourly row.  
To prevent leakage, the dataset uses **previous month values**.

- **`new_vms_ee_prev_month`** *(float/int)* — new VMs created in the previous completed month
- **`new_users_ee_prev_month`** *(float/int)* — new users in the previous completed month

---

## Interaction feature
- **`lag1_sin_hour`** *(float)*  
  Interaction term between short-term persistence and daily cycle  
  (captures that `lag_1` effect can vary by hour-of-day).

---

## Schema

### 24h dataset — 22 columns
`datetime_utc`, `y`, `hour`, `weekday`, `is_weekend`, `sin_hour`, `cos_hour`, `sin_wd`, `cos_wd`,
`lag_1`, `lag_24`,
`roll_mean_24`, `roll_max_24`, `roll_std_24`, `roll_min_24`, `traffic_range_24h`,
`xen200_cpu_util_avg`, `xen201_cpu_util_avg`, `xen203_cpu_util_avg`,
`new_vms_ee_prev_month`, `new_users_ee_prev_month`,
`lag1_sin_hour`

### 168h dataset — 25 columns
All 24h columns плюс weekly features:
- `lag_168`
- `roll_mean_168`
- `roll_std_168`


