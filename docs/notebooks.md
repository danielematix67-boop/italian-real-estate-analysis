# Notebook Guide

The repository uses focused notebooks rather than one monolithic analysis. The quotation stream is explicitly split into **dataset creation** and **exploration**; transactions and population are maintained as independent analytical streams.

## 01_01 — OMI quotations dataset creation

**File:** `notebooks/01_01_omi_quotations_dataset_creation.ipynb`

**Purpose:** ingestion, validation, cleaning, feature engineering and export of the OMI quotation dataset.

### Main stages

1. setup and project-root discovery;
2. semiannual release discovery;
3. filename and metadata validation;
4. raw dataset inspection;
5. data-quality assessment;
6. structural validation;
7. cleaning and standardisation;
8. feature engineering;
9. temporal and geographic coverage;
10. property categories and conditions;
11. final data-quality summary;
12. Parquet export.

**Primary input:** `data/raw/quotations/`

**Primary output:** processed OMI quotation dataset under `data/processed/`.

The documented run covers 44 quotation releases and 7,516,495 rows before subsequent quality and transformation steps.

## 01_02 — OMI quotations exploration

**File:** `notebooks/01_02_omi_quotations_exploration.ipynb`

**Purpose:** exploratory and comparative analysis of residential OMI quotations using the processed quotation dataset.

### Main analytical levels

- national;
- regional;
- municipality;
- Municipality–Zone.

### Main analytical outputs

- median and mean quotation midpoint;
- observation counts;
- historical quotation trends;
- regional comparisons;
- Top/Bottom Municipality–Zone rankings at selected reference periods;
- semester-over-semester growth;
- year-over-year growth;
- first-to-last overall variation;
- CAGR;
- growth persistence;
- growth volatility;
- maximum growth and decline;
- rolling historical maximum;
- drawdown;
- quotation distribution and dispersion;
- intra-municipality zone spreads.

At Municipality–Zone level, the core aggregation produces `median_compr_mid`, `mean_compr_mid` and `observations` by reference period.

## 02 — OMI transactions exploration

**File:** `notebooks/02_omi_transactions_exploration.ipynb`

**Purpose:** construct and analyse a municipality-year panel of OMI normalized transaction volumes (NTN).

**Coverage:** 2011–2025.

**Input:** `data/raw/transactions/`.

### Main stages

- project-root discovery;
- annual release inventory;
- filename-based release identification;
- validation of the expected OMI table set;
- year-specific schema harmonisation;
- explicit municipality-year key construction;
- join-cardinality validation;
- unmatched municipality-year reporting;
- municipality-year panel construction;
- residential NTN size-class reconciliation where applicable;
- transaction-volume analysis.

The municipality dimension is keyed explicitly by `year` and `codcom`, avoiding assumptions that municipality codes alone are sufficient across annual releases.

## 03 — Population exploration

**File:** `notebooks/03_population_exploration.ipynb`

**Purpose:** construct and analyse a reproducible municipality-year population panel from ISTAT POSAS releases.

**Coverage:** 2019–2026.

**Input:** `data/raw/population/`.

### Main stages

- source inventory;
- annual release validation;
- municipality totals using the official `Età = 999` row;
- demographic indicators;
- geographic analysis;
- analytical hand-off for future integration.

## Recommended execution order

For the OMI quotation stream:

```text
01_01 → 01_02
```

For the independent transaction and population streams:

```text
02
03
```

The transaction and population notebooks do not currently depend on the quotation exploration notebook and can therefore be executed independently.

## Notebook design standard

New notebooks should preserve the project's current pattern:

1. state the analytical objective;
2. define inputs and expected file conventions;
3. validate assumptions before transformation or analysis;
4. keep transformations explicit;
5. use descriptive section headings;
6. explain important derived metrics;
7. surface missing, unmatched or structurally unexpected records;
8. separate source fields from derived indicators;
9. finish with an analytical summary or hand-off dataset where appropriate.
