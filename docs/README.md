# Project Documentation

This directory contains the technical documentation for `italian-real-estate-analysis`.

## Documentation map

| Document | Purpose |
|---|---|
| [Data sources](data-sources.md) | Source datasets, release conventions and coverage |
| [Methodology](methodology.md) | Data preparation, aggregation and analytical metrics |
| [Notebooks](notebooks.md) | Scope, inputs, outputs and execution order for each notebook |
| [Reproducibility](reproducibility.md) | Environment setup and execution guidance |

## Project architecture

The repository is organised around three analytical domains:

```text
                    Italian Real Estate Analysis
                              │
             ┌────────────────┼────────────────┐
             │                │                │
       OMI Quotations   OMI Transactions   Population
             │                │                │
       Dataset creation   Harmonisation      Validation
             │                │                │
       Exploratory EDA    Municipality       Municipality
             │              panel             panel
             └────────────────┼────────────────┘
                              │
                       Comparative analysis
```

The current design keeps ingestion/data-quality work separate from exploratory analysis. This makes the quotation pipeline reproducible and allows the analytical notebooks to operate on a stable processed dataset.

## Documentation principles

The documentation distinguishes between:

- **source data** — fields and files supplied by the official providers;
- **derived data** — fields created during cleaning or feature engineering;
- **analytical metrics** — statistics calculated for comparison and interpretation;
- **interpretation** — conclusions that should be supported by the underlying data and methodology.

No transaction-level realised-price interpretation should be inferred from OMI quotation values alone.
