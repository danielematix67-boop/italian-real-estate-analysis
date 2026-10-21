# Project Documentation

This directory contains the technical guides for `italian-real-estate-analysis`.

## Documentation map

| Document | Purpose |
|---|---|
| [Data sources](data-sources.md) | Official source domains, filename conventions, geographic identifiers and coverage |
| [Methodology](methodology.md) | Cleaning, derived measures, transaction statistics and peer-benchmark methodology |
| [Notebooks](notebooks.md) | Notebook scope, inputs, outputs, parameters and execution order |
| [Reproducibility](reproducibility.md) | Environment setup, data-update steps and reproducibility controls |

## Current architecture

The repository contains three source-data streams and one downstream benchmark:

```text
OMI quotations
  -> 01_01 creation and validation
  -> data/processed/omi_quotations.parquet
  -> 01_02 quotation exploration
                    \
                     -> 04 municipality peer benchmark
                    /
OMI transactions   /
  -> 02 harmonisation and analysis
  -> data/processed/omi_transactions_panel.parquet

ISTAT POSAS
  -> 03 population analysis (independent stream)
```

Notebook 04 currently consumes only the processed quotation and transaction Parquet files. Population analysis is not part of the benchmark inputs.

## Data layers

The project distinguishes:

1. **Raw source releases** in `data/raw/`.
2. **Processed datasets** used as stable hand-offs between notebooks.
3. **Derived indicators**, such as quotation midpoint, transaction totals, growth metrics and historical risk-style statistics.
4. **Analytical outputs**, including notebook visualisations and the benchmark CSV exports in `data/processed/benchmark/`.

## Notebook streams

- **OMI quotations:** Notebook 01_01 prepares and validates the semiannual quotation dataset; Notebook 01_02 explores residential quotation levels and trends at national, regional, provincial and municipal levels.
- **OMI transactions:** Notebook 02 harmonises annual historical releases into a municipality-year NTN panel and performs descriptive and statistical analysis.
- **Population:** Notebook 03 analyses ISTAT POSAS population dynamics and regional age structure independently.
- **Municipality benchmark:** Notebook 04 integrates processed quotations and transactions, then compares a configurable target municipality against provincial, dimensional-region and Top-N provincial peer groups.

The detailed input conventions, configuration parameters and output files are described in [`notebooks.md`](notebooks.md).

## Interpretation principles

The documentation distinguishes source measures from derived variables and descriptive interpretations. OMI quotations are not transaction-level realised sale prices; `Compr_mid` is derived from a quoted range. Benchmark percentiles depend on the peer sample and are not forecasts or causal estimates.

Cross-source integration requires validated geographic and temporal keys. In particular, population is not currently used in the benchmark, and OMI and ISTAT identifiers must not be treated as interchangeable without an explicit crosswalk.
