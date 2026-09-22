# Project Documentation

This directory contains the technical documentation for `italian-real-estate-analysis`.

## Documentation map

| Document | Purpose |
|---|---|
| [Data sources](data-sources.md) | Official source datasets, file conventions, identifiers and coverage |
| [Methodology](methodology.md) | Data preparation, aggregation, benchmark construction and analytical metrics |
| [Notebooks](notebooks.md) | Notebook scope, dependencies, inputs, outputs and execution order |
| [Reproducibility](reproducibility.md) | Environment setup, execution controls and data-update workflow |

## Project architecture

The repository is organised as a layered analytical workflow:

```text
Official source releases
        │
        ├── OMI quotations ───────────────┐
        │       │                          │
        │   01_01 ingestion                │
        │       ↓                          │
        │   omi_quotations.parquet         │
        │       ↓                          │
        │   01_02 quotation EDA            │
        │                                  │
        ├── OMI transactions ──────────────┤
        │       │                          │
        │   02 transaction panel           │
        │       ↓                          │
        │   omi_transactions_panel.parquet │
        │                                  │
        └── ISTAT POSAS population ────────┤
                │                          │
            03 population EDA              │
                                           ↓
                              04 peer benchmark
                                           │
                              municipality deep-dive
                                           │
                                  benchmark exports
```

The current repository therefore contains **three source-data analytical streams** and a downstream **municipality benchmarking layer**.

## Analytical data layers

The project distinguishes four conceptual layers:

1. **Raw source data** — original releases from Agenzia delle Entrate/OMI and ISTAT.
2. **Processed analytical data** — cleaned datasets and municipality-year panels.
3. **Derived indicators** — quotation midpoints, growth rates, CAGR, volatility, drawdown, spreads and market-composition measures.
4. **Analytical outputs** — exploratory tables, visualisations and benchmark CSV exports.

The benchmark notebook consumes processed artefacts from upstream notebooks rather than reprocessing raw releases.

## Current analytical streams

### OMI quotations

Notebooks `01_01` and `01_02` cover ingestion, validation, cleaning and exploratory analysis of semiannual residential OMI quotations. Analysis is available at national, regional, municipality and Municipality–Zone levels.

### OMI transactions

Notebook `02` harmonises annual OMI transaction releases for 2011–2025 and constructs a municipality-year panel based on explicit `year + codcom` keys.

### Population

Notebook `03` analyses ISTAT POSAS municipality-level population releases covering 2019–2026. The main municipality total uses the official `Età = 999` row.

### Municipality peer benchmark

Notebook `04` performs a municipality-level deep dive and peer benchmark. The current configuration uses **Brescia** as the target municipality and automatically uses the latest available reference year unless overridden.

The benchmark compares the target with three peer definitions:

- **Provincial** — municipalities in the same province;
- **Dimensional (region)** — municipalities in the same region with total transaction volume within ±50% of the target;
- **Top-N provincial** — the top 30 municipalities in the province by total transaction volume.

The notebook compares quotations, residential and non-residential NTN, total transaction volume, quotation spreads, time-series behaviour, market composition and risk/return-style descriptive metrics.

## Integration status

The project now supports a downstream integration of quotation and transaction data at the municipality-year level for benchmarking.

Population remains a separate source stream in the current benchmark implementation. The benchmark defines market size using total transaction volume because municipal population is not currently required for its peer-group construction.

A future integrated panel can incorporate population through an explicit municipality-code crosswalk and a validated `year + municipality` join.

## Documentation principles

The documentation distinguishes between:

- **source fields** — variables supplied by official providers;
- **derived fields** — variables created during cleaning or feature engineering;
- **analytical metrics** — statistics calculated for comparison;
- **interpretation** — descriptive conclusions supported by the available data.

OMI quotations are reference values and should not be interpreted as transaction-level realised sale prices. Benchmark percentiles describe the target's position within a defined peer sample; they are not causal estimates or forecasts.

## Processed benchmark outputs

Notebook `04` writes municipality benchmark artefacts to:

```text
data/processed/benchmark/
```

For the current Brescia configuration, outputs include:

- provincial benchmark;
- dimensional-region benchmark;
- top-30 provincial benchmark;
- risk/return summary;
- overall benchmark summary;
- metadata describing target, reference year and peer group.

