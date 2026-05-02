## Document 13 ‒ Training Objectives & Loss Functions

### Purpose

This document describes the composite objective function used to train AionFM and elaborates on each of its constituent loss components.  Because AionFM must simultaneously predict future values, quantiles, regimes, scenarios and impute missing data, training requires a carefully balanced set of losses.  Understanding these objectives is essential for ML engineers who will implement and tune the training loop.

### Intended Audience

Machine‑learning engineers and researchers developing or tuning AionFM training pipelines.

### Key Sections

* **Composite objective overview** – definition of the total loss and role of weighting coefficients.
* **Next‑patch prediction loss** – encourages accurate point forecasts.
* **Masked reconstruction loss** – improves imputation and robustness.
* **Token likelihood loss** – trains the regime tokenizer to predict behavioural states.
* **Quantile pinball loss** – calibrates uncertainty through quantile predictions.
* **Rollout consistency loss** – enforces long‑horizon coherence for scenario generation.
* **Contrastive alignment loss** – aligns representations across scales and windows.
* **Calibration loss** – penalises systemic over‑ or underestimation of interval coverage.
* **Loss weighting strategies** – guidelines for balancing terms in different regimes.

### Composite Objective

AionFM minimises a weighted sum of several loss terms:

\[\mathcal{L}=\lambda_1\mathcal{L}_{\text{next}} + \lambda_2\mathcal{L}_{\text{mask}} + \lambda_3\mathcal{L}_{\text{token}} + \lambda_4\mathcal{L}_{\text{quantile}} + \lambda_5\mathcal{L}_{\text{rollout}} + \lambda_6\mathcal{L}_{\text{contrast}} + \lambda_7\mathcal{L}_{\text{calibration}},\]

where \(\lambda_i\) are non‑negative weights chosen to balance competing objectives【594893450643871†L2238-L2252】.  In practice, the weights may be tuned per dataset or during early training to emphasise core forecasting performance before emphasising calibration or scenario quality.

### Loss Components

* **Next‑patch prediction (\(\mathcal{L}_{\text{next}}\))** – measures the distance between the true next patch and the predicted one.  A simple choice is mean absolute error, but squared error or Huber loss can be used.  This term ensures accurate point forecasts.

* **Masked reconstruction (\(\mathcal{L}_{\text{mask}}\))** – randomly masks historical patches and requires the model to reconstruct them.  This enhances the model’s ability to impute missing values, deal with corrupted data and learn consistent representations beyond simple next‑step prediction.

* **Token likelihood (\(\mathcal{L}_{\text{token}}\))** – negative log‑likelihood of the correct regime token given previous tokens and patches.  It trains the residual temporal language to predict behavioural states, essential for interpretability and scenario sampling.

* **Quantile pinball (\(\mathcal{L}_{\text{quantile}}\))** – for each quantile level \(\tau\), the loss is \(\max\{\tau(y-\hat{q}_\tau),(\tau-1)(y-\hat{q}_\tau)\}\).  Minimising this encourages the predicted quantiles to align with empirical quantiles, producing calibrated prediction intervals.

* **Rollout consistency (\(\mathcal{L}_{\text{rollout}}\))** – penalises divergence between multi‑step rollouts and the true future sequence.  Without this term, autoregressive generation may accumulate error across horizons.  The rollout loss trains the model to maintain coherence when generating long sequences.

* **Contrastive alignment (\(\mathcal{L}_{\text{contrast}}\))** – aligns representations across short, medium and long temporal windows.  By contrasting representations of similar regimes at different scales, the model learns to recognise patterns that recur across horizons, improving detection of structural breaks and rare events.

* **Calibration (\(\mathcal{L}_{\text{calibration}}\))** – measures mismatch between the nominal and empirical coverage of prediction intervals.  For example, the empirical coverage of a 90% interval should be close to 90%.  Over‑confident or under‑confident models are penalised.

### Loss Weighting Strategies

Selecting \(\lambda_i\) is crucial.  Early in training, one might emphasise \(\mathcal{L}_{\text{next}}\) and \(\mathcal{L}_{\text{token}}\) to learn accurate forecasts and a robust token vocabulary.  Once the model predicts well, the weight on \(\mathcal{L}_{\text{quantile}}\) and \(\mathcal{L}_{\text{calibration}}\) can be increased to improve uncertainty estimates.  \(\mathcal{L}_{\text{rollout}}\) becomes important when training scenario generation; initially it may be small to avoid unstable gradient signals.  Domain‑specific tasks may require adjusting weights or adding domain‑specific losses (e.g. for hierarchical reconciliation or constraint satisfaction).