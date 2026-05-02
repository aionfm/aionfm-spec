# AionFM

### A Hybrid Continuous-Discrete Temporal Foundation Model for Probabilistic, Regime-Aware Time-Series Forecasting

---

## Overview

**AionFM** is a proposed time-series foundation model built around a central thesis: effective forecasting requires both **numerical precision** and **behavioral abstraction**. Rather than choosing between continuous value representations (which preserve accuracy but lack structural reasoning) and discrete tokenization (which enables transfer but loses precision), AionFM operates both simultaneously through a **dual-stream decoder-only architecture**.

The model processes temporal data through two synchronized representational streams:

- **Continuous Value Patch Stream** — segments normalized observations into subseries-level patches projected into dense embeddings, preserving full numerical resolution throughout the forecasting pipeline.
- **Discrete Regime Token Stream** — computes behavioral feature vectors from local temporal summaries and quantizes them into a learned vocabulary of residual regime tokens encoding trend, volatility, seasonality, discontinuities, missingness, and cross-variable structural states.

AionFM simultaneously serves as a point forecaster, probabilistic forecaster, scenario generator, regime detector, missing-data imputer, synthetic trajectory generator, and domain-adaptable forecasting engine — all within a single set of shared parameters.

---

## Key Contribution: The Residual Temporal Language

The defining innovation of AionFM is the **Residual Temporal Language** — a domain-independent behavioral vocabulary learned from normalized residuals rather than raw observations.

Instead of tokenizing raw values (which are inherently domain-specific), AionFM:

1. Estimates a **local baseline** `b_t = f_baseline(x_{1:t})` capturing level, trend, and dominant seasonality
2. Computes a **normalized residual** `e_t = (x_t - b_t) / (s_t + ε)` that is scale-free and domain-independent
3. Summarizes residuals into **behavioral descriptors** (direction, curvature, volatility, shock magnitude, recovery rate, seasonality phase, cross-variable divergence, etc.)
4. Maps descriptors to **discrete regime tokens** via learned vector quantization

A regime token does not encode *"the value is 17.2"* — it encodes behavioral states such as *"stable upward drift,"* *"high-volatility expansion,"* *"negative shock with partial recovery,"* or *"mean-reverting residual compression."*

This abstraction enables cross-domain transfer because concepts like "volatility expansion" or "shock recovery" recur across finance, energy, retail, traffic, climate, and operations data — even when raw magnitudes differ by orders of magnitude.

### Behavioral Pattern Vocabulary

| Pattern | Token Semantics | Description |
|---------|----------------|-------------|
| **Drift** | Stable directional movement | Sustained positive/negative slope with low curvature |
| **Reversal** | Direction change | Sign change in local slope with high curvature |
| **Shock** | Abrupt deviation | Large standardized residual exceeding local volatility band |
| **Recovery** | Return toward baseline | Post-shock trajectory with exponential/linear decay |
| **Compression** | Volatility contraction | Declining local variance and narrowing range |
| **Expansion** | Volatility increase | Rising local variance and widening range |
| **Saturation** | Flattening near bound | Declining slope near a local max/min |
| **Seasonality Peak/Trough** | Cyclic extremum | Residual phase aligned with seasonal cycle at extremum |
| **Regime Break** | Structural discontinuity | Persistent change in statistical properties |
| **Coupling** | Cross-variable alignment | Increasing correlation among previously independent variables |
| **Decoupling** | Cross-variable divergence | Decreasing correlation among previously co-moving variables |
| **Missing Burst** | Observation gap cluster | Concentrated missing values (sensor failure, outage) |
| **Anomalous Spike** | Isolated extreme value | Single-observation deviation without sustained regime change |

---

## Architecture

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                     INPUT TIME SERIES DATA                      │
  │  (targets, covariates, metadata, calendar, missingness masks)   │
  └────────────────────────────┬─────────────────────────────────────┘
                               │
                               ▼
  ┌──────────────────────────────────────────────────────────────────┐
  │            [1] INPUT NORMALIZATION & PREPARATION                │
  │  Reversible scaling, frequency detection, calendar encoding,    │
  │  missingness masking, entity/domain descriptors                 │
  └────────────────────────────┬─────────────────────────────────────┘
                               │
                  ┌────────────┴────────────┐
                  │                         │
                  ▼                         ▼
  ┌────────────────────────┐  ┌────────────────────────────┐
  │ [2] CONTINUOUS VALUE   │  │ [3] DISCRETE REGIME        │
  │ PATCH ENCODER          │  │ TOKENIZER                  │
  │ Patch segmentation,    │  │ Behavioral features,       │
  │ linear projection,     │  │ vector quantization,       │
  │ numerical embedding    │  │ macro/meso/micro tokens    │
  └──────────┬─────────────┘  └──────────────┬─────────────┘
             │                               │
             └───────────┬───────────────────┘
                         │ Gated Fusion
                         ▼
  ┌──────────────────────────────────────────────────────────────────┐
  │         [4] DUAL-STREAM CAUSAL TEMPORAL BACKBONE                │
  │  Decoder-only Transformer with causal masking                   │
  │  Patch attention + cross-variable attention + memory attention  │
  │  Temporal position encoding + frequency-aware positions         │
  │  Compressed long-context memory                                 │
  └────────────────────────────┬─────────────────────────────────────┘
                               │
                               ▼
  ┌──────────────────────────────────────────────────────────────────┐
  │      [5] PROBABILISTIC FORECAST & SCENARIO DECODER              │
  │  Point forecast │ Quantile │ Distribution │ Regime forecast     │
  │  Scenario generation │ Imputation                               │
  │  Inverse normalization & constraint projection                  │
  └──────────────────────────────────────────────────────────────────┘
```

### Subsystem Summary

| # | Subsystem | Role |
|---|-----------|------|
| 1 | **Input Normalization** | Reversible scaling (z-score, MAD, Box-Cox, return, seasonal residual), frequency detection, calendar encoding, missingness masking |
| 2 | **Continuous Value Patch Encoder** | Segments sequences into patches of length P with stride S; projects via `h^v_k = W_v · p_k + b_v`; preserves full numerical resolution |
| 3 | **Discrete Regime Tokenizer** | Computes behavioral feature vectors; maps to discrete codes via vector quantization; hierarchical macro/meso/micro token structure |
| 4 | **Dual-Stream Causal Backbone** | Decoder-only Transformer with gated fusion (`α_k, β_k, γ_k`); supports patch, cross-variable, regime-token, memory, and covariate-aware attention |
| 5 | **Probabilistic Forecast & Scenario Decoder** | Six output heads: point, quantile, distribution, regime, scenario, imputation |

### Hierarchical Regime Tokens

| Level | Scope | Example States | Change Frequency |
|-------|-------|---------------|-----------------|
| **Macro** | Multi-patch window | "upward trend," "stationary oscillation," "structural break" | Infrequent |
| **Meso** | Single/few-patch | "volatility expansion," "mean-reverting residual," "post-shock recovery" | Moderate |
| **Micro** | Within single patch | "positive residual spike," "residual compression," "noise-dominated" | Frequent |

### Compressed Temporal Memory

| Tier | Scope | Resolution | Purpose |
|------|-------|-----------|---------|
| **Active Window** | Recent W patches | Full attention | Maximum-resolution processing |
| **Recent Memory** | Medium-term past | Learned compression | Key statistical properties preserved |
| **Long-Term Memory** | Distant past | High compression | Historical baselines, prior regimes, seasonal patterns |

---

## Training Framework

### 7-Component Composite Objective

```
L_total = λ₁·L_patch + λ₂·L_mask + λ₃·L_regime + λ₄·L_quantile + λ₅·L_rollout + λ₆·L_contrastive + λ₇·L_calibration
```

| # | Loss Component | Purpose | Key Reference |
|---|---------------|---------|---------------|
| 1 | **Next-Patch Prediction** | Autoregressive temporal prediction; grounds model in numerical accuracy | — |
| 2 | **Masked Temporal Reconstruction** | Robustness, imputation capability, noise tolerance (15–40% masking) | — |
| 3 | **Regime-Token Likelihood** | Trains residual temporal language via cross-entropy over regime vocabulary | — |
| 4 | **Quantile Loss** | Calibrated uncertainty at τ ∈ {0.01, 0.05, 0.10, 0.25, 0.50, 0.75, 0.90, 0.95, 0.99} | Koenker & Bassett, 1978 |
| 5 | **Rollout Consistency** | Multi-step coherence; reduces compounding error in long-horizon forecasts | — |
| 6 | **Cross-Scale Contrastive Alignment** | Connects local movements to broader regime context; separates noise from structure | van den Oord et al., 2018 |
| 7 | **Calibration Loss** | Ensures empirical interval coverage matches nominal levels | Gneiting & Raftery, 2007 |

### Pretraining Data

**Real data** spanning: retail sales, energy demand, web traffic, industrial sensors, weather/climate, transportation, healthcare operations, financial markets, economic indicators, manufacturing telemetry, and supply-chain flows.

**Synthetic data** from a configurable trajectory generator:

```
x_t = T_t + S_t + C_t + E_t + R_t + ε_t
      ───   ───   ───   ───   ───   ───
     trend  seas  cycle event regime noise
```

Simulates: piecewise/exponential/logistic trends, multiple seasonalities, random shocks, step changes, mean reversion, heteroskedasticity, autoregressive noise, sparse demand, irregular sampling, sensor drift, and correlated multivariate systems — with latent regime labels retained for direct supervision.

---

## Probabilistic Forecasting & Scenario Generation

### Six Output Heads

| Head | Output | Use Case |
|------|--------|----------|
| **Point Forecast** | Deterministic mean/median | Planning, benchmarking |
| **Quantile Forecast** | Calibrated quantiles at multiple levels | Prediction intervals, safety stock |
| **Distribution** | Parametric distribution parameters (Gaussian, Student-t, negative binomial, mixture, zero-inflated) | Likelihood evaluation, sampling |
| **Regime Forecast** | Future behavioral token sequence | Interpretable trajectory characterization |
| **Scenario Generation** | Multiple coherent future trajectories | Risk simulation, stress testing, capacity planning |
| **Imputation** | Reconstructed missing historical values | Data quality, gap-filling |

### Two-Stage Scenario Generation

1. **Stage 1 — Regime Path Sampling:** Sample plausible future regime token sequences from the conditional distribution (e.g., "continued growth," "shock at step 3 with recovery," "volatility expansion with reversal")
2. **Stage 2 — Value Decoding:** Generate numerical forecasts conditioned on each sampled regime path

Scenario types: `conservative` · `median` · `optimistic` · `tail-risk` · `stress` · `regime-specific`

### Example Output Schema

```json
{
  "model_version": "AionFM-M v1.0",
  "forecast_origin": "2026-05-01T00:00:00Z",
  "forecast_horizon": 14,
  "frequency": "daily",
  "target": "store_42_sku_1087_units",
  "point_forecast": [142, 138, 155, 161, 170, 168, 152, 145, 140, 158, 164, 173, 171, 155],
  "quantiles": {
    "q05": [112, 107, 119, 124, 131, 128, 117, 111, 107, 121, 126, 133, 131, 119],
    "q50": [142, 138, 155, 161, 170, 168, 152, 145, 140, 158, 164, 173, 171, 155],
    "q95": [176, 172, 194, 202, 213, 210, 190, 181, 175, 198, 206, 217, 214, 194]
  },
  "scenario_paths": {
    "continued_growth": [145, 142, 160, 168, 178, 176, 160, 153, 148, 166, 172, 182, 180, 164],
    "shock_recovery":   [142, 138, 118, 105, 120, 135, 148, 145, 140, 158, 164, 173, 171, 155],
    "seasonal_decline":  [142, 135, 128, 122, 118, 115, 112, 110, 108, 112, 118, 125, 130, 135]
  },
  "regime_probabilities": {
    "stable_growth": 0.42,
    "seasonal_peak": 0.27,
    "shock_recovery": 0.18,
    "reversal": 0.13
  },
  "explanation": {
    "current_regime": "stable_upward_drift with seasonal_approach",
    "uncertainty_driver": "seasonal_phase_transition at horizon 7-9",
    "change_point_probability": 0.23
  }
}
```

---

## Model Family

| Config | AionFM-S (Edge) | AionFM-M (Enterprise) | AionFM-L (High-Accuracy) | AionFM-X (Research) |
|--------|:---:|:---:|:---:|:---:|
| **Patch length P** | 16 | 32 | 64 | 64 |
| **Max context (patches)** | 128 | 512 | 1,024 | 2,048 |
| **Max forecast horizon** | 96 | 384 | 720 | 1,440 |
| **Max variables** | 16 | 128 | 512 | 2,048 |
| **Regime token vocab** | 256 | 1,024 | 4,096 | 8,192 |
| **Regime token levels** | 2 (macro/meso) | 3 (macro/meso/micro) | 3 | 3 |
| **Transformer depth** | 6 | 12 | 24 | 48 |
| **Attention width (d_model)** | 256 | 512 | 768 | 1,024 |
| **Memory compression** | 4:1 | 8:1 | 16:1 | 32:1 |
| **Quantile levels** | 5 | 9 | 9 | 19 |
| **Scenario count** | 10 | 50 | 100 | 500 |
| **Adapter size** | 32 | 64 | 128 | 256 |
| **Target use case** | Edge, mobile, low-latency | General enterprise | High-accuracy batch | Research, high-value |

---

## Domain Adaptation

AionFM supports four adaptation modes:

| Mode | Data Required | What Updates | When to Use |
|------|:---:|:---:|-------------|
| **Zero-shot** | None | Nothing | Cold starts, rapid deployment, limited data |
| **Few-shot calibration** | 50–500 obs | Scale, quantile coverage, seasonal phase, covariate effects | Limited domain data, fast adaptation |
| **Adapter tuning** | Domain dataset | Lightweight adapter modules (core frozen) | Domain-specific frequency, covariates, distributions |
| **Full fine-tuning** | Large domain dataset | All parameters | High-value, specialized applications |

### Covariate Support

- **Known future:** calendar, promotions, maintenance windows, holidays, price schedules, weather forecasts
- **Historical:** actual weather, prices, traffic, inventory, sensor readings, economic indicators
- **Static metadata:** store type, region, product category, sensor location, machine type
- **Latent metadata:** inferred entity descriptors from historical behavior

---

## Application Domains

| Domain | Key Inputs | Key Outputs |
|--------|-----------|-------------|
| **Energy** | Load history, temperature, calendar, weather forecasts, regional metadata | Peak demand intervals, extreme-load scenarios, regime detection, capacity-risk estimates |
| **Retail** | SKU sales, promotions, price changes, holidays, inventory constraints | Demand forecasts, safety stock recommendations, promotion uplift scenarios, hierarchical reconciliation |
| **Industrial** | Multi-sensor streams (temp, pressure, vibration), machine metadata | Predictive maintenance, drift detection, failure-regime probabilities, remaining useful life |
| **Healthcare** | ED arrivals, bed occupancy, procedure volume, staffing, calendar | Demand intervals, surge scenarios, capacity-risk flags, calendar-adjusted projections |
| **Logistics** | Shipment volume, congestion, delivery times, fleet utilization | Volume forecasts, delay-risk scenarios, disruption-sensitive capacity plans |
| **Financial** | Returns, volume, volatility, liquidity, cross-asset dependencies | Risk scenarios, volatility forecasts, regime probabilities, tail-risk estimates |

---

## Advanced Capabilities

- **Constraint-Aware Forecasting** — projects raw forecasts into valid space respecting nonnegativity, capacity limits, conservation laws, market hours, monotonic cumulative quantities
- **Hierarchical Forecasting** — forecasts at multiple aggregation levels (Company → Region → Store → Category → SKU) with reconciliation consistency (Hyndman et al., 2011)
- **Retrieval-Augmented Temporal Forecasting** — retrieves historical windows with similar behavioral signatures to condition forecasts; improves rare-event forecasting, cold starts, and explainability
- **Multivariate & Cross-Series Modeling** — cross-variable attention, correlation-regime tokens, graph-aware adapters, metadata-conditioned attention; retrieval-based context selection for very large panels

---

## Evaluation Framework

### Metrics

| Category | Metrics |
|----------|---------|
| **Point accuracy** | MAE, RMSE, MAPE, sMAPE, MASE, WAPE |
| **Probabilistic** | Pinball loss, CRPS, prediction interval coverage & width, calibration error, weighted quantile loss |
| **Scenario quality** | Path coherence, distributional similarity, tail-event coverage, regime transition realism, multivariate dependency preservation |
| **Robustness** | Missing data degradation, corrupted observations, outliers, regime shifts, covariate dropout, scale shifts |
| **Transfer** | Zero-shot accuracy, few-shot adaptation speed, domain/frequency transfer, new-entity cold start |
| **Interpretability** | Regime-token stability, change-point alignment, regime explanation consistency, component attribution fidelity |

### Benchmarking Protocol

Five evaluation settings: **zero-shot** (unseen datasets) · **fine-tuned** (domain-specific) · **few-shot** (cold-start) · **long-horizon** (rollout stability) · **stress** (shocks, missing data, distribution shifts, rare regimes)

---

## Deployment Modes

| Mode | Latency | Use Case |
|------|---------|----------|
| **Batch forecasting** | Minutes–hours | Daily/weekly/monthly planning, demand forecasting, capacity provisioning |
| **Real-time inference** | Sub-second (S) to sub-5s (M) | Monitoring, sensor anomaly detection, intraday demand, traffic |
| **Interactive scenario engine** | Seconds | Planning teams modify assumptions and receive updated forecasts in real time |
| **Embedded API** | Per-request | Integration into enterprise systems, dashboards, optimization engines |

---

## Limitations

- Forecast quality depends on data quality; garbage in, garbage out
- Extreme regime shifts (geopolitical events, unprecedented policy changes) remain difficult for any historical model
- Long-horizon forecasts may become diffuse as uncertainty compounds
- Interpretability tokens are learned approximations, not ground-truth causal states
- Scenario generation requires careful validation — statistical plausibility ≠ operational plausibility
- Domain constraints must be explicitly encoded for high-stakes applications
- Calibration must be monitored post-deployment for distributional drift
- **This is a conceptual specification — empirical validation across diverse benchmarks is needed**

---

## Future Research Directions

1. **Better Regime Vocabularies** — hierarchical vector quantization, human-interpretable token labeling, domain-specific token adapters, causal regime discovery
2. **Causal Forecasting** — separating correlation from intervention; what-if questions (price changes, capacity reductions, promotion delays)
3. **Agentic Planning Integration** — AionFM as a component in planning systems that generate forecasts, simulate actions, compare scenarios, and optimize decisions
4. **Physics-Aware Constraints** — incorporating physical equations and conservation laws for industrial, climate, and engineering systems
5. **Multimodal Temporal Modeling** — integrating textual event descriptions, news, maintenance logs, images, geospatial data, and graph structures alongside the time-series backbone

---

## References

[1] Adams, R.P. and MacKay, D.J.C. (2007). *Bayesian Online Changepoint Detection.* arXiv preprint arXiv:0710.3742.

[2] Box, G.E.P., Jenkins, G.M., Reinsel, G.C., and Ljung, G.M. (2015). *Time Series Analysis: Forecasting and Control,* 5th edition. John Wiley and Sons.

[3] Durbin, J. and Koopman, S.J. (2012). *Time Series Analysis by State Space Methods,* 2nd edition. Oxford University Press.

[4] Gneiting, T. and Raftery, A.E. (2007). Strictly Proper Scoring Rules, Prediction, and Estimation. *Journal of the American Statistical Association,* 102(477), 359–378.

[5] Hamilton, J.D. (1989). A New Approach to the Economic Analysis of Nonstationary Time Series and the Business Cycle. *Econometrica,* 57(2), 357–384.

[6] Hochreiter, S. and Schmidhuber, J. (1997). Long Short-Term Memory. *Neural Computation,* 9(8), 1735–1780.

[7] Hyndman, R.J. and Athanasopoulos, G. (2021). *Forecasting: Principles and Practice,* 3rd edition. OTexts, Melbourne, Australia.

[8] Hyndman, R.J., Ahmed, R.A., Athanasopoulos, G., and Shang, H.L. (2011). Optimal Combination Forecasts for Hierarchical Time Series. *Computational Statistics and Data Analysis,* 55(9), 2579–2589.

[9] Koenker, R. and Bassett, G. (1978). Regression Quantiles. *Econometrica,* 46(1), 33–50.

[10] van den Oord, A., Vinyals, O., and Kavukcuoglu, K. (2017). Neural Discrete Representation Learning. *Advances in Neural Information Processing Systems,* 30, 6306–6315.

[11] van den Oord, A., Li, Y., and Vinyals, O. (2018). Representation Learning with Contrastive Predictive Coding. arXiv preprint arXiv:1807.03748.

[12] Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A.N., Kaiser, L., and Polosukhin, I. (2017). Attention Is All You Need. *Advances in Neural Information Processing Systems,* 30, 5998–6008.

---

## License

This is an open technical specification released for community development. No empirical results are reported. The architecture and methods described are proposed designs intended to guide implementation and invite community contribution.

---

