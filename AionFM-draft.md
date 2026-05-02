# AionFM

## A Hybrid Continuous-Discrete Temporal Foundation Model for Probabilistic, Regime-Aware Time-Series Forecasting

**White Paper — Conceptual Technical Specification**

---

## Abstract

We introduce **AionFM**, a hybrid continuous-discrete time-series foundation model designed to unify zero-shot forecasting, probabilistic scenario generation, regime detection, and domain adaptation within a single decoder-only architecture. AionFM represents each sequence through two synchronized streams: continuous **value patches** that preserve numerical precision, and discrete **regime tokens** that encode trend, volatility, seasonality, discontinuities, missingness, and cross-variable structural states. Rather than relying solely on direct value regression or full discretization, AionFM learns a **residual temporal language**: it first estimates a coarse future trajectory, then models uncertainty and local behavioral shifts through symbolic residual states, and finally refines the forecast with a conditional quantile-residual decoder. This design preserves metric accuracy while enabling compositional reasoning over temporal patterns. Pretraining combines next-patch prediction, masked temporal reconstruction, cross-scale contrastive alignment, synthetic trajectory augmentation, and stochastic rollout consistency across heterogeneous real and simulated time-series corpora. At inference, AionFM generates point forecasts, calibrated quantiles, and multiple plausible future paths from the same historical context, while optional lightweight adapters incorporate exogenous variables, calendar structure, entity metadata, or sensor descriptors. The model is intended as a general-purpose temporal foundation model: long-context, distribution-aware, robust to nonstationarity, interpretable through learned regime states, and applicable across finance, energy, retail, logistics, mobility, healthcare operations, climate, and industrial telemetry without requiring task-specific architectural redesign.

---

# 1. Executive Summary

Modern forecasting systems are often optimized for a narrow setting: one dataset, one domain, one horizon, one sampling frequency, or one business objective. A model trained for hourly energy demand may not transfer well to retail sales. A financial volatility model may fail on industrial sensor telemetry. A demand forecasting system that produces a single expected value may be inadequate for inventory planning, risk control, or capacity management, where calibrated uncertainty is often more important than the mean forecast.

**AionFM** is proposed as a general-purpose time-series foundation model designed to address these limitations. Its central idea is that time series should be modeled at two levels simultaneously:

1. **Numerical level** — preserving exact values, magnitudes, scales, slopes, and local variation.
2. **Behavioral level** — representing the underlying state of the sequence: trend, seasonality, volatility, regime shift, shock, decay, recovery, saturation, anomaly, and cross-variable coupling.

AionFM introduces a dual representation:

* **Continuous value patches** encode raw or normalized temporal windows as dense numerical embeddings.
* **Discrete regime tokens** encode learned symbolic states describing temporal behavior.

The model then uses a decoder-only temporal backbone to forecast future values and future regimes jointly. This allows it to produce not only a forecast, but also an explanation of the kind of temporal behavior it believes is unfolding.

The result is a model that can operate as:

* A point forecaster.
* A probabilistic forecaster.
* A scenario generator.
* A regime detector.
* A missing-data imputer.
* A synthetic time-series generator.
* A domain-adaptable forecasting engine.

The core contribution of AionFM is its **residual temporal language**: instead of discretizing raw observations directly, the model tokenizes how a sequence is behaving relative to its recent context. This makes the token space more transferable across domains because concepts such as “volatility expansion,” “seasonal uplift,” “trend reversal,” “shock recovery,” or “flat saturation” can appear in finance, energy, retail, traffic, climate, and operations data even when the raw magnitudes differ drastically.

---

# 2. Motivation

## 2.1 The limits of conventional forecasting

Traditional forecasting systems tend to fall into several categories:

* Statistical models such as exponential smoothing, autoregressive models, state-space models, and structural time-series models.
* Machine learning models such as gradient boosting, random forests, and generalized additive models.
* Deep learning models such as recurrent networks, temporal convolutional networks, attention-based models, and sequence-to-sequence Transformers.
* Domain-specific forecasting engines designed around a particular data type or industry.

These approaches can perform well when the training distribution is stable and the deployment setting is close to the modeling assumptions. However, practical time-series problems often involve:

* Nonstationarity.
* Structural breaks.
* Calendar effects.
* Irregular sampling.
* Missing observations.
* Multiple seasonalities.
* Exogenous shocks.
* Cross-series dependencies.
* Sparse historical data.
* Long-context dependencies.
* High uncertainty over long horizons.
* Distributional drift between training and deployment.

A single expected forecast is often insufficient. Real decision-making requires a distribution over possible futures.

For example:

* An energy grid operator needs plausible peak-load scenarios, not only average demand.
* A retailer needs uncertainty ranges to decide inventory buffers.
* A hospital operations team needs capacity risk bands.
* A trading or risk system needs scenario paths, not just one predicted price.
* A logistics platform needs route-volume forecasts under disruptions.
* A climate or sensor system needs anomaly-aware projections.

AionFM is designed for these settings.

---

# 3. Design Philosophy

AionFM is built around six principles.

## 3.1 Preserve numerical precision

Time-series forecasting is not language modeling. In many applications, small numerical differences matter. A model that loses metric precision during tokenization can produce forecasts that are behaviorally plausible but numerically unreliable.

AionFM therefore maintains a continuous value stream throughout the architecture.

## 3.2 Learn behavioral structure

Raw values alone do not expose the state of the process. Two sequences can have different scales but similar dynamics. For example, a temperature sensor, stock return series, and server-load curve may all exhibit a sudden shock followed by mean reversion.

AionFM learns discrete regime tokens that describe temporal behavior independent of absolute scale.

## 3.3 Model uncertainty as a first-class object

AionFM does not treat uncertainty as an afterthought. It is trained to produce:

* Quantile forecasts.
* Prediction intervals.
* Distributional parameters.
* Sampled future trajectories.
* Regime-conditioned scenario paths.

This allows users to reason about risk, not just expected outcomes.

## 3.4 Separate baseline from residual dynamics

Many time series consist of a predictable baseline plus irregular residual structure. AionFM explicitly decomposes forecasting into:

* Baseline trajectory.
* Seasonal component.
* Event or shock component.
* Residual behavioral component.
* Uncertainty envelope.

This decomposition improves robustness and interpretability.

## 3.5 Support long context efficiently

Many temporal dependencies extend beyond the immediate input window. AionFM uses a combination of local patch attention and compressed recurrent memory summaries, allowing it to preserve information about older regimes without forcing every historical timestamp into full attention.

## 3.6 Adapt without redesign

AionFM is intended to support multiple domains through lightweight adapters rather than domain-specific architecture changes. The same core model can ingest different covariates, frequencies, metadata, and variable structures.

---

# 4. Core Concept: The Residual Temporal Language

The defining idea of AionFM is the **residual temporal language**.

Instead of turning raw observations directly into discrete tokens, AionFM first models local context and then tokenizes residual behavior relative to that context.

Given a time series:

[
x_{1:T} = {x_1, x_2, ..., x_T}
]

the model estimates a local baseline:

[
b_t = f_{\text{baseline}}(x_{1:t})
]

and computes a normalized residual representation:

[
e_t = \frac{x_t - b_t}{s_t + \epsilon}
]

where:

* (b_t) is a learned or estimated baseline.
* (s_t) is a local scale estimate.
* (\epsilon) is a numerical stabilizer.

The residual stream is then summarized into behavioral descriptors such as:

* Direction.
* Curvature.
* Volatility.
* Range expansion.
* Seasonality phase.
* Local acceleration.
* Shock magnitude.
* Recovery rate.
* Missingness structure.
* Cross-variable divergence.
* Correlation change.
* Distributional asymmetry.

These descriptors are mapped into discrete regime tokens. A token does not mean “the value is 17.2.” It means something closer to:

* “Stable upward drift.”
* “High-volatility expansion.”
* “Negative shock with partial recovery.”
* “Seasonal peak with residual overshoot.”
* “Flat saturation with low variance.”
* “Cross-variable decoupling.”
* “Abrupt level shift.”
* “Mean-reverting residual compression.”

This abstraction allows AionFM to transfer temporal knowledge across scales and domains.

---

# 5. Model Overview

AionFM contains five major subsystems:

1. **Input normalization and temporal preparation**
2. **Continuous value patch encoder**
3. **Discrete regime tokenizer**
4. **Dual-stream causal temporal backbone**
5. **Probabilistic forecast and scenario decoder**

The simplified flow is:

```text
Raw time series
     ↓
Normalization, metadata alignment, missingness encoding
     ↓
Continuous value patches  +  discrete regime tokens
     ↓
Dual-stream causal Transformer
     ↓
Baseline forecast + residual forecast + regime forecast
     ↓
Point predictions, quantiles, intervals, and sampled scenarios
```

---

# 6. Input Representation

AionFM supports univariate and multivariate time series.

For an entity (i), variable (j), and time (t), the observed value is:

[
x_{i,j,t}
]

The input package may include:

* Historical target values.
* Known future covariates.
* Historical covariates.
* Static metadata.
* Calendar features.
* Frequency indicators.
* Missingness masks.
* Entity identifiers.
* Domain descriptors.
* Sensor or asset descriptors.
* Optional event indicators.

The model is designed to handle several data regimes:

| Data Type            | Example                                    |
| -------------------- | ------------------------------------------ |
| Regular univariate   | Daily sales of one product                 |
| Regular multivariate | Hourly temperature, pressure, and humidity |
| Panel data           | Sales for many stores and SKUs             |
| Hierarchical data    | Product → category → region                |
| High-frequency data  | Sensor or market telemetry                 |
| Irregular data       | Medical measurements or event logs         |
| Sparse data          | New item demand or low-volume operations   |

---

# 7. Normalization Strategy

AionFM uses reversible normalization to improve cross-domain transfer.

For each series or variable, it computes local statistics:

[
\mu_t = \text{local mean}
]

[
\sigma_t = \text{local scale}
]

and transforms values as:

[
z_t = \frac{x_t - \mu_t}{\sigma_t + \epsilon}
]

However, AionFM does not rely on a single normalization method. It supports multiple normalizers:

* Local z-score normalization.
* Robust median absolute deviation scaling.
* Log or Box-Cox-style transformations where valid.
* Relative-return normalization for positive-valued processes.
* Seasonal residual normalization.
* Entity-level scale embeddings.
* Frequency-aware scaling.

The normalization parameters are retained so outputs can be mapped back to original units.

This matters because the same model may need to forecast:

* Megawatt demand.
* Store sales.
* Sensor vibration.
* Website traffic.
* Patient volume.
* Market volume.
* Rainfall intensity.
* Manufacturing defects.

The scales are incomparable, but the temporal behaviors may be related.

---

# 8. Continuous Value Patch Encoder

AionFM divides historical sequences into patches.

Given a patch length (P) and stride (S), a patch is:

[
p_k = [z_{kS}, z_{kS+1}, ..., z_{kS+P-1}]
]

Each patch is projected into a dense embedding:

[
h^{v}_k = W_v p_k + b_v
]

where (h^{v}_k) is the continuous value embedding.

For multivariate data, the patch may include:

* Multiple target variables.
* Covariates.
* Missingness masks.
* Time features.
* Static metadata projections.
* Variable identity embeddings.

The value patch encoder is responsible for preserving fine numerical detail. It captures:

* Local slopes.
* Short-term variation.
* Magnitude.
* Scale-adjusted movement.
* Cross-variable patterns.
* Missingness-aware local structure.

Unlike pure tokenization, this pathway allows AionFM to retain high-resolution numerical information.

---

# 9. Discrete Regime Tokenizer

The discrete tokenizer produces behavioral tokens from local temporal summaries.

For each patch, AionFM computes a behavioral feature vector:

[
g_k = \phi(p_k, p_{k-1}, ..., p_{k-r})
]

where (\phi) extracts or learns features such as:

* Mean shift.
* Slope.
* Curvature.
* Volatility.
* Range.
* Autocorrelation.
* Seasonality phase.
* Residual sign.
* Residual magnitude.
* Missingness pattern.
* Cross-variable correlation.
* Distributional asymmetry.
* Change-point likelihood.

The tokenizer maps this representation into one or more discrete codes:

[
c_k = Q(g_k)
]

where (Q) is a learned quantizer.

AionFM uses a hierarchical token structure:

```text
Macro token: broad regime
    ↓
Meso token: local dynamic pattern
    ↓
Micro token: residual detail
```

For example:

```text
Macro: upward trend
Meso: volatility expansion
Micro: positive residual spike
```

or:

```text
Macro: seasonal decline
Meso: stable low variance
Micro: mild negative deviation
```

This hierarchy allows the model to reason at different temporal resolutions.

---

# 10. Why Behavioral Tokens Instead of Raw-Value Tokens?

Raw-value tokenization has a major drawback: values are domain-specific.

A value of `100` means something different across domains:

* 100 megawatts.
* 100 units sold.
* 100 dollars.
* 100 patients.
* 100 degrees.
* 100 requests per second.

A behavior such as “sudden positive shock followed by decay” is more transferable.

AionFM therefore tokenizes temporal behavior, not raw magnitude.

This creates a model vocabulary that can encode patterns such as:

| Behavioral Pattern | Description                     |
| ------------------ | ------------------------------- |
| Drift              | Persistent directional movement |
| Reversal           | Change in directional tendency  |
| Shock              | Abrupt deviation from baseline  |
| Recovery           | Return toward prior level       |
| Compression        | Declining variance              |
| Expansion          | Increasing variance             |
| Saturation         | Flattening after growth         |
| Seasonality peak   | Periodic high point             |
| Seasonality trough | Periodic low point              |
| Regime break       | Structural change               |
| Coupling           | Variables moving together       |
| Decoupling         | Variables diverging             |
| Missing burst      | Contiguous missing segment      |
| Anomalous spike    | Isolated extreme value          |

The model can then learn how these behaviors evolve over time.

---

# 11. Dual-Stream Causal Backbone

AionFM uses a decoder-only causal temporal backbone.

At each patch index (k), the model receives:

* Continuous value embedding (h^{v}_k)
* Discrete regime embedding (h^{r}_k)
* Time embedding (h^{t}_k)
* Variable or entity embedding (h^{m}_k)
* Optional covariate embedding (h^{c}_k)

These are combined through gated fusion:

[
h_k = \alpha_k h^{v}_k + \beta_k h^{r}_k + \gamma_k h^{c}_k + h^{t}_k + h^{m}_k
]

where (\alpha_k), (\beta_k), and (\gamma_k) are learned gates.

The causal backbone then models:

[
H_{1:K} = \text{CausalTransformer}(h_1, h_2, ..., h_K)
]

The causal structure ensures that the representation at patch (k) cannot see future patches during training.

The backbone supports:

* Long-context patch attention.
* Local temporal attention.
* Cross-variable attention.
* Regime-token attention.
* Memory attention.
* Covariate-aware attention.
* Frequency-aware position encoding.

---

# 12. Temporal Position Encoding

Time series require richer temporal encoding than simple token position.

AionFM combines several temporal signals:

1. **Absolute position**
   The location of the patch within the context.

2. **Relative position**
   Distance between historical patches.

3. **Calendar position**
   Hour, day, week, month, quarter, holiday, or other known calendar features.

4. **Frequency embedding**
   Whether the data is minutely, hourly, daily, weekly, monthly, or irregular.

5. **Elapsed-time embedding**
   Useful for irregularly sampled series.

6. **Seasonal phase embedding**
   Encodes repeating cycles.

7. **Forecast-horizon embedding**
   Allows the model to distinguish near-term from long-term prediction steps.

This enables the model to handle datasets with different temporal granularities.

---

# 13. Memory Mechanism for Long Context

AionFM uses compressed temporal memory to preserve long-range information.

Historical context is divided into:

* Active window.
* Recent memory.
* Long-term memory.

The active window receives full attention. Older history is summarized into memory states:

[
m_l = \psi(H_{a:b})
]

where (\psi) is a learned compression function over a historical segment.

Memory states capture:

* Long-term trend.
* Historical seasonality.
* Prior shocks.
* Regime durations.
* Recurring anomaly patterns.
* Historical volatility bands.
* Entity-specific behavior.
* Slow-moving baseline changes.

During inference, the model attends to these memory states without needing to attend to every historical observation.

This is especially useful for:

* Long seasonal patterns.
* Annual cycles.
* Multi-year demand history.
* Industrial wear patterns.
* Slow drift in sensors.
* Long-horizon financial or economic sequences.
* Climate and environmental data.

---

# 14. Forecast Decomposition

AionFM forecasts the future as a decomposition:

[
\hat{x}*{t+h} =
\hat{b}*{t+h}
+
\hat{s}*{t+h}
+
\hat{e}*{t+h}
+
\hat{u}_{t+h}
]

where:

* (\hat{b}_{t+h}) is the baseline component.
* (\hat{s}_{t+h}) is the seasonal component.
* (\hat{e}_{t+h}) is the event or shock component.
* (\hat{u}_{t+h}) is the residual uncertainty component.

This decomposition is not required to be manually specified. It is learned through auxiliary heads and training objectives.

The model outputs:

* Mean forecast.
* Median forecast.
* Quantile forecasts.
* Prediction intervals.
* Regime probabilities.
* Scenario trajectories.
* Optional component forecasts.

---

# 15. Output Heads

AionFM contains several output heads.

## 15.1 Point forecast head

Produces a deterministic forecast:

[
\hat{x}_{t+1:t+H}
]

This can represent the mean or median forecast depending on training configuration.

## 15.2 Quantile forecast head

Produces calibrated quantiles:

[
\hat{q}_{\tau, t+h}
]

for quantile levels such as:

[
\tau \in {0.01, 0.05, 0.10, 0.25, 0.50, 0.75, 0.90, 0.95, 0.99}
]

This allows the model to produce prediction intervals.

## 15.3 Distribution head

Optionally predicts parameters of a distribution, such as:

* Gaussian.
* Student-t.
* Negative binomial.
* Log-normal.
* Mixture distributions.
* Zero-inflated distributions.

The distribution type may be selected by domain or learned through a mixture head.

## 15.4 Regime forecast head

Predicts future behavioral tokens:

[
\hat{c}_{t+1:t+H}
]

This allows the model to describe likely future regimes.

## 15.5 Scenario generation head

Samples multiple possible futures:

[
\tilde{x}^{(n)}_{t+1:t+H}
]

for scenario index (n).

These scenarios can be used for:

* Risk simulation.
* Inventory buffers.
* Stress testing.
* Capacity planning.
* Portfolio analysis.
* Anomaly anticipation.
* Operational contingency planning.

## 15.6 Imputation head

Predicts missing values in historical windows.

This makes AionFM useful when historical data is incomplete or corrupted.

---

# 16. Training Objectives

AionFM is trained with a composite objective:

[
\mathcal{L}
===========

\lambda_1 \mathcal{L}*{\text{next}}
+
\lambda_2 \mathcal{L}*{\text{mask}}
+
\lambda_3 \mathcal{L}*{\text{token}}
+
\lambda_4 \mathcal{L}*{\text{quantile}}
+
\lambda_5 \mathcal{L}*{\text{rollout}}
+
\lambda_6 \mathcal{L}*{\text{contrast}}
+
\lambda_7 \mathcal{L}_{\text{calibration}}
]

Each component serves a distinct purpose.

## 16.1 Next-patch prediction

The model predicts future continuous patches from historical context.

[
\mathcal{L}*{\text{next}} =
|p*{k+1} - \hat{p}_{k+1}|
]

This trains the basic forecasting capability.

## 16.2 Masked temporal reconstruction

Random patches are masked, and the model reconstructs them.

This improves:

* Imputation.
* Robustness to missing data.
* Internal representation quality.
* Noise tolerance.

## 16.3 Regime-token likelihood

The model predicts future behavioral tokens.

[
\mathcal{L}*{\text{token}} =
-\sum_k \log P(c_k | c*{<k}, p_{\leq k})
]

This trains the residual temporal language.

## 16.4 Quantile loss

For quantile level (\tau), the pinball loss is:

[
\mathcal{L}*{\tau}(y, \hat{q}*{\tau})
=====================================

\max(\tau(y-\hat{q}*{\tau}), (\tau-1)(y-\hat{q}*{\tau}))
]

This encourages calibrated uncertainty bands.

## 16.5 Rollout consistency

The model is trained not only to predict one step ahead, but also to remain coherent across multi-step generated futures.

This reduces compounding error in long-horizon forecasts.

## 16.6 Cross-scale contrastive alignment

AionFM aligns short-window, medium-window, and long-window representations so that the model can connect local movements to broader regimes.

For example:

* A local spike may be noise in one context.
* The same spike may be the start of a structural break in another context.

Cross-scale contrastive learning helps the model distinguish these cases.

## 16.7 Calibration loss

The model is penalized when its prediction intervals are systematically too narrow or too wide.

For example, the empirical coverage of a 90% interval should be close to 90%.

---

# 17. Pretraining Data Strategy

AionFM is designed for broad pretraining across heterogeneous time-series corpora.

The pretraining mixture may include:

* Retail sales.
* Energy demand.
* Web traffic.
* Industrial sensors.
* Weather and environmental measurements.
* Transportation and mobility flows.
* Healthcare operations.
* Financial market data.
* Economic indicators.
* Manufacturing telemetry.
* Supply-chain flows.
* Synthetic time series.
* Simulated regime-switching systems.
* Event-driven temporal processes.

The model benefits from both real and synthetic data.

## 17.1 Real data

Real data provides natural complexity:

* Human behavior.
* Operational constraints.
* Calendar structure.
* Market feedback.
* Sensor noise.
* Physical constraints.
* Domain-specific patterns.

## 17.2 Synthetic data

Synthetic data allows the model to see controlled patterns that may be rare in real data:

* Abrupt regime changes.
* Volatility explosions.
* Long seasonal cycles.
* Missing-data bursts.
* Rare shocks.
* Multi-stage recoveries.
* Saturating growth.
* Cascading failures.
* Structural breaks.
* Counterfactual futures.

Synthetic data is not used to replace real data. It is used to improve coverage of temporal behaviors.

---

# 18. Synthetic Trajectory Generator

AionFM’s training pipeline includes a synthetic trajectory generator capable of producing time series with known latent structure.

A generated series may combine:

[
x_t =
T_t + S_t + C_t + E_t + R_t + \epsilon_t
]

where:

* (T_t) is trend.
* (S_t) is seasonality.
* (C_t) is cycle.
* (E_t) is event effect.
* (R_t) is regime process.
* (\epsilon_t) is noise.

Synthetic generation can simulate:

* Linear trends.
* Piecewise trends.
* Exponential growth.
* Logistic saturation.
* Multiple seasonalities.
* Random shocks.
* Step changes.
* Mean reversion.
* Heteroskedasticity.
* Autoregressive noise.
* Sparse demand.
* Count processes.
* Intermittent zeros.
* Irregular sampling.
* Sensor drift.
* Correlated multivariate systems.

Because the synthetic generator exposes latent regimes, it can help train the discrete regime tokenizer and evaluate interpretability.

---

# 19. Domain Adaptation

AionFM is intended to adapt to domains without retraining the entire model.

It supports several adaptation modes.

## 19.1 Zero-shot forecasting

The model forecasts a new time series using only historical context and optional covariates.

This is appropriate when:

* No labeled training set is available.
* Fast deployment is required.
* The domain is similar to pretraining data.
* The forecast horizon is standard.

## 19.2 Few-shot calibration

A small amount of domain-specific data is used to calibrate:

* Scale.
* Quantile coverage.
* Seasonal phase.
* Residual distribution.
* Covariate effects.

## 19.3 Adapter tuning

Lightweight adapter modules are trained for a specific domain while the core model remains frozen.

Adapters can specialize for:

* Frequency.
* Industry.
* Entity type.
* Sensor type.
* Market type.
* Geographic region.
* Product category.
* Operational process.

## 19.4 Full fine-tuning

For highly specialized or high-value applications, the full model may be fine-tuned on a domain-specific corpus.

This is most useful when:

* The domain has abundant data.
* The data distribution is unusual.
* The cost of forecasting error is high.
* The target task differs from general forecasting.

---

# 20. Covariate Integration

AionFM supports multiple covariate types.

## 20.1 Known future covariates

These are known for the forecast horizon:

* Calendar features.
* Planned promotions.
* Scheduled maintenance.
* Holidays.
* Price changes.
* Capacity changes.
* Weather forecasts.
* Policy schedules.
* Event calendars.

## 20.2 Historical covariates

These are observed only in the past:

* Actual weather.
* Historical prices.
* Past traffic.
* Past inventory.
* Historical sensor readings.
* Realized economic values.

## 20.3 Static metadata

These describe the entity being forecast:

* Store type.
* Region.
* Product category.
* Sensor location.
* Machine type.
* Asset class.
* Customer segment.
* Facility characteristics.

## 20.4 Latent metadata

The model can infer latent entity descriptors from historical behavior. For example, two products may share a seasonal demand shape even if their explicit metadata differs.

---

# 21. Handling Missing and Irregular Data

AionFM treats missingness as information rather than only as a nuisance.

The input includes missingness masks:

[
m_t =
\begin{cases}
1 & \text{if observed} \
0 & \text{if missing}
\end{cases}
]

Missingness patterns may themselves indicate events:

* Sensor outage.
* Market closure.
* Store closure.
* System downtime.
* Data ingestion failure.
* Operational disruption.

For irregular time series, AionFM includes elapsed-time embeddings:

[
\Delta t_k = t_k - t_{k-1}
]

This allows the model to distinguish between:

* No change over a short interval.
* No observation over a long interval.
* True flat behavior.
* Missing behavior.

---

# 22. Probabilistic Forecasting

AionFM treats forecasting as estimating a conditional distribution:

[
P(x_{t+1:t+H} | x_{1:t}, c_{1:t}, z_{1:t}, m_{1:t})
]

where:

* (x) is the observed series.
* (c) is the regime-token sequence.
* (z) represents covariates.
* (m) represents missingness masks.
* (H) is the forecast horizon.

The model can produce:

* Marginal quantiles for each horizon.
* Joint trajectory samples.
* Regime-conditioned scenarios.
* Prediction intervals.
* Tail-risk estimates.

This is important because marginal uncertainty and trajectory uncertainty are not the same.

For example, a forecast may have reasonable per-horizon intervals but generate unrealistic path shapes. AionFM’s scenario decoder is trained to preserve temporal coherence across sampled futures.

---

# 23. Scenario Generation

AionFM can generate future scenarios in two stages.

## 23.1 Stage 1: Future regime sampling

The model samples a sequence of future behavioral tokens:

[
\tilde{c}*{t+1:t+H}
\sim
P(c*{t+1:t+H} | x_{1:t})
]

This produces possible future behavioral paths, such as:

* Continued growth.
* Reversal.
* Shock and recovery.
* Volatility expansion.
* Seasonal decline.
* Stable plateau.

## 23.2 Stage 2: Value decoding

The model then decodes values conditioned on the sampled regimes:

[
\tilde{x}*{t+1:t+H}
\sim
P(x*{t+1:t+H} | \tilde{c}*{t+1:t+H}, x*{1:t})
]

This yields coherent future trajectories.

The user can request:

* Conservative scenarios.
* Median scenarios.
* Optimistic scenarios.
* Tail-risk scenarios.
* Stress scenarios.
* Regime-specific scenarios.

---

# 24. Interpretability

AionFM is designed to expose interpretable intermediate structure.

The model can provide:

* Current regime classification.
* Future regime probabilities.
* Change-point likelihood.
* Volatility-state estimate.
* Seasonal phase estimate.
* Residual anomaly score.
* Cross-variable coupling score.
* Forecast component breakdown.
* Scenario explanation.

For example, a forecast may include an explanation such as:

```text
The model expects demand to remain above baseline for the next 10 days,
primarily due to a learned seasonal peak and recent positive residual momentum.
Uncertainty widens after day 7 because historical analogs show frequent reversal
after similar uplift regimes.
```

Or:

```text
The model identifies a transition from stable low-volatility behavior to an
expansion regime. The upper quantiles increase faster than the median, indicating
asymmetric upside uncertainty.
```

This interpretability is enabled by the discrete behavioral token stream.

---

# 25. Example Use Cases

## 25.1 Energy demand forecasting

AionFM can forecast electricity load using:

* Historical load.
* Temperature.
* Calendar effects.
* Holidays.
* Weather forecasts.
* Regional metadata.

Outputs may include:

* Peak demand intervals.
* Extreme-load scenarios.
* Seasonal regime detection.
* Anomaly identification.
* Capacity-risk estimates.

## 25.2 Retail demand planning

For retail, AionFM can model:

* Product sales.
* Promotions.
* Price changes.
* Store effects.
* Holidays.
* Inventory constraints.
* New-product cold starts.

Outputs may include:

* SKU-level demand forecasts.
* Quantile-based reorder buffers.
* Promotion uplift scenarios.
* Stockout-risk forecasts.
* Hierarchical reconciliation signals.

## 25.3 Industrial telemetry

For industrial systems, AionFM can forecast:

* Temperature.
* Pressure.
* Vibration.
* Throughput.
* Failure indicators.
* Quality metrics.

Outputs may include:

* Predictive maintenance signals.
* Drift detection.
* Sensor anomaly forecasts.
* Failure-regime probabilities.
* Remaining-normal-behavior scenarios.

## 25.4 Healthcare operations

AionFM can support:

* Emergency department arrivals.
* Bed occupancy.
* Staffing demand.
* Procedure volume.
* Supply usage.
* Patient-flow bottlenecks.

Outputs may include:

* Demand intervals.
* Surge scenarios.
* Capacity-risk flags.
* Calendar-adjusted projections.

## 25.5 Logistics and mobility

AionFM can model:

* Shipment volume.
* Route congestion.
* Delivery times.
* Fleet utilization.
* Warehouse throughput.
* Passenger demand.

Outputs may include:

* Volume forecasts.
* Delay-risk scenarios.
* Capacity planning intervals.
* Disruption-sensitive forecasts.

## 25.6 Financial and economic sequences

AionFM can model:

* Returns.
* Volume.
* Volatility.
* Liquidity proxies.
* Economic indicators.
* Cross-asset dependencies.

Outputs may include:

* Risk scenarios.
* Volatility forecasts.
* Regime probabilities.
* Stress paths.
* Tail-risk estimates.

---

# 26. Evaluation Framework

AionFM should be evaluated across accuracy, calibration, robustness, transfer, and interpretability.

## 26.1 Point forecast metrics

Common point metrics include:

* MAE.
* RMSE.
* MAPE.
* sMAPE.
* MASE.
* WAPE.

No single metric is sufficient. Metric choice should reflect the domain.

For example:

* RMSE penalizes large errors more strongly.
* MAE is more robust to outliers.
* MASE supports cross-series comparison.
* WAPE is common in demand forecasting.
* MAPE can fail when actual values are near zero.

## 26.2 Probabilistic metrics

Probabilistic metrics include:

* Pinball loss.
* CRPS.
* Prediction interval coverage.
* Prediction interval width.
* Calibration error.
* Weighted quantile loss.

AionFM should be evaluated not only on whether the median is accurate, but whether uncertainty estimates are reliable.

## 26.3 Scenario quality metrics

Scenario generation should be evaluated using:

* Path coherence.
* Distributional similarity.
* Tail-event coverage.
* Regime transition realism.
* Multivariate dependency preservation.
* Stress scenario plausibility.

## 26.4 Robustness metrics

Robustness tests should include:

* Missing data.
* Corrupted observations.
* Outliers.
* Regime shifts.
* Seasonal changes.
* Covariate dropout.
* Scale shifts.
* Frequency changes.

## 26.5 Transfer metrics

Transfer evaluation should test:

* Zero-shot forecasting.
* Few-shot adaptation.
* Domain transfer.
* Frequency transfer.
* Long-horizon generalization.
* New-entity cold starts.

## 26.6 Interpretability metrics

Interpretability can be evaluated through:

* Regime-token stability.
* Alignment with known change points.
* Consistency of regime explanations.
* Human expert evaluation.
* Counterfactual sensitivity.
* Component attribution fidelity.

---

# 27. Benchmarking Protocol

AionFM should be benchmarked under several deployment-like settings.

## 27.1 Zero-shot benchmark

The model is evaluated on datasets not used during training.

Purpose:

* Measure foundation-model transfer.
* Test broad generalization.
* Identify domain blind spots.

## 27.2 Fine-tuned benchmark

The model is fine-tuned on domain-specific historical data.

Purpose:

* Measure adaptation ceiling.
* Compare with specialized models.
* Evaluate data efficiency.

## 27.3 Few-shot benchmark

The model receives a small amount of target-domain data.

Purpose:

* Evaluate cold-start utility.
* Measure sample efficiency.
* Test adapter effectiveness.

## 27.4 Long-horizon benchmark

The model forecasts far into the future.

Purpose:

* Test rollout stability.
* Evaluate uncertainty widening.
* Detect compounding error.

## 27.5 Stress benchmark

The input contains shocks, missing data, distribution shifts, and rare regimes.

Purpose:

* Test robustness.
* Evaluate scenario usefulness.
* Identify failure modes.

---

# 28. Deployment Architecture

AionFM can be deployed in several modes.

## 28.1 Batch forecasting

Suitable for:

* Daily demand planning.
* Weekly capacity planning.
* Monthly financial planning.
* Periodic risk reports.

The model runs on many series at scheduled intervals.

## 28.2 Real-time inference

Suitable for:

* Monitoring systems.
* Sensor anomaly prediction.
* Intraday demand.
* Traffic systems.
* Market surveillance.

The model updates forecasts as new observations arrive.

## 28.3 Interactive scenario engine

Suitable for planning teams.

Users can modify assumptions:

* Promotion schedule.
* Weather scenario.
* Capacity constraint.
* Price change.
* Event shock.
* Demand surge.
* System outage.

The model generates scenario-conditioned forecasts.

## 28.4 Embedded forecasting service

AionFM can be exposed as an API:

```text
POST /forecast
Input:
  historical_values
  covariates
  metadata
  horizon
  quantiles
  scenario_count

Output:
  point_forecast
  quantile_forecasts
  scenario_paths
  regime_probabilities
  explanation
```

---

# 29. Model Configuration

A practical AionFM family may include several model sizes.

| Model    | Intended Use                        | Characteristics                   |
| -------- | ----------------------------------- | --------------------------------- |
| AionFM-S | Edge or low-latency inference       | Smaller context, fewer parameters |
| AionFM-M | General enterprise forecasting      | Balanced accuracy and cost        |
| AionFM-L | High-accuracy batch forecasting     | Longer context, richer scenarios  |
| AionFM-X | Research and high-value deployments | Maximum context and capacity      |

Possible configuration dimensions:

* Patch length.
* Context length.
* Forecast horizon.
* Number of variables.
* Token vocabulary size.
* Number of regime-token levels.
* Transformer depth.
* Attention width.
* Memory compression ratio.
* Quantile levels.
* Scenario count.
* Adapter size.

---

# 30. Computational Considerations

## 30.1 Efficiency

Patch-based modeling reduces the effective sequence length. If the original sequence has length (T) and patch length (P), the model processes approximately:

[
K = \frac{T}{P}
]

tokens instead of (T) individual timestamps.

This improves efficiency for long histories.

## 30.2 Memory compression

Long-term memory states further reduce computational cost by summarizing older context.

## 30.3 Quantile and scenario cost

Generating many scenarios is more expensive than producing a single point forecast. Deployment systems may use different modes:

* Fast mode: median and selected quantiles.
* Standard mode: full quantile set.
* Scenario mode: multiple sampled paths.
* Stress mode: tail-conditioned paths.

## 30.4 Adapter efficiency

Domain adaptation through adapters reduces training cost because the core model can remain frozen.

---

# 31. Data Governance and Safety

AionFM may be used in sensitive operational contexts, so governance is essential.

## 31.1 Data quality controls

Before training or inference, systems should validate:

* Timestamp consistency.
* Duplicates.
* Missingness.
* Unit changes.
* Outlier patterns.
* Schema drift.
* Covariate availability.
* Entity identity consistency.

## 31.2 Forecast governance

Forecasts should include metadata:

* Model version.
* Training data period.
* Inference timestamp.
* Forecast horizon.
* Quantile levels.
* Scenario seed.
* Calibration status.
* Input data quality flags.

## 31.3 Human oversight

For high-impact applications, AionFM should support human review. The model should not silently automate critical decisions without domain-appropriate validation.

## 31.4 Uncertainty communication

Prediction intervals should be presented clearly. Users should understand that:

* A forecast is not a guarantee.
* Wider intervals indicate higher uncertainty.
* Tail scenarios are plausible but not necessarily likely.
* Calibration can degrade under distribution shift.

---

# 32. Failure Modes

AionFM is designed for robustness, but it may fail in several ways.

## 32.1 Unseen structural breaks

No historical model can fully predict unprecedented changes without relevant signals.

Examples:

* Sudden policy changes.
* Catastrophic system failures.
* Major geopolitical events.
* New market structures.
* Unexpected physical disruptions.

## 32.2 Poor covariate quality

Bad covariates can degrade forecasts.

Examples:

* Incorrect weather forecasts.
* Missing promotion calendars.
* Delayed operational updates.
* Mislabeled events.
* Broken metadata.

## 32.3 Overconfident uncertainty

If the model has not seen similar regimes, its intervals may be too narrow.

Calibration monitoring is required.

## 32.4 Token misclassification

The regime tokenizer may assign an incorrect behavioral state, especially in noisy or short histories.

## 32.5 Scenario implausibility

Generated paths may be statistically plausible but operationally impossible if domain constraints are not encoded.

For example:

* Negative inventory.
* Impossible capacity.
* Invalid physical readings.
* Non-trading periods.
* Regulatory constraints.

AionFM should support constraint-aware decoding for such domains.

---

# 33. Constraint-Aware Forecasting

Many domains impose hard constraints.

AionFM can include constraint modules for:

* Nonnegative values.
* Capacity limits.
* Conservation laws.
* Inventory balance.
* Calendar closures.
* Market hours.
* Physical sensor ranges.
* Monotonic cumulative quantities.
* Hierarchical reconciliation.

A constrained decoder can project raw forecasts into a valid space:

[
\hat{x}*{\text{valid}} = \Pi*{\mathcal{C}}(\hat{x})
]

where (\mathcal{C}) is the constraint set.

Examples:

* Sales cannot be negative.
* Occupancy cannot exceed physical capacity.
* Cumulative meter readings should not decrease.
* A store closed on a holiday should have zero or near-zero demand.
* A machine sensor may have known operating bounds.

---

# 34. Hierarchical Forecasting

Many real systems have hierarchy.

Examples:

```text
Company
  Region
    Store
      Product Category
        SKU
```

or:

```text
Grid
  Region
    Substation
      Feeder
        Meter
```

AionFM can support hierarchical reconciliation by forecasting at multiple levels and enforcing consistency.

If bottom-level forecasts are (\hat{y}_b), aggregate forecasts are:

[
\hat{y}_a = S \hat{y}_b
]

where (S) is a summing matrix.

The model can be trained with a reconciliation loss:

[
\mathcal{L}_{\text{hier}} =
|\hat{y}_a - S\hat{y}_b|
]

This encourages coherence across levels.

---

# 35. Multivariate and Cross-Series Modeling

AionFM can model relationships between variables and entities.

Examples:

* Temperature and energy load.
* Price and sales volume.
* Traffic and delivery time.
* Vibration and machine failure.
* Interest rates and economic indicators.
* Store-level sales across nearby locations.

Cross-series structure is represented through:

* Variable embeddings.
* Entity embeddings.
* Cross-variable attention.
* Correlation-regime tokens.
* Graph-aware adapters.
* Metadata-conditioned attention.

For very large panels, AionFM can use retrieval-based context selection. Instead of attending to all series, it retrieves relevant analogs based on metadata and learned temporal similarity.

---

# 36. Retrieval-Augmented Temporal Forecasting

AionFM can be extended with a temporal retrieval module.

Given a current context, the system retrieves historical windows with similar behavior:

[
R(x_{1:t}) = {w_1, w_2, ..., w_n}
]

These retrieved windows may come from:

* The same entity.
* Similar entities.
* Similar historical regimes.
* Similar seasonal periods.
* Similar shock patterns.

The model then conditions on retrieved analogs.

This can improve:

* Rare-event forecasting.
* Cold starts.
* Regime transition prediction.
* Scenario generation.
* Explainability.

Example explanation:

```text
The current sequence resembles prior post-promotion decay patterns from similar
products. In those analogs, median demand normalized after 9 to 14 days, while
upper-tail demand persisted longer in high-traffic regions.
```

---

# 37. Calibration and Monitoring

AionFM should be monitored after deployment.

## 37.1 Accuracy monitoring

Track:

* Forecast error by horizon.
* Forecast error by entity.
* Forecast error by segment.
* Error during special events.
* Error under missing-data conditions.

## 37.2 Calibration monitoring

Track:

* 50% interval coverage.
* 80% interval coverage.
* 90% interval coverage.
* 95% interval coverage.
* Quantile crossing.
* Interval width drift.

## 37.3 Regime monitoring

Track:

* Regime-token distribution.
* New or rare regime frequency.
* Change-point rates.
* Token entropy.
* Scenario divergence.

## 37.4 Drift monitoring

Track:

* Input distribution shift.
* Covariate shift.
* Residual distribution shift.
* Scale drift.
* Seasonal drift.
* Entity behavior drift.

When drift exceeds thresholds, the system can trigger:

* Recalibration.
* Adapter retraining.
* Full fine-tuning.
* Human review.
* Scenario stress testing.

---

# 38. Security and Privacy Considerations

AionFM may be trained on sensitive operational data. The training and deployment pipeline should include:

* Data minimization.
* Access controls.
* Encryption at rest and in transit.
* Audit logging.
* Entity anonymization where appropriate.
* Differential privacy for sensitive domains where needed.
* Separation of customer-specific adapters.
* Evaluation for data memorization risk.

Forecast explanations should avoid exposing sensitive training examples unless explicitly permitted.

---

# 39. Implementation Blueprint

AionFM can be implemented in modular layers.

## 39.1 Data layer

Responsibilities:

* Load time series.
* Validate schema.
* Align timestamps.
* Handle missingness.
* Normalize values.
* Generate patches.
* Attach metadata.
* Prepare covariates.

## 39.2 Tokenization layer

Responsibilities:

* Compute residual descriptors.
* Quantize behavioral states.
* Maintain regime vocabulary.
* Update hierarchical token codebooks.
* Provide token interpretability maps.

## 39.3 Model layer

Responsibilities:

* Encode value patches.
* Encode regime tokens.
* Fuse streams.
* Apply causal attention.
* Use memory states.
* Decode forecasts.
* Generate scenarios.

## 39.4 Training layer

Responsibilities:

* Mix datasets.
* Sample horizons.
* Apply masking.
* Generate synthetic data.
* Optimize composite loss.
* Evaluate calibration.
* Save checkpoints.

## 39.5 Adaptation layer

Responsibilities:

* Train adapters.
* Fine-tune domain heads.
* Calibrate quantiles.
* Apply constraints.
* Register domain profiles.

## 39.6 Serving layer

Responsibilities:

* Forecast API.
* Batch inference.
* Scenario generation.
* Monitoring.
* Logging.
* Version control.
* Explainability output.

---

# 40. Example Forecast Output Schema

AionFM should return structured outputs.

```json
{
  "model": "AionFM",
  "model_version": "0.1",
  "forecast_horizon": 30,
  "frequency": "D",
  "target": "demand",
  "point_forecast": [124.1, 127.5, 129.0],
  "quantiles": {
    "0.10": [101.2, 103.7, 105.1],
    "0.50": [123.8, 126.9, 128.7],
    "0.90": [149.4, 154.2, 159.8]
  },
  "prediction_intervals": {
    "80": {
      "lower": [101.2, 103.7, 105.1],
      "upper": [149.4, 154.2, 159.8]
    }
  },
  "scenario_paths": [
    [122.0, 126.1, 130.8],
    [131.5, 139.2, 144.0],
    [115.4, 112.7, 118.6]
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
  }
}
```

---

# 41. Research Roadmap

AionFM opens several research directions.

## 41.1 Better regime vocabularies

The regime-token vocabulary can be improved through:

* Hierarchical vector quantization.
* Human-interpretable token labeling.
* Domain-specific token adapters.
* Contrastive token refinement.
* Causal regime discovery.

## 41.2 Causal forecasting

AionFM can be extended to separate correlation from intervention.

Example questions:

* What happens if price changes?
* What happens if capacity is reduced?
* What happens if a promotion is delayed?
* What happens if a shock repeats?

This requires intervention-aware training and causal covariate design.

## 41.3 Agentic planning integration

AionFM can become part of planning systems that:

* Generate forecasts.
* Simulate actions.
* Compare scenarios.
* Optimize decisions.
* Monitor outcomes.
* Update policies.

## 41.4 Physics-aware constraints

For industrial, climate, and engineering systems, AionFM can incorporate physical equations or conservation constraints.

## 41.5 Multimodal temporal modeling

Future versions may integrate:

* Textual event descriptions.
* News.
* Maintenance logs.
* Images.
* Geospatial data.
* Graph data.
* Audio or vibration spectrograms.

The time-series backbone would remain central, while multimodal encoders provide additional context.

---

# 42. Competitive Differentiation

AionFM’s differentiation comes from the combination of:

1. Continuous value precision.
2. Discrete behavioral abstraction.
3. Residual-based tokenization.
4. Probabilistic forecasting.
5. Scenario generation.
6. Regime interpretability.
7. Long-context memory.
8. Domain adapters.
9. Constraint-aware decoding.
10. Cross-domain temporal transfer.

The most important distinction is that AionFM does not force a choice between numerical regression and symbolic sequence modeling. It uses both, with each doing what it is best suited to do.

The continuous pathway preserves exactness.
The discrete pathway captures reusable temporal structure.
The probabilistic decoder turns both into actionable forecasts.

---

# 43. Practical Deployment Example

Consider a retail planning team forecasting demand for thousands of products across hundreds of stores.

A conventional forecasting system may require:

* SKU-level feature engineering.
* Store-specific models.
* Promotion-specific rules.
* Manual holiday adjustments.
* Separate uncertainty estimation.
* Separate anomaly detection.
* Separate cold-start strategy.

AionFM provides a unified workflow:

1. Historical sales, inventory, price, and promotion data are ingested.
2. Store and product metadata are embedded.
3. Calendar features are attached.
4. AionFM generates median and quantile forecasts.
5. Regime tokens identify whether each SKU is in normal demand, promotion uplift, post-promotion decay, seasonal peak, intermittent demand, or anomalous behavior.
6. Scenario paths estimate possible demand outcomes.
7. Inventory buffers are computed from quantiles rather than fixed safety-stock heuristics.
8. Monitoring detects when forecast calibration deteriorates.

This allows the planning system to move from single-number forecasting to risk-aware temporal decision support.

---

# 44. Limitations

AionFM is not a universal oracle. It cannot eliminate uncertainty or guarantee correct forecasts under unprecedented conditions.

Key limitations include:

* Forecast quality depends on data quality.
* Extreme regime shifts remain difficult.
* Long-horizon forecasts may become diffuse.
* Poor covariates can mislead the model.
* Scenario generation requires careful validation.
* Interpretability tokens are learned approximations, not ground-truth causal states.
* Domain constraints must be explicitly encoded for high-stakes use.
* Calibration must be monitored after deployment.

AionFM should therefore be deployed as part of a forecasting system with validation, monitoring, governance, and human oversight.

---

# 45. Conclusion

AionFM is a proposed time-series foundation model built around a simple but powerful idea: effective forecasting requires both numerical precision and behavioral abstraction.

Its dual-stream architecture combines continuous value patches with discrete residual-regime tokens. This allows the model to learn not only what values are likely to come next, but what kind of temporal behavior is unfolding. By forecasting values, regimes, quantiles, and scenarios together, AionFM supports a broader set of real-world decisions than point forecasting alone.

The model is designed for heterogeneous, nonstationary, and high-uncertainty environments. It can be applied across finance, energy, retail, logistics, healthcare operations, climate, mobility, and industrial telemetry. Its residual temporal language makes temporal patterns transferable across domains, while adapters and constraint modules allow domain-specific refinement.

AionFM’s central promise is to turn forecasting from a narrow prediction task into a general temporal reasoning layer: one that can understand context, represent uncertainty, generate scenarios, detect regimes, and support robust decision-making under changing conditions.
