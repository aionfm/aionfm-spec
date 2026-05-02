# Document 50 – Corpus Segmentation & Splits

## Purpose

This document specifies how to segment the AionFM corpus into training, validation and test sets.  Proper splitting is critical for fair evaluation, avoiding information leakage and assessing generalization across domains and regimes.  Segmentation guidelines also support reproducibility and ensure balanced representation of behaviours.

## Audience

Data engineers and researchers preparing the corpus for training and evaluation.

## Splitting Principles

1. **Temporal integrity:** ensure that the validation and test sets contain data that occur strictly after the training set in time.  Avoid using future information to train the model.

2. **Entity stratification:** when the corpus contains multiple entities (e.g. stores, sensors), split by entity to evaluate cross‑entity generalization.  Alternatively, hold out entire entities for zero‑shot tests.

3. **Domain stratification:** maintain a similar distribution of domains (energy, retail, etc.) across splits.  Reserve some domains entirely for out‑of‑domain evaluation.

4. **Regime diversity:** ensure that the validation and test sets contain examples of different regimes and rare events.  Stratify by regime tokens or annotated labels to avoid skewed evaluation.

5. **Multiple splits:** create multiple train/validation/test partitions to reduce variance in evaluation.  For example, use three non‑overlapping test periods (rolling window evaluation) and average metrics.

## Segmentation Procedure

1. **Define minimum context and horizon:** specify the minimum historical context (e.g. 12 months) and forecast horizon (e.g. 30 days).  Ensure that each split contains at least one full context + horizon window.
2. **Assign entities to splits:** randomly assign entities to training, validation and test sets while respecting domain stratification.  For cold‑start evaluation, hold out a subset of entities entirely from training.
3. **Temporal cut‑off:** for each entity, determine a temporal cut‑off date.  Data before the cut‑off goes to training, data within a window after the cut‑off goes to validation and the remainder goes to test.  Example: use the first 80% of each entity’s timeline for training, the next 10% for validation, and the last 10% for testing.
4. **Regime balancing:** adjust assignments to balance regimes across splits.  Use annotated labels or approximate regime detection to ensure that shocks, peaks and anomalies are not all confined to one split.
5. **Metadata recording:** record split membership in metadata files to ensure reproducibility.  Document the splitting criteria and random seed.

## Handling Updates

* When the corpus is updated with new data, update splits by appending new data to the end of the training and validation sets while keeping the test set fixed for comparability.
* For domains with continuous streaming data, implement sliding window evaluation: train on data up to time `t`, validate on `[t, t+Δ]`, test on `[t+Δ, t+2Δ]`, then advance `t` and repeat.

## Avoiding Leakage

* **Feature leakage:** ensure that covariates or derived features for future periods are not used in training (e.g. using future weather forecasts in past training segments).
* **Entity leakage:** avoid the same entity’s data in both training and test sets unless cross‑entity leakage is intended (e.g. fine‑tuning evaluation).  Use unique entity IDs to enforce separation.
