# Notebook Guide

The repository uses focused notebooks rather than one monolithic analysis.

## Notebook map

| Notebook | Domain | Main role | Main output |
|---|---|---|---|
| `01_01` | OMI quotations | Ingestion, validation, cleaning and feature engineering | `omi_quotations.parquet` |
| `01_02` | OMI quotations | Exploratory and comparative analysis | Tables and visualisations |
| `02` | OMI transactions | Annual release harmonisation and municipality-year panel | `omi_transactions_panel.parquet` |
| `03` | Population | ISTAT POSAS exploration and municipality-year population analysis | Population analytical artefacts |
| `04` | OMI integrated benchmark | Municipality deep-dive and peer comparison | `data/processed/benchmark/*.csv` |

## 01_01 — OMI quotations dataset creation

**File:** `notebooks/01_01_omi_quotations_dataset_creation.ipynb`

**Purpose:** ingestion, validation, cleaning, feature engineering and export of the OMI quotation dataset.

### Main stages

1. project-root discovery;
2. semiannual release discovery;
3. filename and metadata validation;
4. raw dataset inspection;
5. data-quality assessment;
6. structural validation;
7. cleaning and standardisation;
8. feature engineering;
9. temporal and geographic coverage;
10. property categories and conditions;
11. final quality summary;
12. Parquet export.

**Input:** `data/raw/quotations/`

**Output:** `data/processed/omi_quotations.parquet`

The documented run covers 44 quotation releases and 7,516,495 rows before subsequent quality and transformation steps.

## 01_02 — OMI quotations exploration

**File:** `notebooks/01_02_omi_quotations_exploration.ipynb`

**Purpose:** exploratory and comparative analysis of residential OMI quotations.

### Analytical levels

- national;
- regional;
- municipality;
- Municipality–Zone.

### Main outputs

- median and mean quotation midpoint;
- observation counts;
- historical quotation trends;
- regional comparisons;
- Top/Bottom Municipality–Zone comparisons;
- semester-over-semester growth;
- year-over-year growth;
- first-to-last variation;
- CAGR;
- growth persistence;
- growth volatility;
- maximum growth and decline;
- rolling historical maximum;
- drawdown;
- quotation distributions;
- intra-municipality zone spreads.

The core Municipality–Zone aggregation produces `median_compr_mid`, `mean_compr_mid` and `observations` by reference period.

## 02 — OMI transactions exploration

**File:** `notebooks/02_omi_transactions_exploration.ipynb`

**Purpose:** construct and analyse a municipality-year panel of OMI normalized transaction volumes (NTN).

**Coverage:** 2011–2025.

**Input:** `data/raw/transactions/`

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
- residential NTN size-band reconciliation where applicable;
- transaction-volume analysis;
- processed-panel export.

The municipality dimension is keyed explicitly by `year + codcom`.

## 03 — Population exploration

**File:** `notebooks/03_population_exploration.ipynb`

**Purpose:** construct and analyse a reproducible municipality-year population panel from ISTAT POSAS releases.

**Coverage:** 2019–2026.

**Input:** `data/raw/population/`

### Main stages

- source inventory;
- annual release validation;
- municipality totals using the official `Età = 999` row;
- demographic indicators;
- geographic analysis;
- analytical hand-off.

Population remains a separate source stream from the OMI benchmark.

## 04 — OMI municipality peer benchmark

**File:** `notebooks/04_omi_comune_peer_benchmark.ipynb`

**Purpose:** perform a municipality-level deep dive and benchmark the target against statistically defined peer groups.

**Current target:** `BRESCIA`

**Reference year:** latest available year unless `LATEST_YEAR` is explicitly set.

### Inputs

The notebook consumes processed upstream artefacts:

- `data/processed/omi_quotations.parquet`
- `data/processed/omi_transactions_panel.parquet`

It does not reprocess the raw OMI releases.

### Analytical stages

1. setup and configuration;
2. loading and integration of quotation and transaction artefacts;
3. quotation aggregation to municipality-year level;
4. target municipality selection;
5. peer-group construction;
6. benchmark tables and percentile comparisons;
7. distribution comparisons;
8. target-vs-peer time series;
9. multivariate positioning;
10. residential market composition by size band;
11. CAGR, volatility and maximum drawdown;
12. benchmark export.

### Peer groups

- **Provincial:** same province;
- **Dimensional (region):** same region and total transaction volume within ±50%;
- **Top-30 provincial:** top 30 municipalities in the province by total transaction volume.

If the dimensional group contains fewer than 8 peers, the notebook falls back to the top-30 provincial group.

### Exported artefacts

Outputs are written under:

```text
data/processed/benchmark/
```

For the current target, the export layer includes:

- dimensional-region benchmark;
- provincial benchmark;
- top-30 provincial benchmark;
- risk/return metrics;
- benchmark summary;
- metadata.

## Execution order

### Full quotation-to-benchmark workflow

```text
01_01 → 01_02
           │
02 ────────┼──→ 04
           │
03 ────────┘
```

Notebook `04` currently requires the processed quotation and transaction artefacts. Notebook `03` is an independent demographic stream and is documented separately from the current benchmark inputs.

### Independent execution

- `01_01` must precede `01_02`.
- `02` can be executed independently.
- `03` can be executed independently.
- `04` requires the processed quotation and transaction artefacts produced upstream.

## Notebook design standard

New notebooks should:

1. state the analytical objective;
2. define inputs and expected file conventions;
3. validate assumptions before transformation;
4. keep transformations explicit;
5. use descriptive section headings;
6. explain important derived metrics;
7. surface missing, unmatched or structurally unexpected records;
8. separate source fields from derived indicators;
9. document parameters that affect the sample;
10. finish with a clear analytical summary or export where appropriate.
