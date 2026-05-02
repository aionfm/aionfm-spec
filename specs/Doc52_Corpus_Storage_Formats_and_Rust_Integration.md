# Document 52 – Corpus Storage Formats & Rust Integration

## Purpose

This document specifies the storage formats for the AionFM corpus and provides guidance for loading and streaming data in Rust.  Proper data representation enables efficient training and inference, and seamless integration with the model pipeline.

## Audience

Rust developers and data engineers working on data ingestion and streaming layers.

## Storage Formats

1. **Parquet:** a columnar storage format that compresses well and supports efficient random access.  Suitable for large time‑series corpora.  Use Parquet’s support for nested schemas to store sequences with multiple variables and metadata.

2. **Apache Arrow:** an in‑memory columnar format enabling zero‑copy data sharing between languages.  Use Arrow for inter‑process communication and streaming pipelines.  The `arrow2` crate in Rust provides efficient readers and writers.

3. **Zipped CSV:** for small datasets or interoperability with legacy systems, CSVs can be stored in zipped archives.  Provide explicit column names and types.  Avoid this format for large corpora due to inefficiency.

4. **Metadata catalogs:** maintain a `metadata.json` or `metadata.parquet` file containing dataset descriptions (domain, frequency, start date, end date, entity count) and pointers to data files.  Use this catalog for dataset discovery and retrieval.

## Data Layout

* **Partitioning:** organise files by domain, frequency and year to facilitate distributed reading (e.g. `energy/hourly/2021/*.parquet`).
* **Columns:** include at least `timestamp`, `entity_id`, `variable_name`, `value` and optional `covariate_xxx` columns.  For hierarchical data, include parent identifiers.
* **Schema versioning:** maintain a `schema_version` field in metadata to track changes to column names or types.

## Rust Integration

### Loading Parquet with `parquet` and `arrow2`

Use the `parquet` and `arrow2` crates to read Parquet files into Arrow record batches.

```rust
use arrow2::datatypes::Schema;
use arrow2::io::parquet::read::{infer_schema, read_metadata, FileReader};
use std::fs::File;

fn load_parquet(path: &str) -> Result<Vec<arrow2::record_batch::RecordBatch>, Box<dyn std::error::Error>> {
    let file = File::open(path)?;
    let metadata = read_metadata(&file)?;
    let schema = infer_schema(&metadata)?;
    let mut reader = FileReader::try_new(file, metadata, schema, None, None, None)?;
    let mut batches = Vec::new();
    for maybe_batch in &mut reader {
        let batch = maybe_batch?;
        batches.push(batch);
    }
    Ok(batches)
}
```

### Streaming Data

For streaming large datasets, use asynchronous iterators and memory‑mapped files.  Implement a `DataStream` trait that yields normalised patches ready for the model:

```rust
pub trait DataStream {
    fn next_batch(&mut self) -> Option<TrainingBatch>;
}

pub struct TrainingBatch {
    pub value_patches: Vec<Vec<f32>>,
    pub covariate_patches: Vec<Vec<f32>>,
    pub masks: Vec<Vec<bool>>,
    pub metadata: Vec<EntityMeta>,
}
```

### Lazy Loading and Memory Efficiency

* Use memory‑mapped files (`mmap`) to avoid loading entire files into memory.  Arrow2 supports reading only selected columns.
* For long sequences, stream windows on demand rather than storing full sequences in memory.  Maintain sliding buffers per entity.

## Example: Loading Training Data

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let batches = load_parquet("/data/energy/hourly/2023.parquet")?;
    for batch in batches {
        // Convert to patches and send to training pipeline
    }
    Ok(())
}
```

## Best Practices

* Validate schema consistency when reading multiple files.  Mismatched schemas can lead to runtime errors.
* Use chunked reading to avoid out‑of‑memory issues.  Process data in batches aligned with training mini‑batches.
* Leverage concurrency (e.g. `tokio`) to read and pre‑process data in parallel with model training.
* Ensure thread safety when sharing data structures between threads; use `Arc` and `Mutex` as appropriate.
