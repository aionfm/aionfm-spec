## Document 35 ‒ Constraint‑Aware Forecasting

### Purpose

To provide methods for enforcing domain constraints in AionFM’s forecasts.  Many applications impose hard limits or conservation laws on predicted quantities (e.g. non‑negativity, capacity limits, physical laws).  Without enforcing constraints, forecasts may be invalid or misleading.  This document describes projection techniques and examples.

### Intended Audience

Application developers, planners and researchers implementing constraint handling for AionFM predictions.

### Key Sections

* **Constraint types** – non‑negativity, capacity, conservation, calendar closures, monotonicity, hierarchical reconciliation.
* **Projection methods** – projecting raw forecasts onto the feasible set.
* **Constraint‑aware decoding** – integrating constraints into the decoding process.
* **Examples** – domain‑specific constraint implementations.
* **Trade‑offs** – tension between constraint satisfaction and accuracy.

### Constraint Types

1. **Non‑negativity** – variables such as sales, energy demand and occupancy cannot be negative.  Forecasts must be clipped at zero or transformed via exponential functions.

2. **Capacity limits** – occupancy cannot exceed physical capacity; inventory cannot exceed storage; throughput cannot exceed machine limits.  Forecasts should be capped at capacity.

3. **Conservation laws** – cumulative quantities (e.g. cumulative energy production) should be non‑decreasing; mass and energy balance constraints apply in physical systems.

4. **Calendar closures** – stores or markets may be closed on certain days; demand must be zero or near zero during closure periods.

5. **Monotonicity** – certain variables (e.g. age, cumulative counts) must be non‑decreasing over time.

6. **Hierarchical reconciliation** – forecasts at lower levels must sum to higher level forecasts (e.g. SKU → category → department).

### Projection Methods

Given unconstrained forecasts \(\hat{x}\), project them onto the constraint set \(\mathcal{C}\):

\[\hat{x}_{\text{valid}} = \Pi_{\mathcal{C}}(\hat{x}).\]

Projection can be implemented via:

* **Simple clipping** – clip values below zero or above capacity.
* **Scaling** – if the sum of bottom‑level forecasts exceeds an aggregate constraint, scale all forecasts proportionally to match the aggregate.
* **Optimisation** – solve a quadratic programme or linear programme to find the nearest valid forecast respecting multiple constraints.

### Constraint‑Aware Decoding

Constraints can also be incorporated during decoding.  For example, apply a softplus activation to enforce non‑negativity or add penalty terms to the loss function encouraging capacity adherence.  Hierarchical constraints can be enforced through reconciliation losses during training (see Document 36).  Domain knowledge about closures can be encoded via time masks that zero out predictions during closed periods.

### Examples

* **Retail** – set sales forecasts to zero on holidays when stores are closed; ensure forecasts do not exceed shelf capacity.
* **Energy** – forecast load must be non‑negative; generation forecasts cannot exceed capacity; ensure sum of substation forecasts matches regional forecast.
* **Healthcare** – bed occupancy cannot exceed bed count; procedure counts must be integer and non‑negative.
* **Industrial** – throughput cannot exceed machine capacity; cumulative production must increase monotonically.

### Trade‑Offs

Enforcing constraints may slightly reduce accuracy if the unconstrained forecast is close to, but not within, the feasible set.  However, valid forecasts are more valuable in practice than slightly more accurate but invalid forecasts.  Fine‑tune the balance by combining constraint projection with calibration and optimisation.