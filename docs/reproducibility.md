# Reproducibility Guide

## 1. Environment setup

The project uses Python notebooks and common data-analysis libraries. Create a virtual environment from the repository root.

Linux/macOS:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
jupyter lab
```

Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
jupyter lab
```

Dependencies are listed in `requirements.txt`. For fully repeatable results, record the Python and dependency versions used for each analysis; the requirements file currently lists packages without pinning every version.

## 2. Expected input layout

```text
data/
└── raw/
    ├── quotations/
    │   └── omi_quotations_YYYY_S1.csv
    │       omi_quotations_YYYY_S2.csv
    ├── transactions/
    │   └── YYYY_<TABLE>.csv
    └── population/
        └── <year>/
            ├── POSAS_<year>_it_Comuni.csv
            ├── POSAS_<year>_it_Province.csv
            └── POSAS_<year>_it_Regioni.csv
```

The actual file availability can vary by release and source. Preserve official source files and naming conventions where possible; the notebooks validate expected files and surface structural differences.

## 3. Build the processed quotation dataset

Run `notebooks/01_01_omi_quotations_dataset_creation.ipynb` from the repository.

It discovers semicolon-separated quotation files in `data/raw/quotations/`, checks schema and data quality, derives temporal metadata and quotation features, then writes:

```text
data/processed/omi_quotations.parquet
```

Next, run `notebooks/01_02_omi_quotations_exploration.ipynb` to explore the processed dataset. Notebook 01_02 does not replace the upstream dataset-creation step.

## 4. Build the transaction panel

Run `notebooks/02_omi_transactions_exploration.ipynb`.

The notebook discovers annual releases in `data/raw/transactions/`, harmonises historical schemas, constructs the municipality-year panel and runs quality checks. The processed hand-off is:

```text
data/processed/omi_transactions_panel.parquet
```

Review the notebook's reconciliation and unmatched-record diagnostics before interpreting aggregates or committing changed outputs.

## 5. Run population analysis

Run `notebooks/03_population_exploration.ipynb`. It discovers annual POSAS releases under `data/raw/population/<year>/` and uses the official `Età = 999` row for municipality totals. The notebook's current analysis is self-contained; population is not currently consumed by Notebook 04.

## 6. Run the municipality benchmark

Notebook 04 requires both processed Parquet files:

```text
data/processed/omi_quotations.parquet
data/processed/omi_transactions_panel.parquet
```

Run `notebooks/04_omi_comune_peer_benchmark.ipynb`. The default target is Brescia, and the reference year is automatically selected from the latest available year unless `LATEST_YEAR` is set explicitly.

The main configuration parameters are:

```python
TARGET_COMUNE = "BRESCIA"
LATEST_YEAR = None
PEER_VOL_TOL = 0.50
MIN_PEERS = 8
TOP_N_PROV = 30
MIN_OBS_FOR_RISK = 3
```

Benchmark CSVs are written to `data/processed/benchmark/`. The notebook creates the output directory when required. Peer definitions and the reference year affect results, so retain those parameters with any reported benchmark.

## 7. Suggested execution sequence

```text
01_01 -> 01_02
02 -------------> 04
01_01 output ----> 04
03 (independent)
```

Run 01_01 and 02 whenever their respective raw inputs change. Run 01_02 after updating the quotation dataset. Rerun 04 after either upstream processed dataset changes. Run 03 when population source data change.

## 8. Data-update checklist

When a new release is added:

1. save it under the matching `data/raw/<domain>/` folder;
2. preserve the original file and check its filename and delimiter;
3. rerun the relevant notebook from the top;
4. inspect schema, date coverage, missingness, duplicates and quality checks;
5. review transaction reconciliation and unmatched-key diagnostics where relevant;
6. regenerate the processed Parquet artefact;
7. rerun dependent notebooks;
8. review charts, summary text and exported tables;
9. document material changes to coverage or methodology.

## 9. Reproducibility and interpretation

- Run notebooks from a clean kernel in top-to-bottom order.
- Keep source data, processed datasets and analytical exports conceptually separate.
- Do not treat notebook output snapshots as proof that a new run has succeeded; inspect the current run's checks and outputs.
- Derived fields such as `Compr_mid`, growth rates and drawdown metrics should be distinguishable from provider-supplied fields.
- OMI quotations are reference ranges, not realised transaction prices.
- Peer percentiles and risk/return-style metrics are descriptive and sample-dependent, not forecasts or causal estimates.
- Population and OMI identifiers differ; do not join them without an explicit, validated crosswalk.

The repository's `.gitignore` excludes virtual environments, notebook checkpoints, caches, temporary outputs and local configuration. Keep sensitive local configuration and unnecessary generated files out of version control.
