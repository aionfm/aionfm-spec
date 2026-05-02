## Document 31 ‒ Model Configuration & Sizes

### Purpose

To define the different AionFM model sizes, their intended use cases and key configuration parameters.  Selecting the appropriate model size and tuning configuration parameters is essential for balancing accuracy, resource usage and latency.

### Intended Audience

Machine‑learning engineers and operations teams determining which AionFM variant to deploy and how to configure it.

### Key Sections

* **Model variants** – S, M, L and X models with intended use cases.
* **Configuration parameters** – patch length, context length, forecast horizon, number of variables, vocabulary size, regime levels, transformer depth, attention width, memory compression ratio, quantile levels, scenario count, adapter size.
* **Resource and latency considerations** – guidance on memory footprint and inference time.
* **Customisation** – adjusting parameters for specific domains and tasks.

### Model Variants

| Model    | Intended Use                        | Characteristics                   |
| -------- | ----------------------------------- | --------------------------------- |
| **AionFM‑S** | Edge or low‑latency inference       | Smaller context, fewer parameters, minimal scenario support |
| **AionFM‑M** | General enterprise forecasting      | Balanced accuracy and cost; default configuration for most applications |
| **AionFM‑L** | High‑accuracy batch forecasting     | Longer context, richer scenarios, improved long‑horizon performance |
| **AionFM‑X** | Research and high‑value deployments | Maximum context and capacity, support for complex scenarios and detailed regimes |

Choose a variant based on operational requirements.  For real‑time systems with tight latency budgets, use AionFM‑S or compress AionFM‑M via quantisation or distillation.  For mission‑critical planning tasks where accuracy and uncertainty quantification are paramount, use AionFM‑L or AionFM‑X.

### Configuration Parameters

* **Patch length (P)** – number of time steps per patch.  Longer patches reduce sequence length but may obscure rapid changes.  Typical values: 16, 32, 64.
* **Context length (K)** – number of patches used as input.  Determines how far back the model looks; longer context improves regime detection but increases memory usage.
* **Forecast horizon (H)** – number of future steps to predict.  Should match the longest planning horizon required by the application.  The model decodes one or more patches ahead.
* **Number of variables (V)** – number of target variables (e.g. multivariate time series).  Each variable may have its own embedding and output head.
* **Vocabulary size (Vocab)** – number of discrete regime tokens.  Larger vocabularies capture more nuanced behaviours but require more training data.
* **Regime levels** – number of hierarchical token levels (macro, meso, micro).  More levels increase interpretability granularity but add complexity.
* **Transformer depth (D)** – number of layers in the dual‑stream backbone.  Deeper networks capture more complex dependencies but require more compute.
* **Attention width** – number of heads and dimension of each head.  Adjust to balance capacity and efficiency.
* **Memory compression ratio** – controls how aggressively long‑term memory compresses older context.  Lower ratios retain more historical detail.
* **Quantile levels** – set of quantiles predicted by the quantile head; adjust based on risk appetite.
* **Scenario count** – number of scenarios generated; choose higher counts for risk simulation, lower for interactive dashboards.
* **Adapter size** – dimension of adapter bottleneck layers; controls adapter capacity and adaptation speed.

### Resource and Latency Considerations

Larger models improve accuracy but require more memory and compute.  Benchmark model variants on representative hardware and choose the smallest variant that meets accuracy requirements.  For GPU deployment, ensure that batch sizes and sequence lengths fit GPU memory.  For CPU deployment, use quantisation and parallelisation.  Scenario generation multiplies inference cost by the number of scenarios; allocate resources accordingly.

### Customisation

Configuration parameters can be tuned for specific domains.  For high‑frequency sensor data, use shorter patch lengths and longer contexts.  For domains with long seasonality (e.g. energy, retail), increase context length and memory retention.  Adjust quantile levels to match risk preferences; for example, include extreme quantiles for risk management or omit tails for conservative planning.  Tailor scenario count and adapter size to user needs and computing budget.