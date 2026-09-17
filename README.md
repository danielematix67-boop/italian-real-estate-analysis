# Italian Real Estate Analysis

Quantitative analysis of the Italian residential real estate market using official OMI (Osservatorio del Mercato Immobiliare) data, complemented by transaction-volume and population datasets.

The project is designed as a reproducible data-analysis workflow: raw releases are inventoried and validated, transformed into analysis-ready panels, and explored through focused Jupyter notebooks.

## Project scope

The repository currently covers three analytical streams:

- **OMI quotations** — semiannual residential property quotations, with temporal, geographic and property-level analysis from the historical OMI releases available in the repository.
- **OMI transactions** — normalized transaction volumes (NTN) at municipality level, with annual releases covering 2011–2025 and a harmonized municipality-year panel.
- **Population** — municipality-year population data based on ISTAT POSAS releases, currently covering 2019–2026.

The analytical hierarchy is designed to move from national and regional patterns to province, municipality and OMI-zone comparisons where the underlying data support that level of detail.

## Main analytical questions

The project is intended to answer questions such as:

- How have residential OMI quotations evolved over time?
- How do price levels and growth differ across regions, municipalities and OMI zones?
- Which Municipality–Zone combinations experienced the largest long-run changes?
- How persistent and volatile has price growth been?
- How large are historical drawdowns from local quotation peaks?
- How dispersed are residential quotation levels across Italian OMI zones?
- How do transaction volumes evolve across municipalities?
- How can quotation, transaction and demographic datasets be prepared for future integrated analysis?

## Repository structure

```text
italian-real-estate-analysis/
├── data/
│   ├── raw/
│   │   ├── population/
│   │   ├── quotations/
│   │   └── transactions/
│   └── processed/
├── notebooks/
│   ├── 01_01_omi_quotations_dataset_creation.ipynb
│   ├── 01_02_omi_quotations_exploration.ipynb
│   ├── 02_omi_transactions_exploration.ipynb
│   └── 03_population_exploration.ipynb
├── requirements.txt
├── LICENSE
├── .gitignore
└── README.md
```

## Analytical workflow

```text
Official data releases
        │
        ├── OMI quotations
        │       ↓
        │   ingestion & validation
        │       ↓
        │   analysis-ready quotation dataset
        │       ↓
        │   temporal / geographic / zone analysis
        │
        ├── OMI transactions
        │       ↓
        │   release discovery & schema harmonisation
        │       ↓
        │   municipality-year transaction panel
        │       ↓
        │   NTN and price-volume analysis
        │
        └── ISTAT POSAS population
                ↓
            inventory & validation
                ↓
            municipality-year population panel
                ↓
            demographic analysis
```

## OMI quotations pipeline

The quotation workflow is intentionally split into two notebooks.

### `01_01_omi_quotations_dataset_creation.ipynb`

Creates the analytical quotation dataset from the semiannual OMI releases. The notebook:

1. discovers files using the expected `omi_quotations_YYYY_S1.csv` / `S2` convention;
2. records year and semester metadata;
3. inspects the raw schema and temporal coverage;
4. performs data-quality and structural validation;
5. applies conservative cleaning and standardisation;
6. engineers analytical fields, including quotation midpoints where appropriate;
7. checks geographic and property-category coverage;
8. exports the processed dataset to Parquet.

The current notebook documents 44 quotation files and 7,516,495 consolidated rows in its recorded run.

### `01_02_omi_quotations_exploration.ipynb`

Uses the processed quotation dataset for exploratory and comparative analysis. The current analysis includes national, regional, municipal and Municipality–Zone views, including:

- median and mean purchase quotations;
- observation counts;
- historical quotation trends;
- top and bottom Municipality–Zone comparisons;
- semester-over-semester growth;
- year-over-year growth;
- overall price variation;
- CAGR and growth persistence;
- growth volatility;
- historical drawdown analysis;
- price-distribution and dispersion analysis;
- intra-municipality differences between OMI zones.

## OMI transactions

`02_omi_transactions_exploration.ipynb` analyses normalized transaction volumes (NTN) using annual OMI releases from 2011 to 2025.

The notebook is release-agnostic: it discovers files from their filenames, validates the expected OMI tables, harmonises year-specific schemas and constructs a municipality-year analytical panel before downstream analysis.

Expected annual tables include:

- `LISTA-COM`
- `VALORI-RES`
- `VALORI-COM`
- `VALORI-PER`

## Population

`03_population_exploration.ipynb` builds a reproducible municipality-year population panel from ISTAT POSAS releases.

The workflow includes source inventory, validation, municipality totals, demographic indicators, geographic analysis and an analytical hand-off for potential integration with real-estate indicators.

The municipality-level panel uses the official `Età = 999` total row rather than aggregating the age-detail records unnecessarily.

## Data sources

Primary data sources are official Italian public-data releases:

- **Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI)** for property quotations and transaction data.
- **ISTAT — POSAS** for population data.

See [`docs/data-sources.md`](docs/data-sources.md) for source conventions, coverage and dataset-specific notes.

## Methodology

The project follows a conservative, auditable approach to data preparation:

- raw data are inspected before transformation;
- filename conventions are validated rather than assumed;
- transformations are measured where practical;
- quotation values are not silently imputed;
- geographic and temporal keys are checked before panel construction;
- derived metrics are kept separate from raw-source fields;
- analytical notebooks document both the methodology and the resulting outputs.

See [`docs/methodology.md`](docs/methodology.md) for the detailed analytical conventions.

## Reproducibility

Create a virtual environment and install the project dependencies:

```bash
python -m venv .venv
```

Activate it on Linux/macOS:

```bash
source .venv/bin/activate
```

Or on Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Launch JupyterLab:

```bash
jupyter lab
```

Run the notebooks in the documented order when reproducing the quotation workflow:

1. `01_01_omi_quotations_dataset_creation.ipynb`
2. `01_02_omi_quotations_exploration.ipynb`
3. `02_omi_transactions_exploration.ipynb`
4. `03_population_exploration.ipynb`

The notebooks resolve the project root from the expected `data/` directories, reducing dependence on a hard-coded local path.

## Dependencies

The current environment is defined in `requirements.txt` and includes:

- pandas
- NumPy
- Matplotlib
- Seaborn
- JupyterLab
- OpenPyXL
- PyArrow
- statsmodels

## Data management

Raw source data are organised under `data/raw/` by analytical domain. Processed analytical artefacts are generated under `data/processed/` when produced by the notebooks.

Local environments, notebook checkpoints, build artefacts and other temporary files are excluded through `.gitignore`.

## Limitations and interpretation

OMI quotations are market reference values rather than transaction-level observed sale prices. They should therefore be interpreted as an official quotation indicator and not as a direct estimate of realised transaction prices.

Municipality–Zone comparisons also require attention to temporal coverage and data availability. A zone with a shorter or incomplete history should not automatically be interpreted as directly comparable with a zone observed over the full period.

Transaction volumes and population measures are maintained as separate analytical streams in the current repository. Integration across these datasets is a future analytical step rather than an assumption embedded in the existing notebooks.

## Documentation

- [`docs/README.md`](docs/README.md) — documentation index and project guide
- [`docs/data-sources.md`](docs/data-sources.md) — source datasets and coverage
- [`docs/methodology.md`](docs/methodology.md) — analytical methodology and metrics
- [`docs/notebooks.md`](docs/notebooks.md) — notebook-by-notebook documentation
- [`docs/reproducibility.md`](docs/reproducibility.md) — environment and execution guide

## License

See [`LICENSE`](LICENSE) for the repository license terms.
