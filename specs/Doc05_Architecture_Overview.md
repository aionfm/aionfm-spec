# Architecture Overview

## Purpose & Content

This document presents a high‑level view of the AionFM architecture.  It identifies the major subsystems, describes how data flows through the system and outlines the relationships between components.  It serves as a map for developers and stakeholders to understand the overall structure before diving into individual modules.

## Audience

System architects and new developers onboarding onto the project.

## Key Sections

- Component diagram
- Data and control flow
- Layer responsibilities (data, model, serving)
- Interaction with external systems

## Dependencies

Doc03 (Design Principles), Doc04 (Use Cases & Requirements)

## Document Body

### Major Components

The AionFM architecture consists of five major subsystems:

1. **Input Normalization and Temporal Preparation:** Handles data ingestion, normalization, metadata alignment and missingness encoding.
2. **Continuous Value Patch Encoder:** Converts normalized time‑series windows into dense numerical embeddings that preserve local variation.
3. **Discrete Regime Tokenizer:** Summarizes residual behaviour into hierarchical tokens describing macro, meso and micro regimes.
4. **Dual‑Stream Causal Backbone:** A decoder‑only Transformer that attends over value and regime embeddings, integrating time and covariates while respecting causality.
5. **Probabilistic Forecast and Scenario Decoder:** Produces baseline, seasonal, event and residual component forecasts along with quantiles, distributions, regime probabilities and scenario samples.

### Data Flow

Raw sequences are first normalized and segmented into patches.  Value patches and behavioural tokens are fused with time and metadata embeddings and processed by the causal backbone.  The backbone outputs hidden states that feed specialized heads to produce forecasts, quantiles, regimes and scenarios.

### Interaction with External Systems

AionFM can be embedded into various platforms via REST/GraphQL APIs or a Rust SDK.  It can run in batch mode for offline planning or real‑time mode for streaming predictions.  The platform repositories (`aionfm-ui`, `aionfm-api`, `aionfm-sdk`) communicate with the core model through well‑defined interfaces described in later documents.