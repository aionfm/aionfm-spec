## Document 19 ‒ Probabilistic Forecasting

### Purpose

To explain how AionFM produces full predictive distributions, how to interpret its quantile and interval outputs, and why distributional forecasting is necessary for risk‑aware decision making.  Unlike point predictors, probabilistic forecasters convey the range of plausible futures and their associated probabilities.  This document details the design of AionFM’s distribution outputs and provides guidance on using them in practice.

### Intended Audience

Quantitative researchers, analysts and practitioners who need to make decisions under uncertainty using AionFM’s probabilistic outputs.

### Key Sections

* **Conditional distribution definition** – AionFM models \(P(x_{t+1:t+H} | x_{1:t}, c_{1:t}, z_{1:t}, m_{1:t})\) where \(x\) are observed values, \(c\) are regime tokens, \(z\) covariates and \(m\) missingness masks【594893450643871†L2238-L2252】.
* **Point vs. probabilistic forecasts** – limitations of single‑value predictions.
* **Quantile forecasts** – interpretation of quantiles and prediction intervals.
* **Distribution heads** – optional parametric distributions and mixture models.
* **Calibration and sharpness** – evaluating probabilistic forecasts.

### Conditional Distribution

AionFM treats forecasting as estimating a conditional probability distribution of future values given past observations, covariates and regime tokens.  This distribution may be summarised through quantiles, intervals or parametric parameters.  The distribution captures both aleatoric uncertainty (irreducible randomness) and epistemic uncertainty (model uncertainty and regime ambiguity).

### Point vs. Probabilistic Forecasts

Point forecasts (mean or median) provide a single scenario but ignore the variability of possible futures.  Two series with the same mean forecast might have very different risk profiles if one has wide uncertainty bands.  For applications such as inventory management, capacity planning, risk management and anomaly detection, the width and shape of the predictive distribution are as important as the central tendency.

### Quantile Forecasts

AionFM’s quantile head outputs values \(\hat{q}_{\tau, t+h}\) for a set of quantile levels \(\tau\) (e.g. 0.01, 0.05, 0.1, 0.25, 0.5, 0.75, 0.9, 0.95, 0.99).  Each quantile corresponds to the value below which the future observation falls with probability \(\tau\).  Prediction intervals can be formed by taking two quantiles (e.g. the 10th and 90th) as lower and upper bounds.  The model learns quantile functions through the pinball loss defined in the training objectives document.  When interpreting quantiles, note that they may cross if the model is not properly calibrated; monotonicity constraints or post‑processing can address this issue.

### Distribution Heads

For some domains, a full parametric distribution may be desirable.  AionFM’s distribution head can output parameters for distributions such as Gaussian (mean and variance), Student‑t (mean, scale, degrees of freedom), negative binomial (count processes), log‑normal (for strictly positive values) or mixture models.  Users can then compute densities, likelihoods or draw samples.  Mixture distributions allow multimodal behaviour; for example, a demand distribution may have separate peaks for normal and promotional periods.

### Calibration and Sharpness

Probabilistic forecasts should be both **calibrated** (the empirical frequency of outcomes within a prediction interval matches the nominal level) and **sharp** (the intervals are narrow without being over‑confident).  Calibration can be assessed by coverage plots, the calibration loss and probability integral transform (PIT) histograms.  Sharpness is evaluated relative to calibration: for two equally calibrated models, the one with narrower intervals is preferred.  When using AionFM, monitor coverage across horizons and adjust training weights or calibration procedures if intervals are consistently too wide or too narrow.