# Document 59 – API Architecture & Specification

## Purpose

This document defines the API architecture and specification for the AionFM platform.  The API exposes forecasting, scenario generation, interpretability and administrative capabilities to clients (UI, SDK, third‑party applications).  A well‑designed API enables secure, scalable and extensible integration.

## Audience

Back‑end developers, API designers and maintainers of the `aionfm-api` repository.

## Architectural Overview

The API service is implemented as a stateless microservice that connects to the serving layer (Doc41) to perform inference.  It may be deployed behind a load balancer and scaled horizontally.  Components include:

* **Request handler:** accepts HTTP requests, validates inputs and maps them to internal function calls.
* **Inference gateway:** interfaces with the model serving crate; may queue requests and handle batching for efficiency.
* **Authentication & authorization:** verifies credentials and enforces role‑based access control.  Supports API keys, OAuth2 or JWT.
* **Rate limiter:** throttles requests per user or IP to prevent abuse.
* **Logging & monitoring:** logs requests, responses and errors; exports metrics to monitoring systems.
* **Error handling:** returns standardized error responses with codes and descriptions.

## Endpoints

### 1. `POST /forecast`

* **Description:** Generate forecasts for one or more entities.
* **Request:**
  ```json
  {
    "entities": [
      {
        "entity_id": "store_42",
        "historical_values": [123.0, 125.4, ...],
        "covariates": {
          "temperature": [15.0, 14.8, ...],
          "promotion": [0, 1, ...]
        },
        "metadata": {
          "region": "west",
          "category": "electronics"
        }
      }
    ],
    "horizon": 30,
    "quantiles": [0.1, 0.5, 0.9],
    "scenario_count": 20,
    "options": {
      "return_regimes": true,
      "return_scenarios": true
    }
  }
  ```
* **Response:** JSON object following the schema in Doc42 for each entity.
* **Errors:** 400 Bad Request (invalid data), 401 Unauthorized, 429 Too Many Requests, 500 Internal Server Error.

### 2. `POST /scenario`

* **Description:** Generate scenario paths for specified entities and covariate assumptions.
* **Request:** similar to `/forecast`, but emphasises scenario generation.  May include alternative covariate trajectories.
* **Response:** scenario arrays.

### 3. `POST /interpretation`

* **Description:** Retrieve interpretability outputs (regime probabilities, change‑point likelihoods, explanations) for given entities.
* **Request:** similar to `/forecast` but focusing on interpretability.  May not require scenario generation.
* **Response:** includes regime tokens, probabilities and textual explanations.

### 4. `GET /models`

* **Description:** List available model versions and metadata.  Useful for administrators and version management.
* **Response:** array of model descriptors (name, version, date, description).

### 5. `POST /adapt`

* **Description:** Adapt the model to a new domain or entity using provided data (Doc57).  May trigger adapter training and calibration.
* **Request:** includes entity data, configuration for adapters and calibration parameters.
* **Response:** status message and adaptation identifier.

### 6. `GET /status`

* **Description:** Check API health, model loading status and serving capacity.
* **Response:** JSON with status indicators (healthy/unhealthy), version, latency statistics and queue length.

## API Design Principles

* **Consistency:** use consistent naming conventions and data structures across endpoints.  Use nouns for resources (`forecast`, `scenario`), verbs for actions (`adapt`).
* **Statelessness:** design endpoints to be idempotent where appropriate and avoid storing session state.
* **Pagination:** for listing endpoints (e.g. `/models`), support pagination parameters (`page`, `page_size`).
* **Extensibility:** version the API (e.g. prefix paths with `/v1/`) and design request/response schemas to allow adding fields without breaking clients.
* **Security:** enforce HTTPS; validate inputs rigorously; sanitise error messages to avoid leaking sensitive data.
* **Documentation:** provide OpenAPI/Swagger specifications for auto‑generated client code.  Include example requests and responses.

## Error Codes

| Code | Description |
|------|------------|
| 400 | Invalid request payload, missing required fields or wrong types |
| 401 | Authentication required or invalid credentials |
| 403 | Forbidden – user does not have permission |
| 404 | Resource not found (e.g. unknown model) |
| 429 | Too many requests – rate limit exceeded |
| 500 | Internal server error – unexpected failure |

## Deployment Considerations

* Use an API gateway or reverse proxy (e.g. Nginx, Envoy) to handle SSL termination, routing and load balancing.
* Scale horizontally by running multiple instances of the API service and using a shared model server or replicating the model across instances.
* Integrate with service discovery and health checks (e.g. Kubernetes readiness probes).
