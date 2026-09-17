# Reproducibility Guide

## 1. Environment

The project is Python-based and uses a virtual environment for local execution.

Create the environment:

```bash
python -m venv .venv
```

Activate on Linux/macOS:

```bash
source .venv/bin/activate
```

Activate on Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Install the project dependencies:

```bash
pip install -r requirements.txt
```

Launch JupyterLab:

```bash
jupyter lab
```

## 2. Expected repository layout

The notebooks expect the following data directories:

```text
data/raw/quotations/
data/raw/transactions/
data/raw/population/
data/processed/
```

The quotation dataset-creation notebook checks for its quotation input directory. The transaction and population notebooks perform equivalent project-root and input-directory checks for their respective domains.

## 3. Reproducing the quotation workflow

First run:

```text
notebooks/01_01_omi_quotations_dataset_creation.ipynb
```

This notebook discovers semiannual releases, validates their filenames and schemas, performs data-quality checks and writes the processed quotation dataset to `data/processed/`.

Then run:

```text
notebooks/01_02_omi_quotations_exploration.ipynb
```

This notebook consumes the processed dataset and performs the national, regional, municipality and Municipality–Zone analysis.

## 4. Reproducing the transaction workflow

Run:

```text
notebooks/02_omi_transactions_exploration.ipynb
```

The notebook discovers annual OMI transaction releases for 2011–2025, validates the expected table set, harmonises historical schemas and constructs a municipality-year panel.

The workflow explicitly validates the `year` + `codcom` municipality key, join cardinality and unmatched municipality-year records. Residential NTN size classes are reconciled against the total where applicable.

## 5. Reproducing the population workflow

Run:

```text
notebooks/03_population_exploration.ipynb
```

The notebook inventories the annual POSAS releases for 2019–2026, validates the expected source structure and constructs the municipality-year population panel using the official `Età = 999` total row for municipality totals.

## 6. Reproducibility controls

The notebooks use several controls to reduce environment-specific behaviour:

- project roots are resolved from the repository structure;
- input directories are checked explicitly;
- release files are discovered using documented filename conventions;
- duplicate or unexpected release combinations are surfaced;
- schemas are harmonised before historical panel construction where required;
- joins are preceded by explicit key/cardinality validation;
- unmatched records are reported rather than silently dropped;
- quotation transformations avoid silent imputation;
- generated processed artefacts are kept separate from raw releases.

## 7. Updating the data

When a new official release is added, the preferred workflow is:

1. place the raw release in the appropriate `data/raw/<domain>/` directory;
2. preserve the provider's original naming and file structure where possible;
3. rerun the relevant ingestion or validation notebook;
4. inspect schema, coverage and quality checks;
5. regenerate the processed analytical artefact where applicable;
6. rerun the corresponding exploration notebook;
7. review tables and visualisations before committing changes.

## 8. Version control

The repository `.gitignore` excludes local virtual environments, notebook checkpoints, temporary files, build artefacts and local configuration.

The notebooks and documentation are the reproducible record of the analytical workflow. Generated outputs should not be treated as the source of truth.
