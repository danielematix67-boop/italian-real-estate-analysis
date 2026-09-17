# Data Sources

## 1. OMI quotations

**Provider:** Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI).

Semiannual quotation releases are stored under:

```text
data/raw/quotations/
```

The ingestion workflow expects the naming convention:

```text
omi_quotations_YYYY_S1.csv
omi_quotations_YYYY_S2.csv
```

The releases are semicolon-separated CSV files. Year, semester and reference-period metadata are derived explicitly from the filename and validated before the files enter the analytical dataset.

The documented dataset-creation run covers **44 valid releases**, with **7,516,495 rows and 25 columns** before subsequent quality and transformation steps.

### Main quotation fields

Geographic identifiers and descriptors include:

- `Regione`
- `Prov`
- `Comune_ISTAT`
- `Comune_cat`
- `Comune_amm`
- `Comune_descrizione`
- `Fascia`
- `Zona`
- `LinkZona`

Property classification and quotation fields include:

- `Cod_Tip`
- `Descr_Tipologia`
- `Stato`
- `Stato_prev`
- `Compr_min`
- `Compr_max`
- rental quotation fields (`Loc_*`)

The analytical workflow derives `Compr_mid` where a midpoint representation is appropriate. Growth, dispersion and drawdown measures are then calculated from the analytical quotation series.

## 2. OMI transactions

**Provider:** Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI).

Annual transaction releases are stored under:

```text
data/raw/transactions/
```

The current transaction analysis covers **2011–2025**.

The workflow validates the expected OMI tables:

```text
LISTA-COM
VALORI-RES
VALORI-COM
VALORI-PER
```

Annual files are discovered from their filenames rather than through fragile positional parsing. Because historical releases can differ in schema, the notebook harmonises year-specific structures before constructing the analytical panel.

The municipality dimension is keyed using `year` + `codcom`. Join cardinality is validated explicitly, and unmatched municipality-year records are reported. Residential NTN size classes are also reconciled against the total where the source structure permits the check.

The principal transaction indicator is **NTN (Numero di Transazioni Normalizzate)**.

## 3. Population

**Provider:** ISTAT — POSAS population releases.

Raw population data are stored under:

```text
data/raw/population/
```

The current repository contains annual releases covering **2019–2026**.

The main municipality-year population measure uses the official `Età = 999` total row. This avoids unnecessary aggregation of age-detail records for the municipality total while retaining the detailed source data for further demographic analysis.

## 4. Source-data handling

The project follows these rules:

1. Raw releases remain separate from processed analytical data.
2. Filename conventions are validated before metadata are assigned.
3. Source fields are not overwritten unnecessarily by derived metrics.
4. Missing, ambiguous or structurally unexpected records are surfaced during validation.
5. Geographic and temporal keys are checked before panel construction.
6. Dataset-specific coverage is documented because the three analytical domains do not share identical frequencies or time spans.

## 5. Processed data

Processed analytical artefacts are written under:

```text
data/processed/
```

For OMI quotations, the main analytical hand-off is stored in **Parquet**. This provides an efficient columnar format for repeated analytical reads while leaving the original CSV releases as the raw source layer.

## 6. Official source references

- Agenzia delle Entrate — [Osservatorio del Mercato Immobiliare](https://www.agenziaentrate.gov.it/portale/web/guest/schede/pagamenti/omi)
- ISTAT — [Istituto Nazionale di Statistica](https://www.istat.it/)

The repository uses official provider releases as the source layer. All transformations performed after ingestion are documented in the notebooks and methodology documentation.
