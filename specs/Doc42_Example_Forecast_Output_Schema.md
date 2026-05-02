# Document 42 – Example Forecast Output Schema

## Purpose

This document defines the structured output returned by the AionFM platform when users request forecasts.  The schema is designed for consistency across front‑end, API and SDK layers, enabling downstream systems to parse and display results.  The schema supports point forecasts, probabilistic estimates, scenario generation, regime probabilities and explanations.

## Audience

API developers, SDK developers and integrators.

## JSON Schema Overview

The output is a JSON object with the following top‑level fields:

| Field | Type | Description |
|------|------|-------------|
| `model` | string | Name of the model (e.g. “AionFM”). |
| `model_version` | string | Version identifier for the deployed model. |
| `forecast_horizon` | integer | Number of time steps forecasted. |
| `frequency` | string | Forecast frequency (e.g. “D” for daily, “H” for hourly). |
| `target` | string | Name of the target variable. |
| `point_forecast` | array of numbers | Deterministic forecast, typically the mean or median. Length equals `forecast_horizon`. |
| `quantiles` | object | Mapping from quantile level (string) to an array of numbers. Each array has length equal to `forecast_horizon`. |
| `prediction_intervals` | object | Mapping from interval level (string) to an object with `lower` and `upper` arrays. |
| `scenario_paths` | array of arrays | Each inner array is a sampled scenario path for the forecast horizon. The number of scenarios is configurable. |
| `regime_probabilities` | object | Mapping from regime label (string) to probability (number between 0 and 1). |
| `explanation` | object | Human‑readable explanation of the forecast, including current regime, uncertainty drivers and change‑point probability. |
| `metadata` | object | Optional additional information (e.g. entity identifiers, covariate summary, calibration status). |

## Example

```json
{
  "model": "AionFM",
  "model_version": "0.4",
  "forecast_horizon": 30,
  "frequency": "D",
  "target": "demand",
  "point_forecast": [124.1, 127.5, 129.0, ...],
  "quantiles": {
    "0.10": [101.2, 103.7, 105.1, ...],
    "0.50": [123.8, 126.9, 128.7, ...],
    "0.90": [149.4, 154.2, 159.8, ...]
  },
  "prediction_intervals": {
    "80": {
      "lower": [101.2, 103.7, 105.1, ...],
      "upper": [149.4, 154.2, 159.8, ...]
    },
    "95": {
      "lower": [90.5, 95.7, 97.4, ...],
      "upper": [160.3, 165.9, 171.2, ...]
    }
  },
  "scenario_paths": [
    [122.0, 126.1, 130.8, ...],
    [131.5, 139.2, 144.0, ...],
    [115.4, 112.7, 118.6, ...]
  ],
  "regime_probabilities": {
    "stable_growth": 0.42,
    "seasonal_peak": 0.27,
    "shock_recovery": 0.18,
    "reversal": 0.13
  },
  "explanation": {
    "current_regime": "stable_growth_with_positive_residual_momentum",
    "uncertainty_driver": "historical reversals after similar uplift regimes",
    "change_point_probability": 0.21
  },
  "metadata": {
    "entity_id": "store_42",
    "covariate_summary": { "temperature_mean": 15.4, "promotion": true },
    "calibration_status": "ok"
  }
}
```

## Schema Notes

* Quantiles should be specified as strings (e.g. "0.10") to preserve ordering and avoid floating point key issues.
* Prediction interval keys (e.g. "80", "95") indicate the coverage percentage and contain both lower and upper bounds.
* `scenario_paths` can be omitted when scenarios are not requested.  The number of scenarios is controlled by the request.
* `regime_probabilities` includes only the most relevant regimes; probabilities sum to 1 across reported regimes but the model may consider additional regimes internally.
* `explanation` fields are free‑text or numeric values summarizing the model’s reasoning.  Their structure may evolve in future versions.
* `metadata` is optional and can include arbitrary key‑value pairs; it is useful for linking forecasts back to the original request.
