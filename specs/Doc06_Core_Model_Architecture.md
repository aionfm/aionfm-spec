# Core Model Architecture

## Purpose & Content

This document details the dual‑stream architecture at the heart of AionFM.  It explains how continuous value patches and discrete regime tokens are generated and processed to produce forecasts.  It also introduces the simplified flow that guides implementation.

## Audience

Core developers implementing the model in Rust.

## Key Sections

- Continuous value representation
- Regime tokenization
- Simplified processing pipeline
- Data structures and interfaces

## Dependencies

Doc05 (Architecture Overview)

## Document Body

### Continuous Value Representation

Time series are divided into patches of length `P` with stride `S`.  Each patch consists of normalized values of one or more variables, optional covariates, missingness masks and time features.  The patch is projected into an embedding via a learned linear transformation.  This representation retains fine numerical detail necessary for high‑fidelity forecasts.

### Regime Tokenization

For each patch, AionFM computes local residuals relative to a learned baseline and local scale.  Behaviour descriptors—such as direction, curvature, volatility, range and seasonality phase—are extracted.  A learned quantizer maps these descriptors into hierarchical tokens: macro (broad regime), meso (local pattern) and micro (residual detail).

### Processing Pipeline

The model follows this simplified flow:

```
Raw time series
     ↓
Normalization, metadata alignment, missingness encoding
     ↓
Continuous value patches  +  discrete regime tokens
     ↓
Dual‑stream causal Transformer
     ↓
Baseline forecast + residual forecast + regime forecast
     ↓
Point predictions, quantiles, intervals and sampled scenarios
```

Continuous value patches and regime tokens flow into the causal Transformer, which applies gated fusion and attention.  The output feeds multiple heads to generate forecasts and uncertainty estimates.