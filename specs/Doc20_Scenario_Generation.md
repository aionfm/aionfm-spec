## Document 20 ‒ Scenario Generation

### Purpose

To specify the two‑stage process by which AionFM generates full future scenarios and to provide guidelines for using these scenarios in decision‑making.  Scenario generation transforms uncertainty into concrete temporal paths that can be analysed, compared and stress‑tested.  This document explains how scenarios are sampled, how they should be interpreted and how to customise them to different risk profiles or planning needs.

### Intended Audience

Planners, risk analysts and developers building scenario‑based forecasting applications with AionFM.

### Key Sections

* **Stage 1: Regime sampling** – sampling future behavioural tokens.
* **Stage 2: Value decoding** – decoding values conditioned on regimes.
* **Scenario types** – conservative, median, optimistic, tail‑risk, stress and regime‑specific scenarios.
* **Using scenarios** – applications in risk simulation, planning, stress testing, inventory management and portfolio analysis.
* **Guidelines** – number of scenarios, random seeds, reproducibility, scenario filtering.

### Stage 1: Regime Sampling

AionFM first samples a sequence of future behavioural tokens \(\tilde{c}_{t+1:t+H}\) from the conditional distribution \(P(c_{t+1:t+H} | x_{1:t})\)【594893450643871†L2238-L2252】.  These tokens represent a plausible path of future regimes – such as continued growth, reversal, shock and recovery, volatility expansion, seasonal decline or plateau.  Regime sampling can be biased towards particular outcomes (e.g. sampling only from tails) or left unbiased to reflect the model’s posterior belief.

### Stage 2: Value Decoding

Given a sampled regime path, the value decoder generates values \(\tilde{x}_{t+1:t+H}\) from \(P(x_{t+1:t+H} | \tilde{c}_{t+1:t+H}, x_{1:t})\).  The decoder conditions on the historical context, future covariates and the sampled regimes to produce a coherent trajectory.  Because regimes encode residual dynamics, the decoder can tailor values to the qualitative behaviour implied by the tokens (e.g. producing a shock followed by a slow recovery).

### Scenario Types

Different planning contexts require different scenario emphases:

* **Conservative scenarios** – emphasise lower quantiles or adverse regimes.  Useful for risk‑averse planning or capacity buffering.
* **Median scenarios** – use median regimes and quantiles; represent “most likely” futures.
* **Optimistic scenarios** – sample from upper regimes and higher quantiles; used to plan for high demand or favourable outcomes.
* **Tail‑risk scenarios** – explicitly sample from extreme regimes (e.g. sustained shocks, extreme volatility).  Help quantify exposure to unlikely but impactful events.
* **Stress scenarios** – user‑specified events or sequences; for example, forcing a shock token at a particular date to simulate a hypothetical promotion or outage.
* **Regime‑specific scenarios** – focus on particular regime transitions (e.g. growth to reversal) to understand resilience.

### Using Scenarios

Scenarios can be used to:

* **Perform risk simulation** – run simulations of inventory levels, capacity usage or financial returns under multiple trajectories to estimate risk metrics (VaR, CVaR, stockout probability).
* **Plan capacity and inventory** – determine safety stock or resource buffers by examining high‑demand trajectories.
* **Stress test systems** – test the robustness of plans under extreme or hypothetical conditions; identify breakpoints.
* **Evaluate strategies** – test policies (pricing, promotions, staffing) across many possible futures.
* **Communicate uncertainty** – visualise multiple possible futures to stakeholders and emphasise that forecasts are distributions rather than certainties.

### Guidelines

* **Number of scenarios** – choose a number commensurate with decision complexity (e.g. 100–1000 scenarios for Monte‑Carlo simulations; fewer for qualitative analysis).
* **Random seeds** – set seeds for reproducibility; store seed values alongside results.
* **Scenario filtering** – discard implausible trajectories based on domain constraints or improbable regime combinations.
* **Combination with quantiles** – use scenarios alongside quantile forecasts for a complete view; scenarios show paths, while quantiles summarise distributions.

Scenario generation transforms the abstract notion of uncertainty into tangible futures, enabling planners to make robust decisions.