# Document 60 – SDK Architecture & Design

## Purpose

This document provides design guidelines for a Rust SDK that wraps the AionFM API.  The SDK simplifies integration with applications by providing typed functions for forecasting, scenario sampling and model management.  It also handles network communication, authentication and error handling.

## Audience

SDK developers and application developers using the `aionfm-sdk` repository.

## SDK Goals

* **Ease of use:** provide high‑level functions that abstract away raw HTTP requests.  Users should be able to call `sdk.forecast(...)` without constructing JSON payloads manually.
* **Type safety:** use Rust structs and enums to represent requests and responses.  This enables compile‑time checks and autocompletion.
* **Asynchronous support:** allow asynchronous API calls using `async/await`, supporting concurrency and non‑blocking I/O.
* **Configurability:** allow users to specify API endpoints, timeouts, authentication tokens and logging preferences.
* **Error handling:** return descriptive error types that encapsulate network errors, deserialization failures and API errors.

## Architecture

1. **HTTP client:** use `reqwest` or `hyper` to perform HTTP requests.  Configure connection pooling, timeouts and TLS certificates.
2. **Models and serialization:** define request and response structs matching the API schema (Doc42 and Doc59).  Derive `serde::Serialize` and `serde::Deserialize` for automatic JSON conversion.
3. **API functions:** implement functions for each endpoint (e.g. `forecast`, `generate_scenario`, `interpret`).  Each function should:
   * Accept typed parameters.
   * Construct the request payload.
   * Send the request and await the response.
   * Deserialize the response into typed structs.
   * Map API errors into Rust error types.
4. **Authentication:** support API keys or OAuth tokens.  Provide a configuration struct with an `auth` field.  Automatically attach headers for authentication.
5. **Retry logic:** implement exponential backoff for transient errors (e.g. network timeouts, 5xx responses).  Allow configurable retry policies.
6. **Logging:** use a logging crate (e.g. `tracing`) to log request IDs, endpoints, status codes and timing.  Allow users to set log levels.
7. **Documentation and examples:** provide comprehensive examples demonstrating forecasting, scenario generation and error handling.

## Example API Usage

```rust
use aionfm_sdk::{AionFmClient, ForecastOptions, QuantileLevel};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = AionFmClient::new("https://api.aionfm.com", "your-api-key");
    let response = client.forecast(
        "store_42",                      // entity ID
        vec![123.0, 125.4, ...],         // historical values
        Some(vec![15.0, 14.8, ...]),      // temperature covariate
        ForecastOptions {
            horizon: 30,
            quantiles: vec![QuantileLevel::Q10, QuantileLevel::Q50, QuantileLevel::Q90],
            scenario_count: Some(20),
            return_regimes: true,
        },
    ).await?;
    println!("Median forecast: {:?}", response.quantile_forecasts.get("0.50"));
    Ok(())
}
```

## Error Types

Define a comprehensive error enum:

```rust
#[derive(thiserror::Error, Debug)]
pub enum AionFmError {
    #[error("network error: {0}")]
    Network(#[from] reqwest::Error),
    #[error("serialization error: {0}")]
    Serialization(#[from] serde_json::Error),
    #[error("api error: {status} – {message}")]
    Api { status: u16, message: String },
    #[error("unexpected error: {0}")]
    Unexpected(String),
}
```

Return `Result<T, AionFmError>` from all functions.  Provide convenient helper methods to extract information.

## Feature Flags and Extensibility

* Use feature flags to enable optional endpoints (e.g. retrieval, adaptation) so users can opt in.
* Provide trait abstractions for the HTTP client to allow testing with mock clients.  Users can inject their own HTTP client implementation.

## Testing

* Write unit tests that mock API responses using `wiremock` or `httpmock` crates.
* Test error cases, timeouts and retry logic.  Ensure that deserialization handles unknown fields gracefully.
