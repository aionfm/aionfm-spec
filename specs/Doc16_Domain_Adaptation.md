## Document 16 ‒ Domain Adaptation

### Purpose

To describe how AionFM adapts to new domains and data distributions without retraining the entire model from scratch.  Domain adaptation is critical because the pretraining mixture may not perfectly match a deployment domain.  This document outlines zero‑shot inference, few‑shot calibration, adapter training and full fine‑tuning procedures, providing guidelines and examples.

### Intended Audience

Modellers and application developers seeking to deploy AionFM in new domains or customise it for specific use cases.

### Key Sections

* **Adaptation modes** – zero‑shot, few‑shot calibration, adapter tuning, full fine‑tuning.
* **Zero‑shot guidelines** – when to rely on the pre‑trained model out of the box.
* **Few‑shot calibration** – adjusting scale, quantiles, seasonality and residual distributions.
* **Adapter modules** – training lightweight adapters for domain‑specific features and frequency adjustments.
* **Full fine‑tuning** – criteria for fine‑tuning the core model and considerations for stability.
* **Practical examples** – adaptation recipes for common domains.

### Adaptation Modes

1. **Zero‑shot forecasting** – Use AionFM directly on a new series without additional training.  Suitable when the domain resembles those in the pretraining mix and only limited data is available.  Covariates and metadata should be aligned with the pretraining schema.  Evaluate performance using short validation windows; if error and calibration are acceptable, zero‑shot deployment may suffice.

2. **Few‑shot calibration** – When the pre‑trained model’s scale or quantile coverage is off, use a small amount of domain‑specific data to recalibrate.  Calibration can adjust the output scale, seasonal phase offsets, quantile biases and residual variance estimates.  Techniques include quantile regression on residuals and histogram matching.  Few‑shot calibration is low‑cost and avoids overfitting.

3. **Adapter tuning** – Insert lightweight adapter modules at specific layers of the model.  Adapters consist of small bottleneck networks that specialise the pre‑trained representations for the target domain.  Train adapters on domain data while keeping the backbone frozen.  Adapters can specialise by frequency (e.g. hourly vs. daily), industry (energy vs. retail), entity type (store vs. machine) or region.  Because only adapter parameters are updated, tuning is fast and preserves general knowledge【594893450643871†L2238-L2252】.

4. **Full fine‑tuning** – Retrain all or most model parameters on domain data.  Use this when the domain is very different from pretraining or when high accuracy is critical.  Fine‑tuning requires more data and careful regularisation to avoid catastrophic forgetting.  Start with a small learning rate, monitor calibration and avoid overfitting by using early stopping and data augmentation.

### Practical Examples

* **New retail categories** – For a new product category with sparse history, begin with zero‑shot forecasts.  If the model consistently underestimates demand, perform few‑shot calibration using the first few weeks of sales.  If there are known promotion effects not learned during pretraining, train a promotion‑specific adapter.

* **Industrial sensor** – A new sensor type may exhibit higher sampling frequency and unique failure patterns.  Introduce a frequency‑specific adapter that accounts for the sampling interval and uses latent metadata describing the machine.  If the sensor exhibits dynamics not seen during pretraining, fine‑tune the model on historical sensor data.

* **Regional energy grid** – When deploying AionFM to a grid with different seasonality patterns, calibrate the seasonal phase and amplitude using a few months of data.  Train a region‑specific adapter to capture region‑unique behaviours without retraining the full model.

Domain adaptation is an ongoing process.  Monitor post‑deployment performance and recalibrate or update adapters as the domain evolves.