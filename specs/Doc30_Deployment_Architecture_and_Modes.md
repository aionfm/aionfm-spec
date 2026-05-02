## Document 30 ‒ Deployment Architecture & Modes

### Purpose

To describe the system architecture and supported deployment modes for AionFM.  Deployment choices depend on operational requirements such as latency, throughput, interactivity and integration with existing systems.  This document outlines batch, real‑time, interactive and embedded deployment options and provides system diagrams and considerations.

### Intended Audience

Infrastructure engineers, solution architects and DevOps practitioners implementing AionFM in production environments.

### Key Sections

* **System overview** – high‑level architecture of AionFM deployment.
* **Batch mode** – scheduled inference for large panels of series.
* **Real‑time mode** – low‑latency streaming forecasts as new data arrives.
* **Interactive scenario engine** – user‑driven scenario exploration and planning.
* **Embedded API service** – exposing AionFM as a service for integration with other applications.
* **Considerations** – scalability, latency, resource allocation, fault tolerance.

### System Overview

The deployment architecture comprises several components:

* **Data ingestion** – pipelines that collect, validate and normalise incoming time series and covariates.  Pre‑processing may happen in streaming or batch fashion.
* **Model serving** – one or more instances of the AionFM core model running on CPUs or GPUs.  Models can be loaded as Docker containers or served via inference engines.
* **Adapter layer** – optional domain‑specific adapters trained for each deployment.  Adapters can be loaded dynamically based on the series metadata.
* **Scenario engine** – module that samples regime paths and decodes scenarios.  Can be invoked on demand by the UI or API.
* **Monitoring and logging** – tracks input quality, forecasting accuracy, calibration and drift.  Integrates with alerting systems.
* **User interface / API** – front‑end dashboards and back‑end endpoints for interacting with forecasts, scenarios and explanations.

### Batch Mode

In batch mode, forecasts are generated at scheduled intervals (e.g. daily or weekly) for large collections of series.  Batch mode suits tasks such as demand planning, capacity planning and monthly financial projections.  Key aspects:

* **Throughput** – the system must handle thousands or millions of series efficiently, leveraging patching and shared computation.  Use vectorised operations and multi‑processing.
* **Scheduling** – run jobs via cron or workflow orchestration systems (Airflow, Prefect).
* **Resource management** – allocate compute resources (GPU or CPU) based on batch size and horizon length.
* **Output storage** – save forecasts and intervals to databases or data lakes for downstream consumption.

### Real‑Time Mode

Real‑time mode serves continuous or near‑continuous forecasts as new observations arrive.  Suitable for monitoring systems, sensor anomaly prediction and intraday demand.  Considerations:

* **Latency** – the model must produce forecasts within strict time budgets (milliseconds to seconds).  Use model quantisation, caching and efficient deployment frameworks (TensorRT, ONNX runtime).
* **Streaming ingestion** – integrate with message queues (Kafka, Pub/Sub) to capture incoming data; update patches incrementally.
* **State management** – maintain model state between calls to preserve context; implement sliding windows for patches and memory.
* **Scalability** – auto‑scale model instances based on traffic.

### Interactive Scenario Engine

An interactive mode allows users to generate scenarios and experiment with what‑if assumptions.  Typically used via a web UI with controls for adjusting covariates (e.g. scheduled promotions, price changes, weather scenarios).  The scenario engine:

* **Accepts user inputs** – modifications to future covariates, promotion schedules or constraints.
* **Samples regimes and decodes values** – returns multiple trajectories with explanations.
* **Visualises uncertainty** – displays quantiles, intervals and scenario paths.
* **Supports planning decisions** – integrates with inventory or capacity planning tools.

### Embedded API Service

AionFM can be exposed as an API that other services can call.  A typical REST endpoint is:

```text
POST /forecast
Input:
  historical_values
  covariates
  metadata
  horizon
  quantiles
  scenario_count

Output:
  point_forecast
  quantile_forecasts
  scenario_paths
  regime_probabilities
  explanation
```

GraphQL or gRPC interfaces can also be defined for more complex queries.  The API service should enforce authentication, rate limiting, schema validation and versioning.

### Considerations

* **Scalability** – design for horizontal scaling; containerise the model and use orchestration platforms (Kubernetes, ECS).
* **Latency vs. accuracy** – smaller model variants (AionFM‑S) can be used for real‑time inference; larger variants (AionFM‑L, AionFM‑X) for batch or offline modes.
* **Fault tolerance** – implement retry logic, backup replicas and monitor system health.
* **Observability** – collect metrics on throughput, latency, error rates, calibration and drift; visualise in dashboards.

Choosing the appropriate deployment mode and architecture enables organisations to integrate AionFM seamlessly into their operations.