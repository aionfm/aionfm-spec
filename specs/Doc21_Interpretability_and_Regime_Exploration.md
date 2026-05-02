## Document 21 ‒ Interpretability & Regime Exploration

### Purpose

To describe AionFM’s interpretability features and how analysts can explore and explain forecasts using regime tokens and related outputs.  Interpretability is crucial for trust, regulatory compliance and understanding the drivers of predictions.  This document details how the model exposes its internal regime classifications, change‑point probabilities, volatility states, residual anomalies and cross‑variable coupling scores.

### Intended Audience

Analysts, domain experts, data scientists and decision makers who require interpretable forecasts and explanations.

### Key Sections

* **Regime classifications** – interpreting macro, meso and micro tokens.
* **Change‑point likelihoods** – identifying structural breaks and regime transitions.
* **Volatility and uncertainty states** – understanding the evolution of uncertainty.
* **Residual anomalies** – detecting unexplained deviations.
* **Cross‑variable coupling** – measuring co‑movement and divergence across variables.
* **Explanation templates** – guidelines for generating human‑readable narratives from model outputs.

### Regime Classifications

The discrete regime tokenizer assigns tokens at multiple levels (macro, meso, micro) that correspond to broad trends (e.g. upward drift, seasonal decline), local dynamics (e.g. volatility expansion, compression) and residual details (e.g. positive spike, negative dip).  Analysts can map these tokens to interpretable labels and visualise regime sequences over time.  For example, a time series may transition from a “seasonal peak with high volatility” token to a “flat saturation” token as demand normalises.

### Change‑Point Likelihoods

By examining the probability of switching to a different macro regime at each step, one can identify potential change points.  High change‑point likelihood may indicate structural breaks, policy changes or emerging trends.  Analysts should monitor these probabilities and investigate underlying causes when sudden regime transitions occur.

### Volatility and Uncertainty States

Volatility expansion or compression tokens capture changes in variability.  AionFM can also provide volatility‑state estimates as continuous variables, which, combined with prediction intervals, offer insight into how uncertainty evolves over the forecast horizon.  For example, an expanding volatility state might explain why upper quantiles diverge faster than lower quantiles.

### Residual Anomalies

Residual anomalies occur when the model’s residual component exhibits behaviour not captured by baseline, seasonality or event terms.  Large residuals or unusual micro tokens may flag anomalies.  Analysts can investigate whether these anomalies correspond to data issues, unexpected events or model limitations.  Persistent anomalies may suggest missing covariates or structural changes.

### Cross‑Variable Coupling

In multivariate settings, AionFM can output scores describing cross‑variable coupling.  Coupling tokens indicate whether variables move together or diverge.  High coupling may reflect common drivers (e.g. temperature affecting energy and retail demand simultaneously), while decoupling may indicate regime shifts or idiosyncratic shocks.  These scores help analysts diagnose interdependencies and inform cross‑series modelling decisions.

### Explanation Templates

To communicate forecasts, produce narratives using the model outputs.  For example:

> *“Demand is forecast to remain above baseline for the next 10 days due to a seasonal peak and positive residual momentum.  However, volatility is expanding, so upper quantiles diverge faster than lower quantiles.  There is a moderate probability of a regime reversal after day 7 based on historical analogs.”*

or

> *“The model detects a transition from stable low‑variance behaviour to a high‑volatility regime.  Prediction intervals widen and the probability of a downturn increases.  Past examples of similar transitions suggest caution in capacity planning.”*

Analysts can create custom templates using the available outputs to explain results to stakeholders.  Clear explanations build trust and facilitate action on forecasts.