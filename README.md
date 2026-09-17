# Italian Real Estate Analysis

Quantitative analysis of the Italian residential real estate market using official **OMI (Osservatorio del Mercato Immobiliare)** data, complemented by **OMI transaction volumes (NTN)** and **ISTAT POSAS population data**.

The repository is structured as a reproducible, notebook-driven analytical workflow: raw releases are inventoried and validated, transformed into analysis-ready datasets or panels, and then explored through focused notebooks.

## Project scope

The project currently covers three analytical streams:

- **OMI quotations** — semiannual residential quotation data with national, regional, municipal and OMI-zone analysis.
- **OMI transactions** — annual normalized transaction volumes (NTN), harmonised into a municipality-year panel for 2011–2025.
- **Population** — annual municipality-level population data from ISTAT POSAS, currently covering 2019–2026.

The analytical hierarchy moves from broad market patterns to increasingly granular geographic comparisons where the source data support that level of detail.

## Main analytical questions

The project is designed to investigate questions such as:

- How have residential OMI quotations evolved over time?
- How do quotation levels and growth differ across regions and municipalities?
- Which Municipality–Zone combinations show the largest long-run changes?
- How persistent and volatile is quotation growth?
- How large are historical drawdowns from local quotation peaks?
- How dispersed are quotation levels across Italian OMI zones?
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
├── docs/
│   ├── README.md
│   ├── data-sources.md
│   ├── methodology.md
│   ├── notebooks.md
│   └── reproducibility.md
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
        │   analysis-ready Parquet dataset
        │       ↓
        │   temporal / geographic / zone analysis
        │
        ├── OMI transactions
        │       ↓
        │   release discovery & schema harmonisation
        │       ↓
        │   municipality-year panel
        │       ↓
        │   NTN analysis
        │
        └── ISTAT POSAS population
                ↓
            source inventory & validation
                ↓
            municipality-year panel
                ↓
            demographic analysis
```

## OMI quotations

The quotation workflow is intentionally split into **dataset creation** and **exploration**.

### `01_01_omi_quotations_dataset_creation.ipynb`

Builds the processed quotation dataset from semiannual OMI releases. The workflow:

1. discovers releases using the expected `omi_quotations_YYYY_S1.csv` / `S2` convention;
2. derives explicit year, semester and reference-period metadata;
3. inspects raw schemas and temporal coverage;
4. performs structural and data-quality checks;
5. applies conservative cleaning and standardisation;
6. engineers analytical fields, including quotation midpoints;
7. checks geographic and property-category coverage;
8. exports the analytical dataset to Parquet.

The current documented run covers **44 quotation releases and 7,516,495 rows** before subsequent quality and transformation steps.

### `01_02_omi_quotations_exploration.ipynb`

Uses the processed quotation dataset for exploratory and comparative analysis at several geographic levels:

- national;
- regional;
- municipality;
- Municipality–Zone.

The analysis includes quotation levels, observation counts, historical trends, Top/Bottom Municipality–Zone comparisons, semester-over-semester and year-over-year growth, long-run change, CAGR, growth persistence, volatility, maximum growth/decline and historical drawdown.

The Municipality–Zone analysis aggregates observations by reference period and calculates `median_compr_mid`, `mean_compr_mid` and `observations`. The median is used as the primary descriptive statistic for cross-sectional comparison.

## OMI transactions

`02_omi_transactions_exploration.ipynb` analyses annual OMI normalized transaction volumes (**NTN — Numero di Transazioni Normalizzate**) for 2011–2025.

The notebook is designed to be robust to release-specific file layouts. It discovers annual releases from filenames, validates the expected OMI table set, harmonises year-specific schemas and uses explicit municipality keys before constructing the municipality-year panel.

The workflow includes checks for:

- expected annual releases;
- schema consistency;
- municipality-year key uniqueness;
- join cardinality;
- unmatched municipality-year records;
- reconciliation of residential NTN size classes against the total where applicable.

Expected annual OMI tables include:

- `LISTA-COM`
- `VALORI-RES`
- `VALORI-COM`
- `VALORI-PER`

## Population

`03_population_exploration.ipynb` builds a reproducible municipality-year population panel from ISTAT POSAS releases covering 2019–2026.

The workflow inventories annual source files, validates municipality/province/region data, uses the official `Età = 999` total row for the main municipality population measure, and produces demographic and geographic summaries.

Population remains a separate analytical stream in the current version of the project; cross-dataset integration is intentionally left as a subsequent analytical step.

## Data sources

The primary sources are official Italian public-data providers:

- **Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI)** for residential quotations and transaction data.
- **ISTAT — POSAS** for population data.

See [`docs/data-sources.md`](docs/data-sources.md) for source conventions, coverage and field-level notes.

## Methodology

The project follows a conservative and auditable approach to data preparation:

- raw data are inspected before transformation;
- filename conventions are validated rather than assumed;
- geographic and temporal keys are checked before panel construction;
- joins are validated explicitly;
- quotation values are not silently imputed;
- derived indicators are kept distinct from source fields;
- material transformations are documented;
- analytical notebooks separate data preparation from interpretation.

See [`docs/methodology.md`](docs/methodology.md) for the detailed analytical conventions and metric definitions.

## Reproducibility

Create and activate a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate
```

On Windows PowerShell:

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

For the quotation stream, run the notebooks in this order:

```text
01_01_omi_quotations_dataset_creation.ipynb
        ↓
01_02_omi_quotations_exploration.ipynb
```

The transaction and population notebooks are independent analytical streams and can be executed separately.

The notebooks resolve the project root from the repository structure rather than relying on a hard-coded local path.

## Dependencies

The environment is defined in `requirements.txt` and currently includes:

- pandas
- NumPy
- SciPy
- statsmodels
- scikit-posthocs
- Matplotlib
- Seaborn
- JupyterLab
- OpenPyXL
- PyArrow

## Data management

Raw source releases are organised under `data/raw/` by analytical domain. Processed analytical artefacts are written under `data/processed/` when generated by the notebooks.

Local environments, notebook checkpoints, temporary files and local configuration are excluded through `.gitignore`.

## Limitations and interpretation

OMI quotations are official market reference values and should not be interpreted as transaction-level realised sale prices. Quotation statistics also do not automatically control for changes in the composition of the property categories represented in the underlying observations.

Municipality–Zone series can have different temporal coverage and observation counts. Comparisons should therefore consider the available history and data coverage rather than treating every series as equally complete.

Transaction volumes and population data have different frequencies, definitions and coverage periods from the quotation dataset. Cross-dataset relationships should only be analysed after explicit alignment of geographic keys, frequency and observation windows.

## Documentation

- [`docs/README.md`](docs/README.md) — documentation index and project architecture
- [`docs/data-sources.md`](docs/data-sources.md) — official sources, file conventions and coverage
- [`docs/methodology.md`](docs/methodology.md) — data preparation, aggregation and analytical metrics
- [`docs/notebooks.md`](docs/notebooks.md) — notebook-by-notebook scope and execution
- [`docs/reproducibility.md`](docs/reproducibility.md) — environment and reproducibility controls

## License

See [`LICENSE`](LICENSE) for the repository license terms.
