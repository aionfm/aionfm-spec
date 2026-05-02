# Document 53 – AionFM Directory Layout & Build System

## Purpose

This document defines the recommended directory structure and build configuration for the Rust implementation of AionFM.  A well‑structured repository facilitates maintainability, modularity and collaboration.  The build system should support multiple crates, feature flags and continuous integration.

## Audience

Rust developers, build engineers and maintainers of the `aionfm` repository.

## Workspace Structure

Organise the repository as a **Cargo workspace** with separate crates for major components.  A suggested layout is:

```
aionfm/
├── Cargo.toml              # Workspace configuration
├── README.md               # Overview and usage
├── LICENSE
├── crates/
│   ├── data/               # Data layer implementation
│   │   ├── src/
│   │   │   ├── lib.rs
│   │   │   └── ...
│   │   └── Cargo.toml
│   ├── tokenization/       # Tokenization layer (residual descriptors and quantizer)
│   │   ├── src/
│   │   │   └── lib.rs
│   │   └── Cargo.toml
│   ├── model/              # Core model (dual‑stream backbone, heads)
│   │   ├── src/
│   │   │   └── lib.rs
│   │   └── Cargo.toml
│   ├── training/           # Training and loss functions
│   │   ├── src/
│   │   │   └── lib.rs
│   │   └── Cargo.toml
│   ├── adapter/            # Domain adaptation modules
│   │   ├── src/
│   │   │   └── lib.rs
│   │   └── Cargo.toml
│   ├── serving/            # Inference server and API
│   │   ├── src/
│   │   │   ├── main.rs     # Binary for the server
│   │   │   └── ...
│   │   └── Cargo.toml
│   └── utils/              # Common utilities (logging, error handling)
│       ├── src/
│       │   └── lib.rs
│       └── Cargo.toml
└── examples/               # Example usage and demos
    ├── retail_demo/
    ├── energy_demo/
    └── ...
```

Each crate has its own `Cargo.toml` specifying dependencies and features.  The workspace `Cargo.toml` lists all member crates and defines workspace‑wide dependencies.

## Cargo Configuration

The workspace `Cargo.toml` should include:

```toml
[workspace]
members = [
    "crates/data",
    "crates/tokenization",
    "crates/model",
    "crates/training",
    "crates/adapter",
    "crates/serving",
    "crates/utils",
]

[workspace.package]
edition = "2021"

[workspace.dependencies]
anyhow = "1.0"
thiserror = "1.0"
arrow2 = "0.17"
parquet = "0.17"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"

```

Individual crates then specify their own dependencies in their respective `Cargo.toml` files.

## Build Scripts and Code Generation

* Use `build.rs` scripts sparingly for tasks such as embedding schema definitions or generating code from templates.
* For large, auto‑generated code (e.g. attention kernels), place the generator in a separate crate and run it as part of the build process.

## Feature Flags

Define feature flags in `Cargo.toml` to enable optional functionality (e.g. retrieval, synthetic generation, constraint enforcement).  Example:

```toml
[features]
default = ["retrieval", "synthetic"]
retrieval = []
synthetic = []
constraint = []

```
Consumers can then enable or disable features as needed.

## Documentation and README

Provide top‑level and per‑crate `README.md` files explaining purpose, usage, API examples and configuration.  Use Rust doc comments for inline documentation.  Generate docs with `cargo doc --open`.

## Continuous Integration

Configure CI (see Doc55) to run `cargo check`, `cargo test`, `cargo clippy` and `cargo fmt -- --check` across all crates.  Use caching to speed up dependency installation.

## Notes on Third‑Party Libraries

* For tensor operations, consider using `ndarray` or `tch-rs` (Rust bindings for PyTorch).  Evaluate performance trade‑offs.
* For attention and Transformer layers, either implement from first principles or wrap existing implementations (e.g. `burn` or `rusty-bert`) if licensing permits.
