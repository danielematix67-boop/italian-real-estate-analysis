# Notebook Guide

The project is organised as four analytical streams: quotation preparation and exploration, transaction analysis, population analysis, and an integrated municipality benchmark. Notebooks are designed to run from the repository and resolve data paths relative to the project structure.

## Notebook map

| Notebook | Inputs | Purpose | Main outputs |
|---|---|---|---|
| `01_01_omi_quotations_dataset_creation.ipynb` | `data/raw/quotations/omi_quotations_YYYY_S1.csv` and `S2.csv` | Validate, clean and enrich semiannual OMI quotation releases | `data/processed/omi_quotations.parquet` |
| `01_02_omi_quotations_exploration.ipynb` | Processed quotation Parquet | Explore residential quotation levels and trends at national, regional, provincial and municipal levels, including a Lombardy/Brescia deep dive | Tables and visualisations in the notebook |
| `02_omi_transactions_exploration.ipynb` | Annual OMI transaction tables in `data/raw/transactions/` | Harmonise historical releases, construct a municipality-year panel, and perform descriptive and statistical analysis | `data/processed/omi_transactions_panel.parquet` plus notebook outputs |
| `03_population_exploration.ipynb` | Annual ISTAT POSAS CSVs under `data/raw/population/<year>/` | Build and analyse municipality-year population data and regional age structure | Notebook tables and visualisations |
| `04_omi_comune_peer_benchmark.ipynb` | Processed quotation and transaction Parquet files | Integrate annual municipality-level data and benchmark a target municipality against three peer groups | CSV exports in `data/processed/benchmark/` |

## 01_01 — OMI quotation dataset creation

**Input folder:** `data/raw/quotations/`  
**Output:** `data/processed/omi_quotations.parquet`

The notebook discovers files matching `omi_quotations_*.csv` and reads the semicolon-separated releases. Its main stages are:

1. derive year, semester and reference-period metadata from filenames;
2. inspect schema, duplicate rows and temporal coverage;
3. quantify missingness and inspect technical artefact columns;
4. convert quotation fields to numeric values and expose parsing failures;
5. assess zero values, range consistency, sign constraints and candidate-key duplicates;
6. standardise text and categorical fields conservatively;
7. derive quotation midpoint and spread measures;
8. check temporal, geographic and property-category coverage;
9. produce a final data-quality summary and export the processed dataset.

The derived midpoint `Compr_mid` is an analytical summary of the quoted range, not an observed sale price.

## 01_02 — OMI quotation exploration

**Input:** `data/processed/omi_quotations.parquet`

This notebook performs exploratory analysis of the processed quotation dataset. It examines the schema and missingness, reviews property types, constructs a residential subset, and analyses quotation levels and changes across geographic scales.

Main areas include:

- national quotation trends, using median and mean `Compr_mid`;
- regional quotation levels and comparisons;
- provincial rankings and trend comparisons;
- municipality and sub-municipal comparisons;
- a focused Lombardy and Province of Brescia analysis;
- growth and risk-style descriptive metrics, including changes over time and volatility.

The notebook is an exploration layer; it does not produce the canonical quotation Parquet file. That file is created by Notebook 01_01. Rankings and time-series comparisons should be interpreted in light of observation coverage and differences in the mix of properties and locations.

## 02 — OMI transaction analysis

**Input folder:** `data/raw/transactions/`  
**Output:** `data/processed/omi_transactions_panel.parquet`

The notebook discovers annual files following the `YYYY_<TABLE>.csv` convention and validates the expected set of OMI tables:

- `LISTA-COM`
- `VALORI-RES`
- `VALORI-COM`
- `VALORI-PER`

Historical releases contain schema differences, so the notebook canonicalises field names and maps year-specific NTN fields to a consistent structure. It then:

1. constructs the residential municipality-year panel;
2. reconciles residential size-band NTN totals where possible;
3. analyses national and regional trends;
4. examines the five residential size bands;
5. aggregates non-residential segments and compares them with the reported totals;
6. assembles the final municipality-year analytical panel;
7. runs fail-fast data-quality checks;
8. performs descriptive, distributional and regional analyses, including non-parametric tests and post-hoc comparisons where appropriate;
9. explores national growth, concentration and municipality-level growth dynamics;
10. exports the final panel to Parquet.

The panel uses explicit municipality and year dimensions. Reported reconciliation gaps and coverage differences should be reviewed rather than assumed to be immaterial.

## 03 — ISTAT POSAS population analysis

**Input folder:** `data/raw/population/<year>/`

The notebook discovers year folders and works with the POSAS municipality, province and region files. For municipality totals it uses the official `Età = 999` records rather than summing detailed age rows. It validates uniqueness of municipality-year keys and checks missing or negative population values.

The analysis includes:

- municipality-year population panel and national trend;
- absolute and percentage changes at municipality level;
- population concentration;
- regional and provincial totals;
- regional age composition, grouped into 0–14, 15–64 and 65+.

Population remains an independent analytical stream in the current Notebook 04 implementation. It is not an input to the current peer-group construction.

## 04 — OMI municipality peer benchmark

**Inputs:**
- `data/processed/omi_quotations.parquet`
- `data/processed/omi_transactions_panel.parquet`

**Output folder:** `data/processed/benchmark/`

The notebook integrates the processed quotation and transaction data at municipality-year level. It normalises municipality labels, aggregates semester-level residential quotations to annual municipality statistics, and joins the quotation and transaction data.

### Configuration

The current defaults are:

```python
TARGET_COMUNE = "BRESCIA"
LATEST_YEAR = None
PEER_VOL_TOL = 0.50
MIN_PEERS = 8
TOP_N_PROV = 30
MIN_OBS_FOR_RISK = 3
```

When `LATEST_YEAR` is `None`, the latest available year in the integrated panel is selected.

### Peer groups

- **Provincial:** municipalities in the target's province.
- **Dimensional (region):** municipalities in the target's region whose total NTN volume is within ±50% of the target.
- **Top-N provincial:** the 30 municipalities in the target's province with the highest total NTN volume.

If the dimensional group contains fewer than `MIN_PEERS` municipalities, the notebook falls back to the Top-N provincial group for the relevant comparison.

### Analysis and exports

The notebook compares target values with peer distributions and percentiles, analyses time-series gaps and indexed quotation evolution, examines quotation versus transaction volume and residential share, compares residential transaction composition by size band, and calculates CAGR, annualised volatility and maximum drawdown where the minimum observation requirement is met.

The export layer writes peer-group tables plus risk/return metrics, summary and metadata CSV files to `data/processed/benchmark/`. Filenames are based on the target municipality and peer-group key.

## Execution order

```text
01_01 ──> 01_02
             (quotation exploration)

02 ───────────────┐
                  ├──> 04 (municipality peer benchmark)
01_01 output ─────┘

03 (population analysis; independent)
```

- Run **01_01 before 01_02**.
- Run **02 independently** to create the transaction panel.
- Run **03 independently** for population analysis.
- Run **04 only after both processed quotation and transaction Parquet files exist**.

## Notebook standards

When extending the project, keep source ingestion separate from exploration, validate assumptions before joins and aggregation, make derived metrics explicit, report missing or unmatched records, parameterise sample-dependent benchmark choices, and distinguish descriptive associations from causal conclusions.
