# Italian Real Estate Analysis

A reproducible Python project for exploring the Italian residential real-estate market using official data from the **Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI)** and **ISTAT POSAS**. The repository combines quotation analysis, normalized transaction volumes (NTN), population analysis and a municipality-level peer benchmark.

## Project objectives

The project turns raw official releases into validated analytical datasets and uses them to explore:

- how residential quotation levels vary across Italy and evolve over time;
- differences between regions, provinces and municipalities, including a focused Lombardy/Brescia analysis;
- how residential and non-residential transaction volumes change across years and geographic areas;
- the composition of residential transactions by property-size band;
- demographic change and regional age structure;
- how a target municipality compares with local and similarly sized markets.

The analyses are descriptive and exploratory. They do not establish causal relationships or forecast future property prices.

## Data domains

| Domain | Source | Current documented coverage | Main hand-off |
|---|---|---|---|
| OMI quotations | Agenzia delle Entrate / OMI | Semiannual releases; the dataset-creation notebook documents 44 releases in its recorded run | `data/processed/omi_quotations.parquet` |
| OMI transactions | Agenzia delle Entrate / OMI | Annual municipality-level NTN panel, 2011–2025 | `data/processed/omi_transactions_panel.parquet` |
| Population | ISTAT POSAS | Annual releases, 2019–2026, subject to locally available files | Analysed in Notebook 03; currently independent of the benchmark |
| Municipality benchmark | Derived from processed OMI quotation and transaction data | Configurable target and latest available reference year | `data/processed/benchmark/` |

## Repository structure

```text
italian-real-estate-analysis/
├── data/
│   ├── raw/
│   │   ├── quotations/
│   │   ├── transactions/
│   │   └── population/<year>/
│   └── processed/
│       └── benchmark/
├── notebooks/
│   ├── 01_01_omi_quotations_dataset_creation.ipynb
│   ├── 01_02_omi_quotations_exploration.ipynb
│   ├── 02_omi_transactions_exploration.ipynb
│   ├── 03_population_exploration.ipynb
│   └── 04_omi_comune_peer_benchmark.ipynb
├── docs/
│   ├── README.md
│   ├── data-sources.md
│   ├── methodology.md
│   ├── notebooks.md
│   └── reproducibility.md
├── requirements.txt
├── .gitignore
└── README.md
```

Raw files are kept separate from processed analytical artefacts. Some raw and generated data may be available only in the local working copy, depending on repository tracking and source-release availability.

## Analytical workflow

```text
OMI quotations
    |
    v
01_01 Dataset creation, validation and cleaning
    |
    v
data/processed/omi_quotations.parquet
    |                         \
    v                          \
01_02 Quotation exploration     \
                                v
OMI transactions ----------> 04 Municipality peer benchmark
    |                           ^
    v                           |
02 Harmonisation                |
    |                           |
    v                           |
data/processed/omi_transactions_panel.parquet

ISTAT POSAS -> 03 Population analysis (independent stream)
```

Notebook 04 requires both processed OMI Parquet files. Notebook 03 analyses population independently and is not currently an input to the benchmark.

## Notebooks

### 01_01 — OMI quotation dataset creation

Reads semiannual, semicolon-separated files matching `omi_quotations_YYYY_S1.csv` and `omi_quotations_YYYY_S2.csv`. It derives reference-period metadata, inspects schema and coverage, measures missingness, validates quotation ranges and candidate keys, standardises data, derives analytical fields and exports:

```text
data/processed/omi_quotations.parquet
```

### 01_02 — OMI quotation exploration

Uses the processed quotation dataset to explore residential quotation levels and trends at national, regional, provincial and municipal levels. It includes a Lombardy and Province of Brescia deep dive, geographic comparisons, and descriptive growth/risk-style analysis. The notebook is exploratory and does not create the upstream Parquet hand-off.

### 02 — OMI transaction analysis

Discovers annual releases in `data/raw/transactions/`, validates the expected `LISTA-COM`, `VALORI-RES`, `VALORI-COM` and `VALORI-PER` table set, harmonises historical schema differences and constructs a municipality-year panel. It analyses residential NTN by size band, non-residential NTN, national and regional trends, distributional differences, market concentration and municipality-level growth. The output is:

```text
data/processed/omi_transactions_panel.parquet
```

### 03 — ISTAT POSAS population analysis

Reads annual source files from `data/raw/population/<year>/`. Municipality totals use the official `Età = 999` row. The notebook examines national and local population dynamics, concentration, regional and provincial totals, and regional age structure. This stream is independent of the current benchmark.

### 04 — OMI municipality peer benchmark

Integrates processed quotation and transaction data at municipality-year level and compares a configurable target (currently `BRESCIA`) with three peer groups:

- **Provincial:** municipalities in the same province.
- **Dimensional (region):** municipalities in the same region with total NTN within ±50% of the target.
- **Top-30 provincial:** the 30 highest-volume municipalities in the target's province.

The latest available year is selected by default. If the dimensional group contains fewer than eight peers, the notebook falls back to the Top-30 provincial group for that comparison. It analyses quotation and transaction levels, peer percentiles, time-series comparisons, residential market composition and historical CAGR, volatility and drawdown where enough observations exist. CSV outputs are written to `data/processed/benchmark/`.

See [`docs/notebooks.md`](docs/notebooks.md) for the detailed notebook map and [`docs/methodology.md`](docs/methodology.md) for metric definitions and benchmark construction.

## Setup and execution

Create a virtual environment and install the dependencies:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
jupyter lab
```

On Windows PowerShell, activate with:

```powershell
.venv\Scripts\Activate.ps1
```

Run notebooks from the repository and execute cells from top to bottom.

Recommended sequence:

1. Run Notebook 01_01 to create the processed quotation dataset.
2. Run Notebook 01_02 to explore quotation data.
3. Run Notebook 02 to create the transaction panel.
4. Run Notebook 03 independently for population analysis.
5. Run Notebook 04 after both processed quotation and transaction files exist.

Detailed instructions and source-file conventions are documented in [`docs/reproducibility.md`](docs/reproducibility.md) and [`docs/data-sources.md`](docs/data-sources.md).

## Technology stack

- Python
- pandas and NumPy
- SciPy, statsmodels and scikit-posthocs
- Matplotlib and Seaborn
- JupyterLab
- PyArrow for Parquet I/O
- Git and GitHub

Dependencies are listed in `requirements.txt`. They are not all version-pinned, so exact environment replication may require recording the package versions used.

## Data quality and interpretation

The workflow checks file conventions, schema differences, missingness, duplicates, geographic and temporal coverage, join cardinality, municipality-year uniqueness, non-negative transaction values and transaction reconciliation where applicable.

Important limitations:

- OMI quotations are reference ranges, not realised sale prices.
- `Compr_mid` is a derived midpoint, not a transaction price.
- Quotation and transaction data have different frequencies and coverage.
- Municipality identifiers differ across OMI and ISTAT; integration requires validated geographic keys.
- Benchmark results depend on the target, reference year, peer-group definitions and data availability.
- Percentiles, growth rates, volatility and drawdown are descriptive measures, not forecasts or causal estimates.

## Documentation

- [Project documentation index](docs/README.md)
- [Data sources and identifiers](docs/data-sources.md)
- [Methodology and metrics](docs/methodology.md)
- [Notebook guide and execution order](docs/notebooks.md)
- [Reproducibility and data updates](docs/reproducibility.md)

## License

See [`LICENSE`](LICENSE) for the applicable license terms.
