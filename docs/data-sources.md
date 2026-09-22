# Data Sources

## 1. OMI quotations

**Provider:** Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI).

Semiannual quotation releases are stored under:

```text
data/raw/quotations/
```

The ingestion workflow expects:

```text
omi_quotations_YYYY_S1.csv
omi_quotations_YYYY_S2.csv
```

The releases are semicolon-separated CSV files. Year, semester and reference-period metadata are derived explicitly from the filename and validated before the files enter the analytical dataset.

The documented dataset-creation run covers **44 quotation releases and 7,516,495 rows** before subsequent quality and transformation steps.

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

The analytical workflow derives `Compr_mid` from the quotation range where appropriate. The Municipality–Zone exploration also derives quotation growth, spreads, volatility and drawdown measures.

## 2. OMI transactions

**Provider:** Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI).

Annual transaction releases are stored under:

```text
data/raw/transactions/
```

The current transaction analysis covers **2011–2025**.

The workflow validates the expected OMI table set:

```text
LISTA-COM
VALORI-RES
VALORI-COM
VALORI-PER
```

Historical releases can differ in layout and schema. The notebook therefore discovers annual files from filenames, harmonises year-specific structures and constructs the municipality dimension explicitly.

The principal transaction indicator is **NTN (Numero di Transazioni Normalizzate)**.

The municipality key is based on **year + `codcom`**. Join cardinality and unmatched municipality-year records are checked before the panel is accepted.

Where the source structure permits, residential NTN size bands are reconciled against the reported residential total.

## 3. Population

**Provider:** ISTAT — POSAS population releases.

Raw population data are stored under:

```text
data/raw/population/
```

The current repository contains annual releases covering **2019–2026**.

The principal municipality population measure uses the official `Età = 999` total row. Detailed age records remain available for demographic analysis.

The population stream uses municipality-level identifiers that are distinct from OMI's alphanumeric `codcom` identifiers. Any cross-dataset integration must therefore validate or construct an explicit crosswalk.

## 4. Benchmark inputs

Notebook `04_omi_comune_peer_benchmark.ipynb` does not ingest raw source releases directly.

Its principal inputs are processed artefacts produced upstream:

- `data/processed/omi_quotations.parquet`
- `data/processed/omi_transactions_panel.parquet`

It aggregates residential quotations from semester-level observations to municipality-year statistics and joins them with the annual transaction panel.

The benchmark currently uses total transaction volume:

```text
vol_tot = ntn_res + ntn_non_res
```

as its market-size proxy for peer construction.

## 5. Identifier conventions

The repository contains several identifier systems that must not be conflated:

| Identifier | Source / use |
|---|---|
| `Comune_ISTAT` | OMI/administrative geographic identifier |
| `codcom` | OMI municipality code used in transaction panels |
| `Comune_n` / `Regione_n` / `Prov_n` | Normalised analytical keys used in the benchmark |
| `reference_year` / `reference_date` | Quotation temporal dimensions |
| `year` | Annual panel dimension |

Joins are therefore validated explicitly rather than assuming that similarly named municipality fields are interchangeable.

## 6. Source-data handling

The project follows these rules:

1. Raw releases remain separate from processed analytical data.
2. Filename conventions are validated before metadata are assigned.
3. Source fields are not overwritten unnecessarily by derived metrics.
4. Missing, ambiguous or structurally unexpected records are surfaced during validation.
5. Geographic and temporal keys are checked before panel construction.
6. Dataset-specific coverage is documented because the three source domains have different frequencies and time spans.
7. Benchmark notebooks consume processed artefacts and do not silently rebuild upstream data.

## 7. Processed data

Processed analytical artefacts are written under:

```text
data/processed/
```

The main quotation hand-off is stored in Parquet. The transaction analytical panel is also stored in Parquet.

Benchmark-specific CSV outputs are stored under:

```text
data/processed/benchmark/
```

## 8. Official sources

- Agenzia delle Entrate — Osservatorio del Mercato Immobiliare (OMI)
- ISTAT — Istituto Nazionale di Statistica

The repository uses official provider releases as its source layer. Transformations performed after ingestion are documented in the notebooks and methodology guide.
