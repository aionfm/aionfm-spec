## Document 12 ‒ Forecast Decomposition & Output Heads

### Purpose

This document defines how AionFM decomposes the forecasting problem into interpretable components and explains the design of the model’s output heads.  A core contribution of AionFM is its explicit separation of baseline, seasonal, event and residual dynamics and the provision of multiple forecasting heads that cover point forecasts, quantile predictions, full distributions, regime trajectories, scenario sampling and imputation.  By formalising the decomposition and output heads, modellers can implement the architecture consistently across domains.

### Intended Audience

Time‑series modellers and developers implementing the output layer of the AionFM core model.

### Key Sections

* **Decomposition rationale** – why separating baseline, seasonality, events and residual uncertainty improves robustness and interpretability.
* **Components defined** – formal definitions of the baseline, seasonal, event/shock and residual terms.
* **Output head overview** – description of each head (point, quantile, distribution, regime forecast, scenario generation, imputation) and its purpose.
* **Architectural considerations** – how the heads interact with the dual‑stream backbone and how they are trained jointly.

### Decomposition Rationale

AionFM treats the future as a combination of several latent components.  Rather than predicting raw values directly, the model learns to estimate:

* **Baseline trajectory** – a slowly varying trend or level capturing long‑term movements in the series.  This may include entity‑level growth or decline, captured by the memory mechanism and long‑horizon attention.
* **Seasonal component** – repeating patterns driven by calendar effects (daily, weekly, annual cycles, holidays) or known cyclic phenomena.
* **Event or shock component** – abrupt changes due to promotions, outages, regulatory changes, macro shocks or exogenous events.  These are captured by the regime token stream and event features.
* **Residual behavioural component** – local fluctuations not explained by the baseline, seasonal or event parts.  The residual may include volatility dynamics, regime‑switching behaviour and cross‑variable coupling.
* **Uncertainty envelope** – the irreducible uncertainty remaining after modelling the above components.  It informs prediction intervals and scenario dispersion.

Separating the forecast into these components improves robustness and interpretability.  For example, the seasonal term can be shared across entities within the same region, while the event term highlights shocks that require domain knowledge.  The residual component captures fine‑scale dynamics encoded through the regime tokens, and the uncertainty envelope is used to calibrate prediction intervals【594893450643871†L2238-L2252】.

### Components Defined

Let \(x_{t+h}\) denote the future observation at horizon \(h\).  AionFM predicts:

\[\hat{x}_{t+h} = \hat{b}_{t+h} + \hat{s}_{t+h} + \hat{e}_{t+h} + \hat{u}_{t+h},\]

where \(\hat{b}_{t+h}\) is the baseline, \(\hat{s}_{t+h}\) the seasonal component, \(\hat{e}_{t+h}\) the event/shock term and \(\hat{u}_{t+h}\) the residual/uncertainty term.  These components are not manually specified; the model learns them through auxiliary heads and multi‑objective training.  The decomposition encourages the network to disentangle systematic structure from noise.

### Output Heads

AionFM exposes several heads to serve diverse forecasting needs:

1. **Point forecast head** – produces a single deterministic trajectory, representing either the mean or median of the predictive distribution depending on training.  Useful for summary reporting and baseline planning.
2. **Quantile forecast head** – outputs calibrated quantiles (e.g. 1%, 5%, 10%, 25%, 50%, 75%, 90%, 95%, 99%) for each horizon.  These quantiles form prediction intervals and support risk‑aware planning.
3. **Distribution head** – optionally produces parameters of a parametric distribution (Gaussian, Student‑t, negative binomial, log‑normal or mixtures) appropriate to the domain.  This head supports sampling and computing likelihoods.
4. **Regime forecast head** – predicts the sequence of future behavioural tokens, enabling the model to describe likely upcoming regimes such as growth, reversal or volatility expansion.
5. **Scenario generation head** – generates multiple full trajectory samples by first sampling a path of future regime tokens and then decoding values conditioned on those regimes.  These scenarios support stress testing, risk simulation, inventory buffers and planning exercises.
6. **Imputation head** – fills missing values in historical windows, improving robustness when data is incomplete or corrupted.

### Architectural Considerations

The output heads share the dual‑stream representation produced by the causal backbone.  Each head has its own set of parameters but is trained jointly through the composite loss function.  The quantile and distribution heads rely on well‑calibrated uncertainty estimation, while the regime head leverages the discrete token vocabulary.  Scenario generation ties the discrete regime forecast to the continuous value decoder, ensuring coherent paths.  The imputation head uses masked reconstruction objectives to learn plausible values for missing data.  All heads can be configured per deployment, for example disabling the distribution head in a low‑resource setting or adjusting the number of scenarios returned.