## Document 27 ‒ Use Cases – Finance & Economics

### Purpose

To provide guidance on using AionFM for financial and economic time series, including returns, volumes, volatility, liquidity measures and macroeconomic indicators.  Financial markets and economic indicators exhibit non‑stationarity, heavy tails and regime shifts; robust probabilistic forecasting is crucial for risk management and policy analysis.

### Intended Audience

Financial researchers, quantitative analysts, risk managers and economists applying AionFM.

### Key Sections

* **Financial series characteristics** – high noise, volatility clustering, heavy tails, market microstructure.
* **Economic indicators** – macro series with structural breaks.
* **Covariates** – macro announcements, sentiment, liquidity, cross‑asset signals.
* **Risk scenarios** – stress testing and tail events.
* **Regulatory considerations** – fair disclosure and model governance.

### Financial Series Characteristics

Financial time series (returns, price changes, volume, volatility) are often:

* **Non‑stationary** – drift and volatility change over time; distribution parameters vary across regimes.
* **Heteroskedastic** – volatility clusters; periods of calm alternate with turbulence.
* **Heavy‑tailed** – extreme events occur more frequently than Gaussian models predict.
* **Multivariate** – cross‑asset correlations shift over time.
* **Market microstructure** – high‑frequency data exhibit bid–ask effects and discretisation due to tick sizes.

### Economic Indicators

Macroeconomic series such as inflation, employment, GDP proxies and interest rates may show structural breaks due to policy changes, crises or measurement revisions.  They often have lower frequency (monthly, quarterly) and longer lags.  Modeling them jointly with financial series may improve risk assessment and macro‑hedging strategies.

### Covariates

Include:

* **Macro announcements** – scheduled releases (employment reports, CPI, FOMC statements) and their surprises relative to expectations.
* **Market sentiment** – news sentiment scores, social media signals.
* **Liquidity measures** – bid–ask spreads, order book depth, trading halts.
* **Cross‑asset signals** – returns or volatility from related assets (e.g. stock and bond markets).
* **Economic policy indicators** – interest rates, policy uncertainty indices, fiscal stimulus.

### Risk Scenarios

Financial and economic forecasting is often used for stress testing and risk management.  Use AionFM scenarios to:

* **Assess tail risk** – sample extreme volatility expansion regimes and evaluate portfolio drawdowns or Value‑at‑Risk (VaR).
* **Stress macro shocks** – simulate unexpected inflation spikes, interest rate jumps or geopolitical events.
* **Test hedging strategies** – evaluate performance of hedges across different regime scenarios and cross‑asset correlations.
* **Analyse liquidity crises** – model scenarios with sudden drops in trading volume or spikes in spreads.

### Regulatory Considerations

Financial applications are subject to regulations such as fair disclosure and model risk management.  When using AionFM, document data sources, model assumptions, calibration procedures and scenario generation techniques.  Provide interpretability outputs for audit and oversight.  For macroeconomic indicators, consider confidentiality of preliminary releases and data revisions.

AionFM’s probabilistic forecasting, scenario generation and interpretability make it a powerful tool for financial and economic analysis when combined with rigorous governance.