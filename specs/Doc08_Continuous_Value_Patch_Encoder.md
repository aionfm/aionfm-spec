# Continuous Value Patch Encoder

## Purpose & Content

This document describes how AionFM converts normalized time‑series windows into dense numerical embeddings.  It explains patch segmentation, embedding matrices and handling of multivariate data and missing values.

## Audience

Modellers and implementers working on the encoder layer.

## Key Sections

- Patch formation (length and stride)
- Input channels (targets, covariates, masks, time features)
- Embedding transformation
- Handling of missingness and variable identities

## Dependencies

Doc07 (Input Representation & Normalization)

## Document Body

### Patch Formation

A sequence of length `T` is divided into overlapping or non‑overlapping patches of length `P` using a stride `S`.  Each patch `p_k` contains normalized values of all variables in the window `[kS, kS+P-1]`.  Overlap can improve smoothness but increases computational cost.

### Input Channels

Each patch includes multiple channels:

- **Target values:** Normalized observed values of the primary series.
- **Covariates:** Known future covariates and historical covariates aligned with each time step.
- **Missingness masks:** Binary indicators showing whether each observation is present (1) or missing (0).
- **Time features:** Encodings of timestamp (e.g., hour, day, month, season), frequency and horizon position.
- **Static metadata projections:** Embeddings of entity identifiers and domain descriptors broadcast across the patch.

### Embedding Transformation

A learned linear projection `W_v` is applied to each flattened patch to produce a dense embedding `h^v_k`.  For multivariate data, the variables are stacked or concatenated before projection.  Additional embeddings for masks and static metadata can be added before or after the projection.

### Handling Missingness and Variable Identities

Missing values are replaced with a sentinel (e.g., zero) before projection, and missingness masks are concatenated or added as separate channels.  Variable identity embeddings indicate the channel origin for each element and are learned jointly with the model.