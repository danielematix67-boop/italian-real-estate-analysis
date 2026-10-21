# Data Sources

This project uses official source releases from Agenzia delle Entrate's Osservatorio del Mercato Immobiliare (OMI) and ISTAT. Raw files are stored by domain and are kept separate from processed analytical artefacts.

## 1. OMI residential quotations

**Provider:** Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI)  
**Raw directory:** `data/raw/quotations/`  
**Processed output:** `data/processed/omi_quotations.parquet`

The dataset-creation notebook discovers files matching:

```text
omi_quotations_YYYY_S1.csv
omi_quotations_YYYY_S2.csv
```

Files are semicolon-separated. The year, semester and reference period are derived from filenames and validated before processing. The ingestion notebook checks schema, duplicate rows, missing values, numeric conversion, range consistency, candidate-key duplicates, temporal coverage and geographic/property-category coverage.

Core fields include geographic descriptors (such as `Regione`, `Prov`, `Comune_ISTAT`, `Comune_descrizione`, `Fascia` and `Zona`), property type and condition, and quotation bounds `Compr_min` and `Compr_max`. `Compr_mid` is derived for analysis from the quotation bounds; it is not a realised transaction price.

The exploration notebook analyses residential quotation levels and trends at national, regional, provincial and municipal levels, with a focused Lombardy/Brescia section. Geographic coverage and observation counts can differ by period and aggregation level.

## 2. OMI transactions (NTN)

**Provider:** Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI)  
**Raw directory:** `data/raw/transactions/`  
**Processed output:** `data/processed/omi_transactions_panel.parquet`

The transaction notebook discovers annual files using a `YYYY_<TABLE>.csv` pattern and validates the expected table set:

- `LISTA-COM`
- `VALORI-RES`
- `VALORI-COM`
- `VALORI-PER`

The current documented analytical coverage is 2011–2025. Historical releases can vary in column casing, names and year-specific NTN fields, so the notebook harmonises schemas before constructing the panel.

The primary indicator is **NTN (Numero di Transazioni Normalizzate)**. Residential size-band measures are retained where available; non-residential segments are aggregated from the relevant source tables. The notebook performs checks on municipality-year uniqueness, non-negative values, missing keys and residential size-band reconciliation.

## 3. ISTAT POSAS population

**Provider:** ISTAT — Istituto Nazionale di Statistica, POSAS (resident population by sex, age and municipality)  
**Raw directory:** `data/raw/population/<year>/`

The notebook discovers annual folders and reads the relevant POSAS files for municipalities, provinces and regions. The documented coverage is 2019–2026, subject to the files present in the local data directory.

For the main municipality total, the notebook selects records with `Età = 999`, the official total row, rather than summing detailed age records. It checks municipality-year key uniqueness and missing or negative population values. Regional age structure is also analysed using age bands 0–14, 15–64 and 65+.

Population remains a separate analytical stream in the current benchmark: Notebook 04 does not consume the population panel.

## 4. Processed benchmark inputs and outputs

Notebook `04_omi_comune_peer_benchmark.ipynb` consumes these upstream processed files:

- `data/processed/omi_quotations.parquet`
- `data/processed/omi_transactions_panel.parquet`

It aggregates quotation observations to municipality-year level and joins them to annual transaction data using normalised municipality dimensions. The total transaction volume used in peer construction is derived as:

```text
vol_tot = ntn_res + ntn_non_res
```

The notebook writes benchmark CSV artefacts to `data/processed/benchmark/`, including peer-group tables, risk/return metrics, a summary and metadata for the configured target municipality.

## 5. Geographic identifiers and joins

The sources use different geographic keys and labels. Examples include OMI quotation fields such as `Comune_ISTAT` and `Comune_descrizione`, the transaction panel's `codcom`, and normalised municipality/province/region keys created for the benchmark.

These identifiers must not be assumed interchangeable. The benchmark applies name normalisation and joins its own prepared dimensions; any future integration with population data should use an explicit and validated municipality-code crosswalk rather than relying only on display names.

## 6. Data-handling principles

1. Keep original source releases separate from derived datasets.
2. Validate file names, expected tables and source structure before assigning metadata.
3. Preserve source fields where practical and label derived measures clearly.
4. Surface missing values, unexpected records, duplicates and unmatched keys.
5. Check join cardinality and panel uniqueness before using aggregates.
6. Record source coverage and reference periods because the domains differ in frequency and time span.
7. Do not interpret quotation midpoints as observed sale prices.
8. Treat benchmark percentiles as descriptive statistics tied to a defined peer sample.

## 7. Official providers

- Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI)
- ISTAT — Istituto Nazionale di Statistica (POSAS)

Consult the provider releases and the notebook-level validation output when updating source files; the documented coverage should be revisited as new releases are added.
