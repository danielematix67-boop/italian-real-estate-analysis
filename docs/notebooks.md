# Notebook Guide

The repository uses focused notebooks rather than one monolithic analysis. The first notebook in the quotation stream creates and validates the analytical dataset; the second consumes that dataset for exploration.

## 01_01 — OMI quotations dataset creation

**File:** `notebooks/01_01_omi_quotations_dataset_creation.ipynb`

**Purpose:** ingestion, validation, cleaning, feature engineering and export of the OMI quotation dataset.

### Main stages

1. setup and project-root discovery;
2. semiannual file discovery;
3. raw dataset inspection;
4. data-quality assessment;
5. structural validation;
6. cleaning and standardisation;
7. feature engineering;
8. temporal coverage;
9. geographic coverage;
10. property categories and conditions;
11. final data-quality summary;
12. Parquet export.

**Primary input:** `data/raw/quotations/`

**Primary output:** processed OMI quotation dataset under `data/processed/`.

## 01_02 — OMI quotations exploration

**File:** `notebooks/01_02_omi_quotations_exploration.ipynb`

**Purpose:** exploratory and comparative analysis of residential OMI quotations.

### Main analytical levels

- national;
- regional;
- municipality;
- Municipality–Zone.

### Main metrics

- median quotation midpoint;
- mean quotation midpoint;
- observation count;
- semester-over-semester growth;
- year-over-year growth;
- overall price variation;
- CAGR;
- growth volatility;
- positive-growth persistence;
- maximum semester growth and decline;
- rolling historical maximum;
- drawdown.

The Municipality–Zone level is used when analysing spatial heterogeneity inside municipalities and across Italy.

## 02 — OMI transactions exploration

**File:** `notebooks/02_omi_transactions_exploration.ipynb`

**Purpose:** construct and analyse a municipality-year panel of OMI normalized transaction volumes (NTN).

### Main stages

- project-root discovery;
- annual release inventory;
- filename validation;
- schema harmonisation;
- key validation before joins;
- municipality-year panel construction;
- transaction-volume analysis.

**Coverage:** 2011–2025 in the current notebook.

**Input:** `data/raw/transactions/`.

## 03 — Population exploration

**File:** `notebooks/03_population_exploration.ipynb`

**Purpose:** construct a reproducible municipality-year population panel from ISTAT POSAS releases.

### Main stages

- source inventory;
- annual release validation;
- municipality totals;
- demographic indicators;
- geographic analysis;
- analytical hand-off.

**Coverage:** 2019–2026 in the current notebook.

**Input:** `data/raw/population/`.

## Recommended execution order

For the OMI quotation stream:

```text
01_01 → 01_02
```

For the other analytical domains:

```text
02
03
```

The transaction and population notebooks are currently independent analytical streams. They can therefore be run separately from the quotation notebooks.

## Notebook design standard

New notebooks should preserve the project's current pattern:

1. document the analytical objective;
2. define inputs and expected file conventions;
3. validate assumptions before analysis;
4. keep transformations explicit;
5. use descriptive section headings;
6. explain important derived metrics;
7. finish with an analytical summary or hand-off dataset where appropriate.
