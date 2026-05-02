## Document 22 ‒ Use Cases – Energy

### Purpose

To outline the specific requirements, covariates, horizons and evaluation metrics relevant to energy load forecasting with AionFM.  Energy planners require accurate, probabilistic forecasts of electricity demand to balance supply and demand, plan capacity and maintain grid stability.  This document details how AionFM can be applied to energy use cases.

### Intended Audience

Energy planners, utilities, data scientists and engineers deploying AionFM in energy contexts.

### Key Sections

* **Domain description** – unique characteristics of energy load time series.
* **Covariates** – weather, calendar and operational drivers.
* **Forecast horizons** – short‑term, mid‑term and long‑term horizons.
* **Evaluation metrics** – metrics commonly used in energy forecasting.
* **Scenario usage** – how energy planners can use scenarios for risk management.

### Domain Description

Electricity load data exhibit strong daily and weekly seasonality, significant dependence on temperature and weather, and occasional shocks due to outages or extreme weather events.  Demand can vary by region, industrial vs. residential usage and day type (weekday vs. weekend).  Grid operators often require forecasts at multiple aggregation levels (grid, region, substation) and must consider capacity constraints and regulatory requirements.

### Covariates

Relevant covariates include:

* **Weather forecasts** – temperature, humidity, wind speed, solar irradiance.  Load is highly sensitive to temperature; heating/cooling degree days can be derived as features.
* **Calendar features** – hour of day, day of week, holiday flags, season indicators.
* **Operational data** – scheduled maintenance, generation capacity, demand response events.
* **Regional metadata** – climate zone, urbanisation level, industrial composition.

### Forecast Horizons

Energy planners often require:

* **Short‑term forecasts** (hours to days ahead) for real‑time balancing and spot pricing.
* **Mid‑term forecasts** (days to weeks) for maintenance planning and resource scheduling.
* **Long‑term forecasts** (months to years) for capacity expansion and investment decisions.

AionFM’s long context and probabilistic outputs make it suitable for all three horizons.  For long‑term horizons, emphasise scenario generation and incorporate long‑range weather forecasts or macro indicators.

### Evaluation Metrics

Common metrics for energy forecasts include mean absolute error (MAE), root mean squared error (RMSE), mean absolute percentage error (MAPE), mean absolute scaled error (MASE) and prediction interval coverage.  Because extreme peaks and troughs matter for grid stability, evaluate tail performance using quantile coverage and scenario‑based stress tests.  Consider seasonal normalisation when comparing across months.

### Scenario Usage

Use AionFM’s scenarios to assess:

* **Peak demand risk** – simulate extreme temperature scenarios and plan reserve capacity accordingly.
* **Generation outages** – model scenarios where unplanned outages coincide with high demand.
* **Renewable variability** – integrate renewable generation forecasts and sample scenarios to understand variability and curtailment risk.
* **Policy changes** – simulate demand under new rate structures or demand‑response programmes.

By capturing uncertainty and regime dynamics, AionFM helps energy planners make robust, risk‑aware decisions.