# Input Representation & Normalization

## Purpose & Content

This document specifies how raw time series and covariates are represented and normalized before entering the model.  It enumerates supported data regimes, explains normalization strategies and describes how normalization parameters are stored to allow invertibility.

## Audience

Data engineers and modellers responsible for preprocessing and ingestion.

## Key Sections

- Supported data regimes and examples
- Normalization strategies (z‑score, robust scaling, Box‑Cox, etc.)
- Handling multivariate and hierarchical data
- Metadata and covariate alignment

## Dependencies

Doc06 (Core Model Architecture)

## Document Body

### Data Regimes

AionFM supports:

* **Regular univariate:** e.g., daily sales of a single product.
* **Regular multivariate:** e.g., hourly temperature, pressure and humidity.
* **Panels:** e.g., sales across many stores and SKUs.
* **Hierarchical:** e.g., product → category → region.
* **High‑frequency:** e.g., sensor telemetry or tick data.
* **Irregular:** e.g., medical measurements or event logs.
* **Sparse:** e.g., new item demand or low‑volume processes.

### Normalization Strategies

Multiple reversible normalizers are provided:

- Local z‑score (subtract local mean, divide by local standard deviation).
- Robust scaling using median and median absolute deviation.
- Log and Box‑Cox transformations for positive‑valued processes.
- Relative change or return normalization for ratio‑scale variables.
- Seasonal residual normalization for series with strong seasonality.
- Entity‑level scale embeddings and frequency‑aware scaling.

Normalization statistics (mean, scale, transformation parameters) are retained to convert forecasts back to original units.  Domain‑specific normalizers can be implemented via a trait defined in the Rust codebase.

### Metadata & Covariates

Along with the primary target series, known future covariates (e.g., calendar features, planned promotions, weather forecasts) and historical covariates (e.g., past prices, previous traffic) may be present.  Static metadata (e.g., store type, region) and latent metadata inferred from historical behaviour are encoded and aligned with the main sequence.