## Document 23 ‒ Use Cases – Retail

### Purpose

To describe how AionFM can be applied to retail demand forecasting.  Retailers must forecast sales for thousands of products across stores, considering promotions, pricing, holidays and inventory constraints.  This document provides domain‑specific guidance on covariates, hierarchies and scenario usage.

### Intended Audience

Retail planners, merchandisers and data scientists deploying AionFM for retail demand forecasting.

### Key Sections

* **Domain characteristics** – intermittent demand, promotions, price sensitivity, product hierarchies.
* **Covariates** – sales drivers including promotions, prices and holidays.
* **Hierarchical structure** – SKU → product category → store → region.
* **Evaluation metrics** – metrics appropriate for intermittent and highly skewed demand.
* **Scenario usage** – planning promotions, inventory and service levels.

### Domain Characteristics

Retail demand is often intermittent (many zeros) and can vary sharply due to promotions and price changes.  Hierarchies are common: SKUs belong to categories, categories to departments, stores to regions.  Cross‑product cannibalisation and substitution effects may occur.  Data quality varies by store and product; new items have little history.

### Covariates

Important covariates include:

* **Price** – list price, discounts, competitor prices.
* **Promotions** – start and end dates, promotion type (e.g. discount, 2‑for‑1), promotional depth.
* **Marketing spend** – advertising campaigns, loyalty programmes.
* **Calendar events** – holidays, payday periods, seasonal events.
* **Inventory constraints** – stock levels, replenishment lags, shelf capacity.
* **External factors** – weather (for certain products), foot traffic, macroeconomic indicators.

### Hierarchical Structure

Forecasts should be coherent across levels.  AionFM supports hierarchical reconciliation so that SKU‑level forecasts aggregate to category‑ and store‑level totals.  Incorporate static metadata such as store size, region and product attributes via the covariate integration methods described earlier.  Hierarchical constraints should be enforced through reconciliation loss functions or post‑processing.

### Evaluation Metrics

Retail forecasts are evaluated using metrics such as weighted absolute percentage error (WAPE), mean absolute scaled error (MASE), median absolute percentage error (MdAPE) and prediction interval coverage.  For intermittent demand, use specialised metrics like the intermittent scaled error or zero‑inflated loss.  Assess performance across products, categories and stores, and monitor bias during promotions.

### Scenario Usage

Scenarios can support:

* **Promotion planning** – generate demand scenarios under different promotion depths or timings to estimate uplift and assess stock requirements.
* **Inventory optimisation** – combine quantile forecasts with cost models to determine reorder points and safety stock levels.
* **Portfolio planning** – evaluate how product additions or removals impact total category demand and cross‑product cannibalisation.
* **Risk management** – identify the probability of stockouts or overstock under various demand regimes and plan mitigation strategies.

AionFM’s ability to forecast promotions, handle hierarchies and generate scenarios makes it suitable for complex retail environments.