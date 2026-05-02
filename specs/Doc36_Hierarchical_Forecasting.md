## Document 36 ‒ Hierarchical Forecasting

### Purpose

To explain how AionFM supports forecasting at multiple hierarchical levels and ensures coherence between aggregate and disaggregate forecasts.  Hierarchical forecasting is common in retail (SKU→category→store→region), energy (feeder→substation→region), and other applications.  This document describes summing matrices, reconciliation methods and practical examples.

### Intended Audience

Data scientists, analysts and developers building hierarchical forecasting pipelines with AionFM.

### Key Sections

* **Hierarchy definitions** – how entities are organised into aggregation levels.
* **Summing matrices** – representation of hierarchical relationships.
* **Reconciliation methods** – adjusting forecasts to ensure coherence.
* **Loss functions** – hierarchical reconciliation loss and training strategies.
* **Examples** – applying hierarchical forecasting in retail and energy.

### Hierarchy Definitions

A hierarchy is defined by a set of levels and a mapping from bottom‑level series to higher levels.  For example:

```text
Company
  Region
    Store
      Product Category
        SKU
```

Each SKU belongs to exactly one product category, each category to one store, etc.  Hierarchies can be more complex (e.g. cross‑classification), but the principle is that sums of lower‑level series equal higher‑level totals.

### Summing Matrices

Represent hierarchical relationships using a summing matrix \(S\).  If \(\hat{y}_b\) is the vector of bottom‑level forecasts, aggregate forecasts \(\hat{y}_a\) are given by:

\[\hat{y}_a = S \hat{y}_b.\]

For example, if there are 3 SKUs in a category, the summing matrix sums the first 3 elements of \(\hat{y}_b\) to form the category forecast.  Summing matrices can be constructed automatically based on metadata tables.

### Reconciliation Methods

Unreconciled forecasts may violate hierarchical constraints.  Methods to enforce coherence include:

* **Bottom‑up** – aggregate bottom‑level forecasts using the summing matrix.  Simple but can propagate bottom‑level noise to aggregates.
* **Top‑down** – forecast aggregate series and distribute them to lower levels using proportions (historical or predicted).  Requires distribution keys.
* **Middle‑out** – forecast an intermediate level (e.g. category) and reconcile up and down.
* **Optimal reconciliation** – solve a least‑squares problem to adjust forecasts while minimising the difference from original forecasts subject to \(S\hat{y}_b = \hat{y}_a\).  Also known as the MinT method.

AionFM can learn to produce coherent forecasts by incorporating a **hierarchical reconciliation loss** during training:

\[\mathcal{L}_{\text{hier}} = \|\hat{y}_a - S\hat{y}_b\|.\]

This loss encourages bottom‑level forecasts to sum up correctly.  Alternatively, reconciliation can be done as a post‑processing step via optimisation.

### Loss Functions

Include the reconciliation loss along with point and probabilistic losses.  Weight it to prevent dominating other objectives.  When training across many entities, one can use mini‑batches containing series from the same hierarchy to compute the loss efficiently.

### Examples

* **Retail** – forecast SKU sales and ensure that they sum to category and store forecasts.  Use static metadata (category membership) to build the summing matrix.
* **Energy** – forecast feeder loads and reconcile to substation and regional loads.  Use network topology to define summing matrices.

Hierarchical forecasting ensures consistent projections across organisational levels and supports planning at multiple granularities.