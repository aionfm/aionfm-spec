## Document 18 ‒ Handling Missing & Irregular Data

### Purpose

To describe strategies for representing and handling missing observations and irregularly sampled time series within AionFM.  Real‑world data frequently contains gaps, irregular sampling intervals or periods with no observations.  AionFM treats missingness as informative rather than simply as noise; this document details the mechanisms for encoding missingness and elapsed time and explains how the model differentiates between genuine flat behaviour and unobserved periods.

### Intended Audience

Data engineers, modellers and researchers preparing inputs for AionFM and developing support for irregular time series.

### Key Sections

* **Missingness masks** – definition and usage.
* **Missingness patterns** – information conveyed by missing periods.
* **Elapsed‑time embeddings** – representing irregular intervals.
* **Irregular sampling** – handling series with variable time gaps.
* **Imputation strategies** – imputation during pre‑processing versus relying on AionFM’s imputation head.

### Missingness Masks

AionFM includes a binary mask \(m_t\) for each observation【594893450643871†L2238-L2252】:

\[m_t = \begin{cases}1 & \text{if observed},\\0 & \text{if missing} \end{cases}.\]

The mask is fed into the model alongside the value and covariate embeddings.  Missingness patterns may signal important events; for example, a sensor outage may indicate a machine failure, and a market closure conveys different dynamics than a zero return.

### Missingness Patterns

Missing data can arise due to sensor failures, market closures, holidays, data ingestion issues or operational disruptions.  Distinguish between:

* **Short gaps** – may indicate an outage or temporary pause.  The model should recognise that the absence of data does not imply zero activity.
* **Long gaps** – could correspond to off‑hours, weekends or non‑operating periods.  Domain knowledge can mark such periods as logically missing rather than zero.

The regime tokenizer can learn tokens that reflect missingness patterns (e.g. “missing burst” tokens), enabling interpretable forecasting around gaps.

### Elapsed‑Time Embeddings

For irregularly sampled series, define \(\Delta t_k = t_k - t_{k-1}\), the time difference between successive observations.  AionFM encodes \(\Delta t_k\) via learned embeddings or continuous transformations, allowing the model to distinguish between no change over a short interval and no observation over a long interval.  The elapsed‑time embedding is combined with the usual temporal position encoding to support irregular sequences.

### Irregular Sampling

When time gaps vary, patch formation must be flexible.  Two strategies:

* **Resampling** – convert irregular series to a regular grid (e.g. hourly) and mark interpolated periods as missing.  This simplifies modelling but may introduce noise.
* **Variable‑length patches** – allow patches to span varying numbers of observations, using \(\Delta t_k\) to signal the effective duration.  The patch encoder can attend to both value and elapsed time vectors.

### Imputation Strategies

Data can be imputed before feeding it to the model (e.g. forward‑fill, linear interpolation, zero‑fill) or left as missing.  AionFM’s imputation head can reconstruct missing values based on context, so aggressive pre‑processing may not be necessary.  However, obvious errors (e.g. isolated missing values due to sensor noise) may still be corrected.  Document the imputation policy used to avoid leaking future information into the past.