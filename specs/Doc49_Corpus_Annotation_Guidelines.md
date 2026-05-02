# Document 49 – Corpus Annotation Guidelines

## Purpose

This document provides instructions for annotating the AionFM corpus to support evaluation and interpretability research.  Annotated data helps evaluate regime detection, change‑point detection and other interpretability outputs.  Clear guidelines ensure consistent, high‑quality annotations across annotators.

## Audience

Annotation teams, data scientists and researchers.

## Annotation Tasks

1. **Regime labeling:** assign a regime label to each segment indicating the dominant temporal behaviour, such as stable growth, seasonal peak, shock, recovery, volatility expansion, volatility compression, plateau, mean reversion or anomaly.
2. **Seasonality phase:** identify the phase of recurring seasonal cycles (e.g. start, peak, end) for time series with known seasonality (weekly, monthly, yearly).  Annotate the start and end of each cycle.
3. **Shock events:** mark timestamps where a sudden change occurs (positive or negative shock) and optionally annotate the type (external event, promotion, sensor failure).  Record magnitude and duration of impact when possible.
4. **Change points:** annotate points where the statistical properties of the series change significantly (e.g. level shift, trend reversal, volatility shift).  Use guidelines to determine significance thresholds.
5. **Anomalies:** label anomalous segments or points that deviate strongly from expected patterns.  Classify anomalies as transient spikes, sustained outliers or missing bursts.

## Labeling Schema

* Use a pre‑defined list of regime categories.  Provide definitions and examples for each category to ensure consistency.
* Create a hierarchical schema when possible (e.g. macro: seasonal vs non‑seasonal; meso: trend direction; micro: residual noise pattern).
* Support multi‑label annotations when multiple behaviours overlap (e.g. seasonal peak with high volatility).
* Use time ranges for events rather than single points when events have duration.

## Annotation Process

1. **Training annotators:** provide a training session with examples for each label, including borderline cases.  Discuss the rationale behind each category.
2. **Tools:** use annotation tools that support time‑series visualization, zooming, and labeling of segments.  Tools should allow overlaying raw values, smoothed trends and derivative signals.
3. **Quality control:** have multiple annotators label the same segments.  Compute inter‑annotator agreement (e.g. Cohen’s kappa) and resolve disagreements through discussion or adjudication.
4. **Label storage:** store annotations in a structured format (e.g. JSON, CSV) aligned with the original time stamps.  Include annotator ID and confidence scores.
5. **Feedback loop:** regularly review annotations and refine guidelines.  Provide annotators with feedback on errors and ambiguities.

## Use of Annotations

* **Evaluation:** annotated regimes and change points allow objective evaluation of the model’s interpretability outputs (Doc21).  Metrics include precision, recall and F1 scores for regime prediction and change‑point detection.
* **Training:** while AionFM is primarily self‑supervised, annotated data can be used to guide token vocabulary learning or fine‑tune the regime decoder for specific domains.
* **Research:** annotations enable research into new regime categories, causal analysis and improved explanation mechanisms.

## Ethical Considerations

* Ensure that annotators are aware of privacy requirements.  Do not annotate personally identifiable or sensitive information.
* Provide fair compensation and respect intellectual property rights when working with proprietary data.
