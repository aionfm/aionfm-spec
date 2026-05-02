## Document 17 ‒ Covariate Integration

### Purpose

To provide detailed guidance on incorporating covariates and metadata into AionFM.  Covariates enrich the temporal context with exogenous information such as calendar features, price changes, weather forecasts and static entity attributes.  Proper integration of covariates improves forecast accuracy, interpretability and adaptability.

### Intended Audience

Data engineers and modellers responsible for preparing inputs and implementing covariate encoding.

### Key Sections

* **Covariate types** – known future covariates, historical covariates, static metadata and latent metadata.
* **Encoding methods** – normalisation, embeddings, one‑hot encoding, continuous transformations.
* **Integration points** – where covariate embeddings are fused with value and regime streams.
* **Dynamic vs. static covariates** – handling time‑varying versus entity‑fixed attributes.
* **Examples** – domain‑specific covariate lists.

### Covariate Types

AionFM supports four classes of covariates【594893450643871†L2238-L2252】:

1. **Known future covariates** – variables whose future values are known or planned.  Examples include calendar features (hour of day, day of week, month), scheduled promotions, maintenance schedules, holidays, price changes, capacity adjustments and policy interventions.  These covariates influence future behaviour and can be provided alongside the forecast horizon.

2. **Historical covariates** – variables observed only in the past.  Examples include past temperature, realised prices, past traffic volumes, inventory levels, historical sensor readings and economic indicators.  They provide context for estimating relationships and seasonality.

3. **Static metadata** – attributes that do not change over time.  Examples include store type, region, product category, sensor location, machine type, asset class, customer segment and facility size.  Static metadata is encoded once per series or entity and influences the model’s baseline and seasonal components.

4. **Latent metadata** – learned descriptors inferred from historical behaviour.  Without explicit metadata, the model can cluster entities with similar patterns and attach latent codes describing those clusters.  These codes are learned during pretraining via embedding tables or vector quantisation.

### Encoding Methods

* **Numerical covariates** – scaled using z‑score or robust scaling, then passed through linear or non‑linear transformations.
* **Categorical covariates** – encoded via learned embeddings or one‑hot vectors.  For high‑cardinality categories, shared embeddings reduce parameters.
* **Temporal covariates** – such as calendar features, represented via cyclical encodings (e.g. sine/cosine for day of week) or learned embeddings.  Forecast horizon embeddings differentiate near‑term versus long‑term covariate effects.
* **Covariate embeddings** – concatenated or added to value and regime embeddings via gated fusion.  AionFM uses separate gates (\(\gamma_k\)) to control the influence of covariate embeddings relative to value and regime streams.

### Integration Points

Covariates can be integrated at multiple stages:

* **Input stage** – attach historical and future covariates to each patch before encoding.  For example, augment the continuous patch vector with covariate values, or create parallel covariate embeddings.
* **Fusion stage** – fuse covariate embeddings with value and regime embeddings using the gating mechanism described in the backbone.  The learned gate weights \(\gamma_k\) determine how much covariate information influences each patch.
* **Output stage** – some output heads, such as distribution parameters or scenario generation, may condition on future covariate values directly.  For example, scenario paths can reflect scheduled promotions or weather forecasts.

### Dynamic vs. Static Covariates

Dynamic covariates vary over time and are fed in at each time step or patch, while static metadata remains constant for an entity.  Static metadata can be embedded once and either concatenated to each patch or used to index entity‑specific adapter modules.  Dynamic covariates require careful alignment with the target horizon; known future covariates must be provided for the forecast period, while historical covariates are only available up to the present.

### Examples

* **Retail** – price changes, promotion flags, competitor prices, marketing spend (known future); historical sales drivers such as traffic, past prices, past promotions; store metadata (size, type, region).
* **Energy** – temperature forecasts, wind forecasts, planned generation capacity (known future); past weather, past load; plant characteristics, region identifiers.
* **Healthcare** – scheduled procedures, staffing plans (known future); historical patient arrivals, disease incidence; facility type and location.
* **Finance** – scheduled economic releases, earnings announcements (known future); past returns, volatility; asset class, sector.

Proper covariate integration ensures that AionFM can leverage external signals and entity attributes to improve forecasting accuracy and interpretability.