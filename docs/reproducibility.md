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

Install the pinned project dependency list:

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

The quotation dataset-creation notebook explicitly checks for `data/raw/quotations`. The transaction and population notebooks perform equivalent project-root and input-directory checks for their respective domains.

## 3. Reproducing the quotation dataset

Run:

```text
notebooks/01_01_omi_quotations_dataset_creation.ipynb
```

The notebook discovers quotation releases, validates the expected filename convention and creates the analysis-ready dataset. It is the required upstream step for the quotation exploration notebook when the processed dataset has not already been generated.

Then run:

```text
notebooks/01_02_omi_quotations_exploration.ipynb
```

## 4. Reproducing transactions analysis

Run:

```text
notebooks/02_omi_transactions_exploration.ipynb
```

The notebook discovers annual OMI transaction releases from filenames, validates the expected table set and constructs the municipality-year analytical panel.

## 5. Reproducing population analysis

Run:

```text
notebooks/03_population_exploration.ipynb
```

The notebook inventories the annual POSAS releases, validates the expected municipality/province/region files and constructs the municipality-year population panel.

## 6. Reproducibility controls

The notebooks use several controls to reduce environment-specific behaviour:

- project roots are detected relative to the notebook execution location;
- input directories are checked explicitly;
- release files are discovered using documented filename conventions;
- duplicate or unexpected release combinations are surfaced;
- joins are preceded by key validation in the transaction workflow;
- quotation transformations are designed to avoid silent data loss;
- generated processed artefacts are kept separate from raw releases.

## 7. Updating the data

When a new official release is added, the preferred workflow is:

1. place the raw release in the appropriate `data/raw/<domain>/` directory;
2. preserve the provider's original file structure and naming convention where possible;
3. rerun the relevant ingestion/validation notebook;
4. inspect coverage and quality checks;
5. regenerate the processed analytical artefact;
6. rerun the corresponding exploration notebook;
7. review the resulting tables and visualisations before committing notebook changes.

## 8. Version-control considerations

The repository `.gitignore` excludes local virtual environments, notebook checkpoints, temporary files, build artefacts and local configuration such as Streamlit secrets. Generated output directories are also excluded from version control.

The source notebooks and documentation remain the reproducible record of the analytical workflow; generated outputs should not be treated as the source of truth.
