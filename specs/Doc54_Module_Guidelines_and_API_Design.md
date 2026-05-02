# Document 54 – Module Guidelines & API Design

## Purpose

This document provides guidelines for structuring modules, traits and APIs within the `aionfm` Rust codebase.  A clear module and API design promotes code reuse, testability and ease of extension.  It also defines how external consumers will interact with the model through a stable API.

## Audience

Rust developers designing the internal modules and the external API.

## Module Structure

Follow Rust’s module system conventions:

* Each crate defines its **public API** in `lib.rs` or `main.rs` and private implementation details in submodules.
* Use `mod` declarations to structure code into logical units (e.g. `encoder`, `tokenizer`, `attention`, `decoder`).
* Expose only necessary types, traits and functions with `pub`.  Keep helper functions private to reduce API surface.
* Use nested modules sparingly; prefer flat hierarchies for readability.

Example:

```rust
pub mod encoder {
    pub struct PatchEncoder { /* fields */ }
    impl PatchEncoder { /* methods */ }
}

mod attention {
    pub struct CausalAttention { /* fields */ }
    // Internal helper functions hidden from the crate API
    fn compute_scores(/* ... */) {}
}

pub use encoder::PatchEncoder;
```

## Traits and Generics

* Define **traits** to abstract over common behaviour (e.g. `Embedding`, `Tokenizer`, `ForecastModel`).  This enables swapping implementations and mocking in tests.
* Use **generic parameters** to support different numeric types or tensor backends.  For example, `PatchEncoder<T: Float>`.
* Avoid overly generic APIs that complicate type inference.  Provide sensible defaults via wrapper types.
* Document trait methods thoroughly to clarify expected inputs, outputs and side effects.

## Error Handling

* Use `Result<T, E>` for functions that can fail.  Define custom error enums with `thiserror` or `anyhow`.
* Prefer explicit error propagation (`?`) over panicking.  Reserve `panic!` for unrecoverable programmer errors.
* Document error types in API docs and provide meaningful error messages.

## Stable External API

Expose a `pub` API for external consumers (SDKs, UI, CLI).  Examples:

```rust
pub struct ForecastRequest {
    pub entity_id: String,
    pub historical_values: Vec<f32>,
    pub covariates: Option<Vec<f32>>,
    pub horizon: usize,
    pub quantiles: Vec<f32>,
    pub scenario_count: Option<usize>,
}

pub struct ForecastResponse {
    pub point_forecast: Vec<f32>,
    pub quantile_forecasts: HashMap<String, Vec<f32>>,
    pub scenarios: Option<Vec<Vec<f32>>>,
    pub regime_probs: Option<HashMap<String, f32>>,
    pub explanation: Option<Explanation>,
}

pub trait ForecastService {
    fn forecast(&self, request: ForecastRequest) -> Result<ForecastResponse, ForecastError>;
}
```

This trait can be implemented by different backends (in‑memory model, remote API proxy).  It allows SDKs to depend on a single trait rather than the full implementation.

## Documentation and Versioning

* Document each public item with Rustdoc comments.  Include examples where applicable.
* Follow semantic versioning.  Increment the major version for breaking changes, minor for new features and patch for bug fixes.
* Use feature flags to deprecate old APIs while introducing new ones.

## Testing Interfaces

* Provide mock implementations of traits to enable unit tests without running the full model (e.g. a `MockForecastService` that returns fixed responses).
* Use `#[cfg(test)]` modules for test helpers and sample data.
* Write integration tests that exercise the public API with realistic data flows.
