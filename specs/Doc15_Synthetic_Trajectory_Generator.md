## Document 15 ‒ Synthetic Trajectory Generator

### Purpose

To describe the design of the synthetic time‑series generator used to augment AionFM’s training data.  Synthetic trajectories allow controlled exposure to rare or extreme temporal patterns that may not be well represented in real datasets.  By simulating latent structure, the generator helps train the regime tokenizer, calibrate uncertainty and evaluate interpretability.

### Intended Audience

Machine‑learning engineers, researchers and data engineers building or using synthetic data for model training and evaluation.

### Key Sections

* **Generator formulation** – mathematical description of how synthetic series are constructed.
* **Components** – definitions of trend (T), seasonality (S), cycles (C), events (E), regimes (R) and noise (ε).
* **Configurable parameters** – user‑adjustable settings for patterns, regime lengths, noise distributions, frequency.
* **Use cases** – when and how to inject synthetic data into training or evaluation.
* **Integration with training** – how synthetic and real data are mixed and weighted.

### Generator Formulation

A synthetic time series \(x_t\) is generated as a sum of components【594893450643871†L2238-L2252】:

\[x_t = T_t + S_t + C_t + E_t + R_t + \epsilon_t,\]

where:

* **Trend (T_t)** – linear, piecewise linear, exponential or logistic trajectories capturing monotonic growth/decline or saturation.
* **Seasonality (S_t)** – sinusoidal or cyclical functions at daily, weekly or annual periods; multiple seasonalities can be superposed.
* **Cycles (C_t)** – slower oscillations representing business cycles, product lifecycles or macroeconomic factors.
* **Events (E_t)** – deterministic or stochastic shocks at specified times; may cause abrupt level shifts or transient spikes.
* **Regimes (R_t)** – latent state processes (e.g. Markov chains) controlling volatility, mean or autoregressive parameters.  Regime switches simulate structural breaks.
* **Noise (ε_t)** – random fluctuations drawn from distributions such as Gaussian, Student‑t, Poisson or mixture models; may be heteroskedastic or heavy‑tailed.

### Configurable Parameters

Users of the generator can adjust:

* **Trend type and slope** – linear slope, change points, exponential growth rate, saturation level.
* **Seasonal periods and amplitudes** – periods (24 h, 7 d, 365 d), amplitude variation across time.
* **Cycle frequency and phase** – slower cycles representing multi‑year patterns.
* **Event schedule** – number, timing and magnitude of shocks; distribution of shock sizes.
* **Regime switching matrix** – number of regimes, state transition probabilities, regime‑specific parameters (mean, variance, autoregressive coefficients).
* **Noise distribution** – variance, distribution family, autocorrelation structure.
* **Sampling frequency** – from high‑frequency (seconds) to low‑frequency (monthly) series.

### Use Cases

Synthetic trajectories serve several purposes:

* **Data augmentation** – supplement real data with synthetic examples of rare events, volatility explosions or structural breaks.
* **Model pretraining** – pretrain the regime tokenizer and scenario decoder on known regimes to improve interpretability and long‑horizon coherence.
* **Stress testing** – evaluate the model’s response to extreme or hypothetical scenarios, such as sustained shocks or abrupt regime changes.
* **Annotation training** – use synthetic data with known latent regimes to teach human annotators how to label real series for interpretability research.

### Integration with Training

Synthetic series should not replace real data but complement it.  In each training batch, mix synthetic and real samples according to a configurable ratio (e.g. 10–20% synthetic).  Ensure synthetic patterns do not dominate; the model must still learn realistic temporal dynamics from real data.  When evaluating interpretability, hold out synthetic series with known regimes and assess whether the predicted regimes and change points align with ground truth.  Synthetic data can also be used to calibrate scenario generation; if the model can reproduce synthetic shocks accurately, it is likely to generate plausible scenarios on real data.