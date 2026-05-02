# Dual‑Stream Causal Backbone

## Purpose & Content

This document defines the temporal backbone of AionFM.  It describes how value, regime and covariate embeddings are fused and processed by a causal Transformer.  It explains attention mechanisms and gating strategies to handle diverse inputs and preserve causality.

## Audience

Core developers implementing the neural architecture.

## Key Sections

- Gated fusion of embeddings
- Causal attention and masking
- Attention components (local, cross‑variable, regime, memory)
- Position and covariate encoding integration

## Dependencies

Doc08 (Continuous Value Patch Encoder), Doc09 (Discrete Regime Tokenizer)

## Document Body

### Gated Fusion

At each patch index `k`, the model receives embeddings from multiple streams: `h^v_k` for values, `h^r_k` for regimes, `h^c_k` for covariates, `h^t_k` for temporal encoding and `h^m_k` for variable/entity metadata.  Learned gates `α_k`, `β_k` and `γ_k` control how much each stream contributes to the fused representation:

```
h_k = α_k h^v_k + β_k h^r_k + γ_k h^c_k + h^t_k + h^m_k.
```

### Causal Attention

The backbone uses a Transformer with causal masking so that predictions at time `k` cannot depend on future embeddings.  Attention layers include:

- **Local patch attention:** Focuses on nearby patches to capture short‑term dependencies.
- **Cross‑variable attention:** Captures relationships across multiple variables or entities.
- **Regime attention:** Allows the model to condition on behavioural state transitions.
- **Memory attention:** Retrieves compressed summaries of older history for long‑context effects.
- **Covariate attention:** Weights covariate embeddings based on relevance.

Position encodings (absolute, relative, calendar, frequency, horizon) and covariate embeddings are added to maintain temporal and contextual information.