# Document 51 – Data Augmentation & Synthetic Generation

## Purpose

This document outlines techniques for augmenting time‑series data and using synthetic generation to enrich the AionFM training corpus.  Augmentation increases diversity, reduces overfitting and prepares the model for rare events.  Synthetic series allow controlled experimentation and coverage of behaviours that may be under‑represented in real data.

## Audience

Machine‑learning engineers and researchers preparing data for training.

## Augmentation Techniques

1. **Scaling and rescaling:** multiply entire series or patches by random factors (e.g. between 0.8 and 1.2) to simulate changes in scale.  This teaches scale invariance and robustness.

2. **Noise injection:** add Gaussian or heteroskedastic noise to values.  Vary noise magnitude based on local scale to mimic measurement error or sensor noise.

3. **Time warping:** slightly compress or stretch the time axis to simulate variations in speed (e.g. slower or faster processes).  Use random time dilation factors while preserving overall length.

4. **Time shifting:** shift the series forward or backward by a small number of steps.  Useful for generating additional training samples from limited data.

5. **Mixup:** linearly combine two series or patches with a random weight `α`.  The resulting series inherits properties of both parents; this can improve generalisation and help with regime blending.

6. **Window cropping:** randomly crop sub‑windows from longer series to create diverse contexts.  Ensure that cropping preserves important patterns and regimes.

7. **Permutation and reordering:** shuffle subsequences in low‑dependence segments to generate variations.  Use caution to avoid breaking temporal dependencies.

## Synthetic Series Generation

The synthetic trajectory generator (Doc15) can produce time series with controlled latent structure.  Synthetic generation complements augmentation by creating entirely new sequences rather than modifying existing ones.

### Generator Components

* **Trend models:** linear, piecewise linear, exponential, logistic, polynomial.
* **Seasonality:** sinusoidal, Fourier series, multi‑seasonal, randomised amplitude and phase.
* **Cycles:** cyclical components with random period and amplitude to simulate business cycles or repeated patterns.
* **Event shocks:** step changes, pulse shocks, damped shocks with configurable magnitude and decay.
* **Regime processes:** hidden Markov models or state machines controlling drift and volatility regimes.
* **Noise:** Gaussian, Laplacian, Student‑t, heteroskedastic noise processes; random spikes and outliers.
* **Missingness:** simulate missing intervals or dropout bursts with specified patterns.

### Configuration

* Define ranges for each component (e.g. trend slope, seasonality amplitude, shock probability).
* Sample random parameters to generate a population of synthetic series.  Use domain‑specific prior distributions.
* Optionally condition generation on metadata to simulate multi‑entity corpora (e.g. different store sizes).

### Use Cases

* **Pretraining:** supplement real data with synthetic sequences to expose the model to rare or extreme regimes.
* **Stress testing:** evaluate model behaviour under controlled shocks and scenarios.
* **Explainability research:** generate data with known latent regimes for testing regime token interpretation.

## Integration into Training

* Mix augmented and synthetic data with real data according to sampling weights (e.g. 80% real, 20% synthetic).
* Ensure that augmentation does not leak future information (e.g. avoid time shifting across forecasting horizons).
* Adjust normalization and scaling to account for augmented series.

## Limitations

* Aggressive augmentation may distort the underlying distribution and degrade performance if not carefully balanced.
* Synthetic series cannot fully replace real data; they should be used to cover gaps rather than as the primary source of training.
* Domain‑specific patterns may not be captured by generic augmentation; use domain knowledge to design appropriate augmentations.
