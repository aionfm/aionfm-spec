## Document 29 ‒ Benchmarking Protocol

### Purpose

To specify procedures for benchmarking AionFM under various deployment‑like scenarios.  Benchmarking provides objective comparisons with alternative models and reveals performance across zero‑shot, few‑shot, fine‑tuned, long‑horizon and stress settings.  This document outlines dataset selection, experiment design and fairness considerations.

### Intended Audience

Evaluators and researchers designing experiments to compare AionFM with other forecasting methods.

### Key Sections

* **Benchmark scenarios** – zero‑shot, fine‑tuned, few‑shot, long‑horizon and stress benchmarks.
* **Dataset selection** – criteria for choosing benchmark datasets and ensuring diversity.
* **Experiment design** – training protocols, hyperparameter selection, replication.
* **Fairness considerations** – controlling for covariate availability and data splits.
* **Reporting guidelines** – metrics and visualisations.

### Benchmark Scenarios

1. **Zero‑shot benchmark** – evaluate AionFM on domains not seen during pretraining without additional training.  Measure generalisation and identify blind spots.

2. **Fine‑tuned benchmark** – fine‑tune AionFM on a domain‑specific training set and evaluate on held‑out data.  Compare with domain‑specific models to assess adaptation ceiling and data efficiency.

3. **Few‑shot benchmark** – provide a small number of labelled examples for calibration or adapter training.  Evaluate the improvement over zero‑shot and the gap to full fine‑tuning.

4. **Long‑horizon benchmark** – forecast further into the future (beyond typical horizons) to test rollout coherence and uncertainty widening.  Evaluate both point and probabilistic metrics across horizons.

5. **Stress benchmark** – introduce shocks, missing data bursts, regime shifts, covariate corruption or extreme events.  Evaluate robustness and scenario usefulness.

### Dataset Selection

Choose datasets that represent the diversity of domains and regimes AionFM is designed to handle.  Include series with different frequencies, horizons, levels of missingness and covariates.  Public datasets (e.g. M4, UCI energy, traffic, electricity, weather) and proprietary corpora can be used.  For fair comparisons, ensure that all models have access to the same covariate information and splits.  Document dataset characteristics and pre‑processing steps.

### Experiment Design

* **Training protocol** – specify the number of epochs, batch size, learning rate schedule and loss weights.  Use early stopping to prevent overfitting.
* **Hyperparameter selection** – tune hyperparameters on validation sets separately for each model.  Use grid search or Bayesian optimisation within a pre‑defined budget.
* **Replication** – run each experiment multiple times with different random seeds to account for variability.  Report mean and standard deviation of metrics.
* **Baselines** – include classical statistical models (ARIMA, exponential smoothing), tree‑based models, deep learning models (RNN, TCN, Transformer) and domain‑specific baselines for context.

### Fairness Considerations

Ensure fairness by:

* Using identical training/validation/test splits across models.
* Providing identical covariates to all models; if a baseline cannot use certain covariates, run an ablation for AionFM without those covariates.
* Reporting the same metrics and horizons for all models.
* Normalising evaluation where necessary (e.g. scaling errors) to allow cross‑series comparison.

### Reporting Guidelines

* Present metric tables and plots across horizons and segments.
* Include calibration plots (e.g. reliability diagrams) and scenario visualisations.
* Report training time, inference latency and resource usage.
* Describe data pre‑processing, hyperparameters and tuning procedures.
* Discuss strengths and weaknesses observed in each scenario and highlight unusual failure modes.

Consistent, fair benchmarking enables the community to understand where AionFM excels and where improvements are needed.