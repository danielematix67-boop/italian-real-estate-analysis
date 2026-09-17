# Project Documentation

This directory contains the technical documentation for `italian-real-estate-analysis`.

## Documentation map

| Document | Purpose |
|---|---|
| [Data sources](data-sources.md) | Official source datasets, file conventions and coverage |
| [Methodology](methodology.md) | Data preparation, aggregation and analytical metrics |
| [Notebooks](notebooks.md) | Notebook scope, inputs, outputs and execution order |
| [Reproducibility](reproducibility.md) | Environment setup, validation controls and execution guidance |

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

The current architecture deliberately separates source ingestion and validation from downstream exploratory analysis. This makes the quotation pipeline reproducible and allows the transaction and population streams to evolve independently.

## Analytical data layers

The project distinguishes four conceptual layers:

1. **Source data** — original releases supplied by Agenzia delle Entrate/OMI and ISTAT.
2. **Processed data** — cleaned and harmonised analytical datasets or panels.
3. **Derived indicators** — measures such as quotation midpoints, growth rates, CAGR, volatility and drawdown.
4. **Exploration outputs** — tables, rankings, distributions and visualisations produced by the notebooks.

This separation is intended to make transformations traceable and prevent analytical outputs from becoming an implicit replacement for the source data.

## Current analytical streams

### OMI quotations

The quotation workflow is split into dataset creation (`01_01`) and exploration (`01_02`). The exploration operates at national, regional, municipality and Municipality–Zone levels.

### OMI transactions

The transaction workflow (`02`) harmonises annual releases from 2011–2025 and constructs a municipality-year panel using explicit year + municipality-code keys and join validation.

### Population

The population workflow (`03`) constructs a municipality-year panel from ISTAT POSAS releases for 2019–2026, using the official `Età = 999` total row for the main municipality population measure.

## Documentation principles

The documentation distinguishes between:

- **source data** — fields and files supplied by official providers;
- **derived data** — fields created during cleaning or feature engineering;
- **analytical metrics** — statistics calculated for comparison and interpretation;
- **interpretation** — conclusions that should be supported by the underlying data and methodology.

OMI quotation values should not be interpreted as transaction-level realised prices. Cross-dataset comparisons should only be performed after explicit alignment of keys, frequency and coverage.
