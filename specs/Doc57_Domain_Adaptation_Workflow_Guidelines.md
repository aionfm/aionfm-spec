# Document 57 – Domain Adaptation Workflow Guidelines

## Purpose

This document provides a step‑by‑step workflow for adapting AionFM to new domains or entities.  The adaptation process leverages the pretrained foundation model and uses lightweight adapters, calibration and evaluation to specialize the model while retaining general knowledge.

## Audience

Application developers and data scientists responsible for deploying AionFM in specific domains.

## Workflow Overview

1. **Define the target domain and requirements:** identify the variables to forecast, the relevant covariates, the forecast horizon and accuracy requirements.  Assess whether domain constraints (e.g. non‑negativity) apply.

2. **Prepare domain data:** collect historical series, covariates and metadata for the target domain.  Pre‑process data to align timestamps, handle missingness and normalize values using the same normalization strategy as the pretrained model.

3. **Load pretrained model:** obtain the baseline AionFM weights.  Freeze the core model parameters to retain general knowledge.

4. **Select adaptation strategy:** choose among
   * **Zero‑shot inference:** directly run the model on new data if the domain is similar to pretraining domains.
   * **Few‑shot calibration:** adjust the quantile outputs using a small validation set to improve calibration.
   * **Adapter training:** insert domain‑specific adapter modules and train them on domain data while keeping the core frozen.
   * **Full fine‑tuning:** unfreeze some layers for large domains with abundant data and high performance requirements.

5. **Configure adapters:** decide which layers to attach adapters to (e.g. embedding layers, attention layers) and the size of adapters.  Initialize adapters to identity or small random values.

6. **Train adapters:** train the adapters and any lightweight heads on domain data.  Use the composite loss (Doc13) but limit gradient updates to adapter parameters.  Monitor convergence and avoid overfitting.

7. **Calibrate quantiles:** on a validation set, compute empirical coverage and adjust the calibration layer to align quantiles with nominal levels.  Use isotonic regression or scaling techniques.

8. **Evaluate:** evaluate the adapted model on a hold‑out test set using the metrics defined in Doc28.  Compare performance against baseline (zero‑shot) and domain‑specific models.

9. **Iterate:** if performance is unsatisfactory, adjust adapter architecture (e.g. deeper adapters, additional domain features), increase training data or perform partial fine‑tuning on selected layers.

10. **Deploy:** package the adapted model and calibration parameters for deployment.  Update the serving layer configuration.  Document the adaptation process for reproducibility.

## Tips and Best Practices

* **Data sufficiency:** ensure that the domain dataset covers the range of behaviours needed for forecasting.  Supplement with synthetic data if necessary.
* **Early stopping:** use validation loss and calibration metrics to decide when to stop training adapters.  Avoid overfitting small domains.
* **Hyperparameter tuning:** experiment with adapter size, learning rate and weight decay.  Small adapters may underfit; large adapters may overfit.
* **Cold‑start entities:** for new entities with no history, use metadata embeddings and cross‑entity retrieval to initialize predictions.
* **Continuous adaptation:** retrain or recalibrate adapters periodically to account for distribution shift.
