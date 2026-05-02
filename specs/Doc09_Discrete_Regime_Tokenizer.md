# Discrete Regime Tokenizer

## Purpose & Content

This document explains how AionFM extracts local behavioural descriptors from residual values and converts them into discrete regime tokens.  It introduces hierarchical token vocabularies that capture macro trends, meso patterns and micro anomalies.

## Audience

Modellers and developers working on the residual abstraction and quantisation components.

## Key Sections

- Computing residuals and local scale
- Extracting behavioural descriptors
- Quantization and vocabulary maintenance
- Hierarchical token structure

## Dependencies

Doc07 (Input Representation & Normalization), Doc08 (Continuous Value Patch Encoder)

## Document Body

### Residuals and Local Scale

For a patch `p_k`, a baseline `b_t` is estimated from past values using a moving average or a learned baseline function.  A local scale `s_t` is computed (e.g., using standard deviation or median absolute deviation).  Residuals are normalized as `e_t = (x_t - b_t)/(s_t + ε)` where `ε` is a small constant.

### Behavioural Descriptors

Features are extracted from residuals and include direction, curvature, volatility, range expansion, seasonality phase, local acceleration, shock magnitude, recovery rate, missingness structure, cross‑variable divergence, correlation change and distributional asymmetry.

### Quantization and Vocabulary

The descriptor vector `g_k` is passed through a vector quantization layer `Q` that assigns codes according to a learned codebook.  The vocabulary is organised hierarchically:

- **Macro tokens:** Broad regimes such as upward trend, seasonal decline, volatility expansion or saturation.
- **Meso tokens:** Local patterns such as stable low variance, volatility spikes or sudden shocks.
- **Micro tokens:** Fine residual detail such as small positive/negative deviations or irregular oscillations.

The quantizer is trained jointly with the model to capture salient behaviours.  Vocabularies are updated periodically or through scheduled re‑learning.