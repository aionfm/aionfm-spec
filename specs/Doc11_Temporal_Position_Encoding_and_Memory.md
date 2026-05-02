# Temporal Position Encoding & Memory

## Purpose & Content

This document describes how AionFM encodes temporal information and maintains long‑term memory.  It defines multiple temporal embedding schemes and details the memory compression mechanism used to handle long contexts efficiently.

## Audience

Core developers and researchers interested in time encoding and memory design.

## Key Sections

- Absolute and relative position encodings
- Calendar and frequency encodings
- Elapsed time and seasonal phase encodings
- Forecast horizon encodings
- Memory segmentation and compression

## Dependencies

Doc10 (Dual‑Stream Causal Backbone)

## Document Body

### Temporal Encoding Schemes

To handle diverse time scales, AionFM combines several encodings:

- **Absolute position:** Index of the patch within the context window.
- **Relative position:** Distance between patches to model recency effects.
- **Calendar position:** Hour of day, day of week, day of year, month, quarter and holiday indicators.
- **Frequency embedding:** Distinguishes minutely, hourly, daily, weekly, monthly or irregular series.
- **Elapsed time:** Captures the time gap between irregularly sampled events.
- **Seasonal phase:** Encodes position within repeating cycles (e.g., daily or annual seasonality).
- **Forecast horizon embedding:** Differentiates between near‑term and long‑term prediction steps.

These encodings are concatenated or added to the patch embeddings before feeding into the causal backbone.

### Memory Mechanism

The model divides historical context into an active window, recent memory and long‑term memory.  The active window receives full attention.  Older segments are compressed into memory states using a learned function `ψ` that summarises long‑term trends, seasonality, shocks and drift.  During inference, memory states are attended to by the backbone, enabling the model to incorporate information from far back in time without attending to every timestamp.