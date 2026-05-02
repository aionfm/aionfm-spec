## Document 34 ‒ Failure Modes & Mitigation

### Purpose

To describe potential failure modes of AionFM and outline strategies to detect and mitigate them.  Even well‑designed models can fail under certain conditions; understanding these modes helps ensure robust deployment and guides future improvements.

### Intended Audience

Reliability engineers, operations teams and researchers responsible for monitoring and improving AionFM.

### Key Sections

* **Unseen structural breaks** – limitations in predicting unprecedented changes.
* **Poor covariate quality** – impact of inaccurate or missing covariates.
* **Over‑confident uncertainty** – dangers of narrow prediction intervals.
* **Token misclassification** – errors in regime assignment.
* **Scenario implausibility** – generation of unrealistic or invalid scenarios.
* **Mitigation strategies** – detection signals and remedial actions.

### Unseen Structural Breaks

No model can fully predict events outside the training distribution, such as sudden policy changes, catastrophes, major geopolitical events or new market structures.  When such a break occurs, forecasts may be inaccurate and interval coverage poor.  **Mitigation:**

* Monitor external indicators and covariates for early signs of a break.
* Incorporate textual or event data as covariates to anticipate disruptions.
* Trigger recalibration or retraining when significant drift is detected.
* Use stress scenarios to explore the impact of potential breaks on decisions.

### Poor Covariate Quality

If future covariates are inaccurate (e.g. weather forecasts) or missing (e.g. incomplete promotion calendars), forecasts will degrade.  **Mitigation:**

* Validate covariate quality; include uncertainty measures for covariate predictions.
* Use robust models that can operate with missing covariates and rely on historical patterns.
* Postpone certain forecasts until reliable covariate updates are available.

### Over‑Confident Uncertainty

Intervals may be too narrow if the model has not seen similar regimes.  Users may underestimate risk.  **Mitigation:**

* Monitor calibration metrics; adjust quantile loss weights or perform post‑hoc recalibration (e.g. Platt scaling for quantiles).
* Incorporate heavier‑tailed distributions or mixture models to capture extreme events.
* Generate additional synthetic scenarios to expose the model to rare events during training.

### Token Misclassification

The regime tokenizer may assign incorrect behavioural states when history is short or noisy.  Misclassifications can mislead scenario generation and explanations.  **Mitigation:**

* Improve token vocabulary by training on more diverse data or refining quantisation strategies.
* Smooth token sequences using hidden Markov models or CRFs to enforce temporal consistency.
* Provide human feedback to correct misclassifications and update token definitions.

### Scenario Implausibility

Generated scenarios may violate domain constraints (e.g. negative inventory, impossible travel times) or show abrupt jumps.  **Mitigation:**

* Incorporate constraint‑aware decoding to project forecasts onto feasible sets.
* Filter scenarios post‑generation using rule‑based or optimisation‑based checks.
* Enhance the model with additional loss terms penalising implausible behaviour.

### Mitigation Workflow

1. **Detection** – monitor metrics (accuracy, calibration, drift) and interpretability outputs (regime distribution, change‑point probabilities) to detect anomalies.
2. **Diagnosis** – identify which failure mode is likely responsible; examine covariate quality, token assignments, scenario plausibility and external events.
3. **Response** – recalibrate, retrain adapters or the full model, adjust loss weights or perform manual overrides.
4. **Prevention** – incorporate additional data, covariates or synthetic training examples to mitigate future occurrences.

By anticipating failure modes and planning mitigation, AionFM can maintain performance in the face of unexpected challenges.