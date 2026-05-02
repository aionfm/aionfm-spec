## Document 28 ‒ Evaluation Framework

### Purpose

To define the metrics and procedures used to evaluate AionFM’s forecasting performance.  Evaluation must assess point accuracy, probabilistic calibration, scenario realism, robustness and interpretability.  This document outlines recommended metrics, tests and reporting protocols for thorough evaluation across domains and deployment settings.

### Intended Audience

Researchers, evaluators and practitioners benchmarking AionFM or comparing it to other models.

### Key Sections

* **Point forecast metrics** – MAE, RMSE, MAPE, sMAPE, MASE, WAPE.
* **Probabilistic metrics** – pinball loss, continuous ranked probability score (CRPS), prediction interval coverage, interval width, calibration error.
* **Scenario quality metrics** – path coherence, distributional similarity, tail‑event coverage, regime transition realism, multivariate dependency preservation, stress scenario plausibility.
* **Robustness tests** – missing data, corruption, outliers, regime shifts, seasonal changes, covariate dropout, scale shifts, frequency changes.
* **Interpretability metrics** – regime token stability, alignment with known change points, explanation consistency, human evaluation.

### Point Forecast Metrics

Evaluate deterministic forecasts with:

* **Mean Absolute Error (MAE)** – average absolute difference between forecast and actual.
* **Root Mean Squared Error (RMSE)** – sensitive to large errors; emphasises extreme deviations.
* **Mean Absolute Percentage Error (MAPE)** and **symmetric MAPE (sMAPE)** – percentage errors; be cautious when actual values are near zero.
* **Mean Absolute Scaled Error (MASE)** – scale‑invariant metric allowing comparison across series.
* **Weighted Absolute Percentage Error (WAPE)** – emphasises high‑volume periods; common in retail.

### Probabilistic Metrics

Assess the quality of prediction intervals and distributions using:

* **Pinball loss** – quantile loss summarised across quantiles and horizons.
* **Continuous Ranked Probability Score (CRPS)** – integrates over all quantile levels to measure distribution accuracy.
* **Prediction interval coverage** – compare nominal coverage (e.g. 80%) to empirical coverage; compute calibration error.
* **Prediction interval width** – measure sharpness; narrower intervals are better if calibrated.
* **Weighted quantile loss** – emphasise upper or lower tails depending on risk preferences.

### Scenario Quality Metrics

Scenarios should be realistic and coherent:

* **Path coherence** – individual trajectories should not exhibit physically impossible jumps or negative values where invalid.
* **Distributional similarity** – the distribution of scenarios should match marginal quantiles and variance of the predictive distribution.
* **Tail‑event coverage** – scenarios should capture extreme events with appropriate frequency.
* **Regime transition realism** – regime paths should reflect plausible patterns; too many or too few transitions suggest issues.
* **Multivariate dependency preservation** – correlation structure between variables should be maintained.
* **Stress scenario plausibility** – user‑specified stress scenarios should reflect credible worst‑case conditions.

### Robustness Tests

Evaluate the model under perturbations:

* **Missing data** – randomly remove observations and assess imputation accuracy and forecast degradation.
* **Corrupted observations** – add noise or outliers to inputs; measure resilience.
* **Regime shifts** – simulate sudden changes in trend or variance and measure how quickly the model adapts.
* **Seasonal changes** – shift seasonal patterns to test adaptability to seasonality changes.
* **Covariate dropout** – remove covariates to test dependence on external signals.
* **Scale shifts** – multiply series by scale factors; verify scale invariance.
* **Frequency changes** – resample series at different frequencies and evaluate performance.

### Interpretability Metrics

Assess interpretability by:

* **Regime token stability** – track stability of regime classifications across similar series and repeated runs.
* **Alignment with known change points** – compare detected regime switches with annotated change points.
* **Explanation consistency** – evaluate whether explanations are consistent for similar patterns across domains.
* **Human evaluation** – have domain experts rate the plausibility of explanations and regime labels.

### Reporting Protocols

* Report metrics across multiple horizons and segments (entities, regions, categories).
* Include calibration plots and interval coverage tables.
* Provide scenario visualisations and distribution comparisons.
* Document robustness experiment settings and results.
* When comparing models, ensure identical data splits and covariate information.

A thorough evaluation framework ensures that AionFM’s strengths and weaknesses are understood and guides ongoing improvement.