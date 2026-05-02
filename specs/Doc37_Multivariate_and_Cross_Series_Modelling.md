## Document 37 ‒ Multivariate & Cross‑Series Modelling

### Purpose

To explain how AionFM models relationships between variables and entities, enabling cross‑series forecasting and capturing multivariate dependencies.  Many applications involve multiple target variables (e.g. temperature, pressure) or multiple related series (e.g. stores in the same region).  This document details embedding design, cross‑attention, correlation tokens and metadata‑conditioned attention.

### Intended Audience

Machine‑learning engineers and researchers implementing multivariate extensions of AionFM.

### Key Sections

* **Variable embeddings** – representing each target variable.
* **Entity embeddings** – representing each entity or series.
* **Cross‑variable attention** – learning dependencies across variables.
* **Correlation regimes** – tokens capturing correlation and coupling patterns.
* **Graph‑aware adapters** – leveraging relational structures between entities.
* **Metadata‑conditioned attention** – conditioning attention on metadata.

### Variable Embeddings

For each target variable in a multivariate time series, allocate a distinct embedding vector.  These embeddings allow the model to distinguish variables (e.g. temperature vs. pressure) and learn variable‑specific dynamics.  When variables share similar behaviour, embeddings may cluster; when they differ, embeddings ensure separation.

### Entity Embeddings

When forecasting many related entities (stores, sensors, assets), assign each entity a learnable embedding.  Entity embeddings capture long‑term behaviour and serve as keys for retrieving relevant patterns.  Entities with similar historical behaviour may have similar embeddings, enabling transfer learning and retrieval.

### Cross‑Variable Attention

The dual‑stream backbone can incorporate cross‑variable attention, enabling the model to attend to other variables’ histories when predicting a particular variable.  This captures contemporaneous relationships and lagged effects.  For example, energy load may depend on temperature; cross‑variable attention allows the load head to attend to temperature patches.  Attention weights can be conditioned on metadata or learned automatically.

### Correlation Regimes

Beyond numeric attention, discrete tokens can represent correlation regimes.  The tokenizer may learn tokens indicating “high coupling” or “decoupling” across variables or entities.  These tokens become part of the regime sequence and aid interpretability.  For example, a token might indicate that sales of two products are correlated during promotions but decoupled otherwise.

### Graph‑Aware Adapters

For large panels with known relational structures (e.g. stores connected by supply chains, sensors connected via network topology), graph‑aware adapters can incorporate relational information.  Use graph neural networks or attention mechanisms that leverage adjacency matrices or distance matrices.  The adapter produces embeddings that reflect network connectivity, enabling better cross‑series modelling.

### Metadata‑Conditioned Attention

Condition attention weights on metadata by incorporating static or latent descriptors into the attention score computation.  For example, weighting similarity between series by geographic proximity or product category.  Metadata‑conditioned attention ensures that the model learns relationships that align with domain knowledge.

Effective multivariate and cross‑series modelling allows AionFM to capture richer dependencies and improves forecasting accuracy when multiple interrelated variables or entities are present.