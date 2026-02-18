This repository **does not include the raw dataset** used in this project.

The source data comes from real operational network telemetry.  
Even though it contains **no personal data** (no user/content-level information) and is aggregated to an hourly level, publishing it could still:

- reveal **infrastructure / monitoring naming conventions** (hostnames, internal metric labels),
- expose **traffic patterns** that may be considered commercially sensitive,
- make it easier to infer details about the underlying organization or environment.

For these reasons, the raw data is intentionally excluded from this public repo.

Notes on saved model artifacts
This project includes serialized model artifacts in `artifacts/` for transparency and demonstration.

You can still read the notebooks as a full end-to-end example of:
- leakage-safe feature engineering for time series
- walk-forward cross-validation with a gap
- baseline benchmarking
- interpretable Ridge model selection

To execute the notebooks end-to-end, you will need to provide your own dataset in the expected format
