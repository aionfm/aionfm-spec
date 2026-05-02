# Document 45 – Practical Deployment Examples

## Purpose

This document provides concrete examples of how AionFM can be deployed in real-world systems.  The examples illustrate end‑to‑end workflows including data ingestion, model configuration, forecasting, scenario generation, decision‑making and monitoring.  They demonstrate the integration between the core model and platform components (UI, API, SDK).

## Audience

Solution architects, system integrators and DevOps teams.

## Example 1: Retail Planning System

### Scenario

A large retailer wants to forecast daily demand for 10,000 SKUs across 500 stores to optimise inventory and promotions.  The system must generate median forecasts, prediction intervals, scenario paths for risk planning and interpretability outputs for planners.

### Deployment Steps

1. **Data ingestion:** ingest historical sales, prices, promotions, holidays, inventory levels and static metadata (store type, region, product category) into the data layer.
2. **Normalization:** apply reversible scaling per product and store; generate patches of length 28 days with stride 7 days.
3. **Model configuration:** select model size AionFM‑M, patch length 28, context 12 months, horizon 30 days.  Enable scenario generation with 20 paths and quantile outputs at 0.1, 0.5 and 0.9.
4. **Adapter training:** train store and product adapters on the retailer’s data for a few epochs.  Calibrate quantiles using the last six months of data.
5. **Forecasting:** call the API endpoint `/forecast` from the retailer’s planning software.  The API returns the JSON schema defined in Doc42, including point forecasts, quantiles, scenario paths and regime explanations.
6. **Decision support:** compute reorder quantities using the 0.90 quantile to ensure 90% service level.  Evaluate scenario paths to set promotion budgets and contingency plans.  Use regime explanations to identify products entering volatility expansion or seasonal peaks.
7. **Monitoring:** track WAPE and calibration metrics by store and product category.  Set alerts when coverage drops or demand spikes.

## Example 2: Energy Load Forecasting for a Grid Operator

### Scenario

A regional grid operator forecasts hourly electricity demand across multiple zones to schedule generation, manage reserves and detect risk of blackouts.  The operator requires probabilistic load profiles, scenario analysis under weather uncertainty and real‑time updates.

### Deployment Steps

1. **Data ingestion:** ingest hourly load, temperature, humidity, wind speed, calendar information and holiday markers.  Include zone metadata (population, industrial share).
2. **Normalization:** apply Box‑Cox transformation to stabilize variance; compute local z‑score for each zone.
3. **Model configuration:** choose AionFM‑L with context length 60 days and horizon 48 hours.  Use hourly patches of length 24 with stride 12.  Include weather forecasts as known future covariates.
4. **Retrieval module:** enable retrieval of analogous weather‑load patterns from previous seasons.  Use similarity on residual embeddings and temperature trends.
5. **Scenario generation:** request 50 scenarios representing different weather evolutions (e.g. cold snap, heat wave).  The scenario decoder generates plausible load trajectories conditioned on sampled regime tokens.
6. **Operational planning:** run unit commitment and dispatch models using the scenarios to allocate generation and reserves.  Use high quantile (e.g. 0.95) as worst‑case planning input.
7. **Real‑time updates:** update forecasts every hour as new load and weather observations arrive.  Monitoring tracks MAE, RMSE and coverage per zone; recalibrate adapters weekly.

## Example 3: Healthcare Capacity Planning

### Scenario

A hospital system forecasts daily emergency department arrivals and inpatient bed occupancy.  The model must account for seasonal patterns, flu outbreaks, holidays and day‑of‑week effects.  Forecasts guide staffing and resource allocation.

### Deployment Steps

1. **Data ingestion:** ingest historical admissions, weather data (temperature, flu activity), holidays, staffing levels and bed counts.  Include metadata for each hospital (capacity, location).
2. **Normalization:** compute median absolute deviation scaling; incorporate day‑of‑week and holiday embeddings.
3. **Model configuration:** choose AionFM‑S for low latency.  Set patch length 14 days with stride 7.  Forecast horizon 14 days.  Add hospital‑specific adapters.
4. **Scenario generation:** generate 10 scenarios to simulate surge events (e.g. flu spikes) and staffing disruptions.  Combine with patient flow models to estimate bed occupancy trajectories.
5. **Decision support:** allocate staff and resources based on median forecast plus safety buffer derived from high quantile.  Use scenario paths to plan contingency staffing and resource reallocation.
6. **Monitoring:** track forecast accuracy daily; calibrate quantiles monthly.  Monitor regime tokens to detect early signals of epidemic peaks.

## Lessons Learned

These examples highlight that successful deployment requires:

* Careful data preparation and normalization tailored to the domain.
* Appropriate model size and patch configuration based on horizon and data volume.
* Adapter training and calibration for domain specificity.
* Scenario generation to explore risk and plan contingencies.
* Continuous monitoring and recalibration to maintain performance.
