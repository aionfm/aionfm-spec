# Document 38 – Retrieval‑Augmented Forecasting

## Purpose

This document specifies how AionFM can be extended with a **retrieval module** that searches for analogous historical windows to improve forecasting accuracy and interpretability.  While the core dual‑stream backbone learns a general temporal representation, retrieval provides additional context by drawing on similar patterns from the same or other entities.  Retrieval‑augmented forecasting enables rare‑event prediction, cold‑start adaptation and scenario explanation by grounding forecasts in historical analogs.

## Audience

Researchers, ML engineers and Rust developers implementing the retrieval subsystem.

## Key Concepts

* **Similarity search:** compute a measure of similarity between the current context and candidate windows.  Similarity may be defined on raw values, normalized patches, regime tokens or learned embeddings.
* **Retrieval corpus:** a memory of historical segments, either from the same series (self‑retrieval) or from a library of related series (cross‑series retrieval).  The corpus can be partitioned by domain, entity type or regime class.
* **Integration with backbone:** retrieved windows are encoded into embeddings and fused with the current context through cross‑attention or gating.  Retrieval influences the model’s beliefs about the future without overriding its core forecast.
* **Use cases:** rare regimes, cold starts, anomaly anticipation, interpretability and scenario justification.

## Retrieval Design

1. **Indexing:** define a key for each candidate window.  Keys can be:
   * Averaged continuous patch embeddings from the encoder.
   * Regime‑token sequences summarised with bag‑of‑tokens or n‑gram embeddings.
   * Learned key vectors via an auxiliary network trained to maximize retrieval utility.

2. **Similarity measures:** use cosine similarity, Euclidean distance, dynamic time warping on normalized values, or more advanced metrics (e.g. kernel similarity in latent space).  The choice depends on domain and computational cost.

3. **Retrieval sources:** define pools of candidate windows:
   * **In‑entity:** prior segments of the same series (avoids cross‑series leakage).
   * **Similar entities:** series that share metadata or regime patterns (e.g. stores in the same region).  A k‑nearest‑neighbour search on metadata can identify candidates.
   * **Domain‑wide:** a broad library containing diverse patterns.  Useful for discovering analogs in unrelated domains.

4. **Integration:** once relevant windows are retrieved, their embeddings are fed into the causal backbone.  Possible integration strategies:
   * Concatenate retrieved embeddings with current context and allow attention to weigh them.
   * Compute a retrieval summary vector and add it as an extra token in the value or regime stream.
   * Use gating to modulate the influence of retrieval based on similarity score.

5. **Interpretability:** retrieval results can be surfaced to users as part of the explanation.  For example, the model may note that a sequence resembles a particular past event and show the corresponding retrieved path and outcome.

## Implementation Considerations

* **Efficient search:** for large corpora, use approximate nearest neighbour search (e.g. HNSW, faiss) to retrieve candidates in sub‑linear time.  Pre‑compute keys and keep them updated as new data arrives.
* **Memory management:** maintain separate indexes by domain or entity type to reduce search space.  Purge obsolete data periodically to control memory usage.
* **Rust integration:** implement retrieval as a separate crate that exposes a search API.  Use `parquet`/`arrow` for storing key vectors and `annoy` or `faiss` bindings for retrieval.
* **Privacy:** ensure that retrieval does not leak sensitive data across customers.  Retrieval indexes should be partitioned by tenant and anonymized where necessary.

## Benefits and Limitations

* **Benefits:** retrieval can improve long‑horizon forecasts, support cold‑start entities, provide analog‑based explanations and help anticipate rare regimes.
* **Limitations:** retrieval quality depends on the diversity and relevance of the corpus; similarity measures may require tuning; privacy concerns restrict cross‑customer retrieval; retrieval increases computational complexity.
