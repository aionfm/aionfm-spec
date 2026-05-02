# Document 39 – Calibration & Monitoring

## Purpose

This document defines procedures for **post‑deployment calibration** and **continuous monitoring** of AionFM.  Calibration ensures that the model’s probabilistic forecasts match observed frequencies, while monitoring tracks performance and drift over time.  Together these activities maintain trust and reliability in production.

## Audience

Operations teams, ML engineers, data scientists and reliability engineers responsible for running AionFM in production.

## Calibration Procedures

1. **Initial calibration:** after training or domain adaptation, evaluate quantile forecasts against a hold‑out validation set.  Compute prediction interval coverage (e.g. 50%, 80%, 90%) and compare to nominal levels.  Adjust calibration layers or post‑processing to correct systematic biases.

2. **Temporal recalibration:** calibration may drift as data distributions shift.  Use rolling windows (e.g. 30–60 days) to compute empirical coverage of recent forecasts.  Apply recalibration techniques such as isotonic regression, Platt scaling or quantile mapping to re‑align intervals.

3. **Cross‑entity calibration:** calibrate per entity or segment when data heterogeneity is high.  Entities with different volatility profiles may require separate calibration parameters.

4. **Interval width monitoring:** monitor average interval widths to detect under‑ or over‑confident forecasts.  Sharp intervals that are frequently violated indicate underestimation of uncertainty; overly wide intervals reduce utility.

## Monitoring Metrics

Monitoring spans **accuracy**, **calibration**, **regime distribution** and **drift**:

* **Point accuracy:** track metrics such as MAE, MASE, RMSE, sMAPE and WAPE by horizon, entity and segment.  Detect deterioration trends.
* **Quantile calibration:** compute pinball loss and coverage for each quantile level; measure quantile crossing rates (quantiles should not invert).  Use reliability diagrams to visualise calibration.
* **Scenario quality:** assess scenario coherence and diversity using metrics like energy distance or maximum mean discrepancy on sampled paths.
* **Regime distribution:** monitor the frequency of regime tokens to detect shifts toward rare or unseen behaviours.  Sudden changes in regime distribution may signal anomalies or structural breaks.
* **Data drift:** compare input distribution (mean, variance, seasonality) to training distribution.  Use population stability index or KL‑divergence to quantify drift.  Monitor covariate drift separately.
* **Missingness patterns:** track missingness rates and patterns; sudden increases may indicate sensor failures or data ingestion issues.

## Alerting and Thresholds

* Define thresholds for each metric (e.g. 10% degradation in MAE, 5% drop in coverage).  Trigger alerts when thresholds are exceeded.
* For regime distribution, set alarms when token frequencies deviate significantly from baseline (e.g. more than two standard deviations).
* Integrate alerts with incident management tools (PagerDuty, Slack) and dashboards.

## Recalibration Triggers

* Data drift beyond threshold.
* Calibration error consistently above allowed bounds.
* Appearance of previously unseen tokens or regimes.
* Operational events (e.g. major market shifts, policy changes) that require model adjustment.

## Deployment Integration

* Implement monitoring as part of the serving layer; metrics should be logged and aggregated per forecast request.
* Use a monitoring service (Prometheus, Grafana) to collect and visualise metrics.  Provide dashboards for operations teams to inspect performance by segment and time.
* Calibration adjustments can be applied via configuration without re‑training, using post‑processing functions or adapter adjustments.

## Documentation and Governance

* Document calibration and monitoring procedures, thresholds and responsibilities.  Keep an audit trail of calibration updates and model changes.
* In regulated domains (finance, healthcare), include calibration and monitoring results in compliance reports.
