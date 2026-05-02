# Document 46 – Limitations & Practical Considerations

## Purpose

This document candidly discusses known limitations of AionFM and practical considerations for deployment.  Understanding these constraints helps manage expectations, identify mitigation strategies and guide future improvements.

## Audience

Stakeholders, project managers, developers and evaluators planning to deploy AionFM.

## Known Limitations

1. **Dependence on data quality:** AionFM cannot compensate for extremely noisy, inconsistent or biased data.  Poor covariates or missing metadata can lead to inaccurate forecasts and miscalibrated uncertainty.

2. **Extreme regime shifts:** Unprecedented events (e.g. pandemics, abrupt regulatory changes) may not be captured by the pretraining corpus.  The model may struggle to extrapolate beyond learned patterns.  Synthetic data and retrieval help but are not panaceas.

3. **Long‑horizon uncertainty:** As the forecast horizon increases, uncertainty widens and scenarios become diffuse.  Quantile and scenario outputs may become less informative beyond certain time frames.

4. **Interpretability approximations:** Regime tokens are learned abstractions; they may not correspond exactly to human labels.  Explanations are indicative rather than definitive, and should not be treated as causal statements.

5. **Constraint enforcement complexity:** Enforcing domain‑specific constraints can be computationally intensive.  Projection into feasible spaces may degrade calibration if not handled carefully.

6. **Retrieval privacy risks:** Cross‑entity retrieval, if not properly partitioned, can leak patterns from sensitive series.  Retrieval must respect tenant boundaries and anonymization.

7. **Adapter explosion:** Managing a large number of adapters (e.g. per entity) can increase storage and complexity.  Mechanisms for sharing adapters across similar entities may be necessary.

8. **Hardware and performance:** Large AionFM models (L, X) require significant memory and compute.  Serving scenario paths can be expensive, especially with large horizons and high scenario counts.

9. **Regulatory constraints:** In regulated industries, the model may need additional validation and documentation.  Certain domains may not allow the use of synthetic data or cross‑domain transfer.

## Practical Considerations

* **Data governance:** invest in data quality, lineage and governance to maximize model performance.
* **Calibration monitoring:** implement continuous calibration monitoring to detect and correct drift.
* **Human oversight:** for high‑stakes decisions, maintain human review loops and use the model as decision support rather than automation.
* **Explain results:** provide training for end‑users on interpreting probabilistic forecasts and regimes to avoid misinterpretation.
* **Start small:** pilot the model on a subset of entities or products to validate performance before full rollout.
* **Iterative improvement:** adopt an iterative deployment strategy; incorporate user feedback and new data to refine models and adapters.

## Future Work

Many limitations can be addressed through research directions outlined in Doc43.  Improving retrieval, integrating causal and domain constraints, and developing better calibration techniques are active areas of development.
