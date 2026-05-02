# Document 56 – Performance Benchmarking & Profiling

## Purpose

This document provides guidelines for measuring the performance of AionFM models and associated systems.  Benchmarking and profiling help identify bottlenecks, guide optimization and ensure that the system meets latency, throughput and resource requirements.

## Audience

Machine‑learning engineers, performance engineers and DevOps teams.

## Benchmarking Dimensions

1. **Runtime latency:** measure the time to compute forecasts for a given horizon and batch size.  Include overhead from data pre‑processing and post‑processing.
2. **Throughput:** evaluate how many forecasts per second the system can handle, both in batch and streaming modes.
3. **Memory usage:** track peak and average memory consumption during training and inference.  Identify memory leaks or inefficiencies.
4. **GPU/CPU utilization:** monitor utilisation of compute resources to ensure balanced load and identify under‑used hardware.
5. **Scalability:** assess how performance scales with model size (S, M, L, X), number of variables, horizon length and scenario count.
6. **Calibration metrics:** measure probabilistic calibration quality alongside runtime to capture the trade‑off between speed and accuracy.

## Benchmark Design

* **Synthetic and real data:** use synthetic series (Doc15) for controlled experiments and real datasets for representative workloads.
* **Representative workloads:** test typical batch sizes (e.g. 100, 1,000, 10,000 series), horizon lengths (e.g. 24, 48, 168) and scenario counts (e.g. 0, 10, 100).
* **Hardware variations:** benchmark on different hardware configurations (CPU only, single GPU, multi‑GPU).  Document the environment (CPU model, memory, GPU type, OS).
* **Instrumentation:** insert timers around key components (embedding, attention, output heads) to localise bottlenecks.  Use metrics libraries (e.g. `metrics`, `opentelemetry`) to collect and aggregate data.
* **Repetition:** run each benchmark multiple times to account for variability.  Report mean, median and variance.

## Profiling Tools

* **Rust built‑in tools:** use `cargo bench` with `criterion` for micro‑benchmarks and benchmarking harness.  The `criterion` crate provides statistical analysis and plotting.
* **Flamegraphs:** generate flamegraphs with `cargo flamegraph` to visualise call stacks and hot spots.  This helps identify functions consuming most CPU time.
* **Memory profiling:** use `valgrind`, `perf`, or `heaptrack` to analyse memory allocations and leaks.
* **GPU profiling:** if using `tch-rs` or other GPU backends, use CUDA profilers (e.g. `nvprof`, `Nsight`) to measure kernel performance.

## Reporting

* Summarize benchmarks in tables or plots showing latency vs horizon, throughput vs batch size, memory usage vs model size.
* Compare model variants and configurations.  Highlight trade‑offs between accuracy and speed (e.g. quantile vs scenario generation cost).
* Document environment details to ensure reproducibility.
* Include guidelines for interpreting results and recommended settings for different deployment scenarios (e.g. choose AionFM‑S for real‑time, AionFM‑L for batch scenarios).

## Optimization Tips

* **Vectorization:** leverage Rust’s SIMD capabilities and efficient numeric libraries to speed up core operations (e.g. matrix multiplications).
* **Parallelism:** use multi‑threading (Rayon) or asynchronous programming (Tokio) to parallelize data loading and model execution.
* **Memory reuse:** avoid unnecessary allocations by reusing buffers and employing memory pools.
* **Quantization:** consider quantizing weights and activations to reduce memory and increase throughput, especially for inference.
* **Lazy computation:** compute only the outputs needed for a specific request (e.g. skip scenarios when not requested).
