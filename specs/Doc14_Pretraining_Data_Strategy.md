## Document 14 ‒ Pretraining Data Strategy

### Purpose

To define the strategy for assembling the diverse data mixture used to pretrain AionFM.  The pretraining phase must expose the model to a broad range of temporal behaviours, domains and patterns so that it generalises well to unseen applications.  This document outlines the domain coverage, dataset selection criteria, sampling strategies and quality controls necessary for building the pretraining corpus.

### Intended Audience

Data engineers, ML engineers and researchers responsible for preparing the pretraining dataset.

### Key Sections

* **Domain coverage** – list of domains included in the pretraining mix (retail, energy, sensors, weather, mobility, healthcare, finance, economics, manufacturing, supply chain, synthetic).
* **Dataset sourcing** – guidelines for selecting public or private datasets and ensuring legal compliance.
* **Sampling strategies** – methods for balancing domains, frequencies and regimes.
* **Quality control** – checks for duplicates, unit consistency, outliers, missingness and schema drift.
* **Diversity metrics** – metrics to track diversity of patterns and regimes in the corpus.

### Domain Coverage

AionFM pretraining leverages a heterogeneous mix of real and synthetic time‑series corpora【594893450643871†L2238-L2252】.  The real data should cover:

* **Retail sales** – daily or weekly sales for many items and stores, including promotions and price changes.
* **Energy demand** – load at various temporal resolutions (hourly, subhourly) with temperature and calendar covariates.
* **Web traffic** – page views, clicks or user sessions across multiple sites.
* **Industrial sensors** – high‑frequency measurements such as vibration, pressure, temperature and throughput.
* **Weather and environmental** – temperature, precipitation, wind and other meteorological variables.
* **Transportation and mobility** – passenger counts, congestion levels, travel times, shipment volumes.
* **Healthcare operations** – patient arrivals, bed occupancy, procedure counts, staffing levels.
* **Financial markets** – returns, volumes, volatility indices, liquidity metrics.
* **Economic indicators** – inflation, employment, GDP proxies, interest rates.
* **Manufacturing telemetry** – quality metrics, production counts, downtime events.
* **Supply‑chain flows** – inbound/outbound shipments, inventory levels, lead times.

### Dataset Sourcing

Datasets may come from open‑source repositories, publicly available APIs or partner organisations.  Each source should be evaluated for:

* **Licensing and privacy** – ensure the data can be used for research and model training without violating privacy or proprietary agreements.
* **Temporal resolution** – prefer data with clear timestamps and consistent frequency; irregular series are acceptable if accompanied by time stamps.
* **Covariate availability** – select datasets that include relevant exogenous variables (weather, price, promotions, calendar events).
* **Domain diversity** – avoid over‑representing one domain; maintain a balanced mix.

### Sampling Strategies

To prevent the model from overfitting to dominant domains, sampling weights can be assigned.  For example, each domain might contribute an equal number of samples or be weighted by the inverse of its size.  Stratified sampling ensures that high‑frequency series, sparse series and hierarchical series are all represented.  When constructing batches, mix series from different domains to encourage cross‑domain generalisation.  Additionally, ensure that training covers a wide range of regimes (drift, shocks, volatility expansions) by including examples with those patterns.

### Quality Control

Quality checks should be run before adding a dataset to the pretraining mix:

* **Schema validation** – verify the presence of timestamps, target values, covariates and identifiers.
* **Unit consistency** – ensure units are consistent within a dataset (e.g. same currency, same temperature unit).
* **Missingness analysis** – measure missing rates and decide whether to impute, mask or exclude.
* **Outlier detection** – identify extreme values that may be errors; either correct or remove them.
* **Duplicate removal** – remove duplicate timestamps or records.
* **Normalization feasibility** – check that local scale and mean can be computed; for strictly positive series, consider log transforms.

### Diversity Metrics

Track metrics to quantify the diversity of the pretraining corpus.  Examples include:

* **Entropy of regimes** – measure distribution of regime tokens across the corpus to ensure rare regimes are seen.
* **Coverage of horizons** – track the distribution of series lengths and forecast horizons.
* **Frequency mix** – ensure a balance between minute‑level, hourly, daily and weekly series.
* **Domain entropy** – measure the proportion of samples contributed by each domain.

Maintaining a diverse and high‑quality pretraining corpus is crucial for achieving the zero‑shot generalisation goals of AionFM.