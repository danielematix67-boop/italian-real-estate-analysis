# Italian Real Estate Analysis

Quantitative analysis of the Italian residential real-estate market using official **OMI (Osservatorio del Mercato Immobiliare)** data, complemented by **OMI transaction volumes (NTN)** and **ISTAT POSAS population data**.

The repository implements a reproducible, notebook-driven analytical workflow: official releases are ingested and validated, transformed into analysis-ready datasets or municipality-year panels, explored at multiple geographic levels, and then used for municipality-level peer benchmarking.

## Project scope

The project has three source-data streams and one downstream analytical layer:

- **OMI quotations** — semiannual residential quotation data with national, regional, municipal and OMI-zone analysis.
- **OMI transactions** — annual normalized transaction volumes (NTN), harmonised into a municipality-year panel for 2011–2025.
- **Population** — annual municipality-level population data from ISTAT POSAS, currently covering 2019–2026.
- **Municipality peer benchmark** — integrated quotation + transaction analysis for a configurable target municipality, currently **Brescia**.

## Main analytical questions

The project investigates questions such as:

- How have residential OMI quotations evolved over time?
- How do quotation levels and growth differ across regions and municipalities?
- Which Municipality–Zone combinations show the largest long-run changes?
- How persistent and volatile is quotation growth?
- How large are historical drawdowns from local quotation peaks?
- How dispersed are quotation levels across Italian OMI zones?
- How do transaction volumes evolve across municipalities?
- How does a target municipality compare with provincial and market-size peers?
- How do quotation levels, transaction volumes and market composition differ across peer groups?
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
├── LICENSE
├── .gitignore
└── README.md
```

## Analytical architecture

```text
Official data releases
        │
        ├── OMI quotations
        │       ↓
        │   01_01 ingestion & validation
        │       ↓
        │   omi_quotations.parquet
        │       ↓
        │   01_02 quotation EDA
        │
        ├── OMI transactions
        │       ↓
        │   02 schema harmonisation
        │       ↓
        │   omi_transactions_panel.parquet
        │
        └── ISTAT POSAS
                ↓
            03 population analysis

Processed quotation + transaction panels
                ↓
        04 municipality benchmark
                ↓
        peer comparisons + exports
```

The architecture deliberately separates **source ingestion**, **processed analytical data**, **exploration** and **downstream benchmarking**.

## OMI quotations

### `01_01_omi_quotations_dataset_creation.ipynb`

Builds the processed quotation dataset from semiannual OMI releases. The workflow:

1. discovers releases using the `omi_quotations_YYYY_S1.csv` / `S2` convention;
2. derives year, semester and reference-period metadata;
3. inspects raw schemas and temporal coverage;
4. performs structural and data-quality checks;
5. applies conservative cleaning and standardisation;
6. derives analytical fields including `Compr_mid`;
7. checks geographic and property-category coverage;
8. exports the analytical dataset to Parquet.

The documented run covers **44 quotation releases and 7,516,495 rows** before subsequent quality and transformation steps.

### `01_02_omi_quotations_exploration.ipynb`

Uses the processed quotation dataset for exploratory and comparative analysis at:

- national level;
- regional level;
- municipality level;
- Municipality–Zone level.

The analysis includes quotation levels, observation counts, historical trends, Top/Bottom Municipality–Zone comparisons, semester-over-semester and year-over-year growth, long-run change, CAGR, growth persistence, volatility, maximum growth/decline, historical drawdown and spatial dispersion.

At Municipality–Zone level, the core aggregation produces `median_compr_mid`, `mean_compr_mid` and `observations`.

## OMI transactions

`02_omi_transactions_exploration.ipynb` analyses annual OMI normalized transaction volumes (**NTN — Numero di Transazioni Normalizzate**) for **2011–2025**.

The notebook is designed to handle release-specific file layouts. It:

- discovers annual releases from filenames;
- validates the expected OMI table set;
- harmonises year-specific schemas;
- constructs explicit municipality-year keys;
- validates join cardinality;
- reports unmatched municipality-year records;
- reconciles residential NTN size classes against the total where possible;
- exports the municipality-year transaction panel.

Expected OMI tables include:

```text
LISTA-COM
VALORI-RES
VALORI-COM
VALORI-PER
```

## Population

`03_population_exploration.ipynb` builds a municipality-year analytical view from ISTAT POSAS releases covering **2019–2026**.

The main municipality population measure uses the official `Età = 999` total row. Population analysis remains a separate source stream and is not required by the current peer-group definition in Notebook 04.

## Municipality peer benchmark

`04_omi_comune_peer_benchmark.ipynb` is the downstream municipality deep-dive.

### Current configuration

- **Target municipality:** Brescia
- **Reference year:** latest available year by default
- **Dimensional peer tolerance:** ±50% of target total transaction volume
- **Minimum dimensional peers:** 8
- **Top provincial peers:** 30

### Peer groups

The notebook constructs three complementary peer groups:

1. **Provincial** — municipalities in the same province.
2. **Dimensional (region)** — municipalities in the same region with total transaction volume within ±50% of the target.
3. **Top-30 provincial** — the top 30 municipalities in the province by total transaction volume.

If the dimensional group has fewer than 8 peers, the notebook falls back to the top-30 provincial group.

### Benchmark dimensions

The target is compared on:

- residential NTN;
- non-residential NTN;
- total transaction volume;
- median quotation €/m²;
- median minimum and maximum quotation €/m²;
- quotation spread;
- indexed quotation evolution;
- market composition by residential NTN size band;
- CAGR;
- annualised volatility;
- maximum drawdown.

For each benchmark metric, the notebook reports the target value, peer median, peer P25/P75, percentile and peer count.

### Output

Benchmark artefacts are written to:

```text
data/processed/benchmark/
```

The export layer contains provincial, dimensional and top-30 benchmark tables plus risk/return, summary and metadata files.

## Data sources

The primary sources are official Italian public-data providers:

- **Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI)** for residential quotations and transaction data.
- **ISTAT — POSAS** for population data.

See [`docs/data-sources.md`](docs/data-sources.md) for source conventions, identifiers and coverage.

## Methodology

The project follows a conservative and auditable approach:

- raw data are inspected before transformation;
- filename conventions are validated;
- geographic and temporal keys are checked before panel construction;
- joins are validated explicitly;
- quotation values are not silently imputed;
- derived indicators remain distinct from source fields;
- benchmark parameters are explicit and reproducible;
- benchmark results are treated as descriptive comparisons rather than causal estimates.

See [`docs/methodology.md`](docs/methodology.md) for metric definitions and peer-group construction.

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

### Full analytical workflow

```text
01_01 → 01_02
           │
02 ────────┼──→ 04
           │
03 ────────┘
```

Notebook `04` requires the processed quotation and transaction artefacts. Notebook `03` is an independent demographic stream in the current implementation.

See [`docs/notebooks.md`](docs/notebooks.md) and [`docs/reproducibility.md`](docs/reproducibility.md) for the detailed execution guide.

## Dependencies

The environment is defined in `requirements.txt` and includes:

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

Raw source releases are organised under `data/raw/` by analytical domain. Processed analytical artefacts are written under `data/processed/`.

Local environments, notebook checkpoints, temporary files and local configuration are excluded through `.gitignore`.

## Limitations

OMI quotations are official market reference values and should not be interpreted as transaction-level realised sale prices. `Compr_mid` is a derived midpoint of the quoted range.

Municipality–Zone series can have different temporal coverage and observation counts. Comparisons should therefore consider the available history and data coverage.

Transaction volumes and population have different frequencies, definitions and coverage periods. Cross-dataset relationships require explicit alignment of geographic keys, frequency and observation windows.

The municipality benchmark is sample-dependent: peer definitions, reference year and transaction-volume thresholds affect the comparison. Percentiles describe the target's position within the selected peer sample and are not forecasts or causal estimates.

## Documentation

- [`docs/README.md`](docs/README.md) — documentation index and project architecture
- [`docs/data-sources.md`](docs/data-sources.md) — official sources, identifiers and coverage
- [`docs/methodology.md`](docs/methodology.md) — data preparation, aggregation, benchmark construction and analytical metrics
- [`docs/notebooks.md`](docs/notebooks.md) — notebook scope, inputs, outputs and execution order
- [`docs/reproducibility.md`](docs/reproducibility.md) — environment and reproducibility controls

## License

See `LICENSE` for the repository license terms.
