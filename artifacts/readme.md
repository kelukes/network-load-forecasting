# Saved Model Artifacts (Ridge, 168h feature set)

This folder contains the final trained forecasting model.

## Files
- `ridge_week_final.joblib`  
  Trained scikit-learn Pipeline: `StandardScaler -> Ridge(alpha=...)`.

- `feature_cols_week.joblib`  
  List of feature column names (in the exact order expected by the model).

- `ridge_week_meta.joblib`  
  Metadata dictionary: dataset name, alpha, number of rows/features, time range.

## Training data
Model was refit on the full available modelling dataset:
- Dataset: `df_model_week` (168h feature set)
- Target column: `y`
- Time index: `datetime_utc`
- Training range: see `ridge_week_meta.joblib`

## How to load and predict
```python
import joblib
import pandas as pd

model = joblib.load("artifacts/ridge_week_final.joblib")
feature_cols = joblib.load("artifacts/feature_cols_week.joblib")

# df must contain all columns in feature_cols
X = df[feature_cols]
y_pred = model.predict(X)
