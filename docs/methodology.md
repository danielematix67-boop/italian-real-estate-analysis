# Methodology

## 1. Analytical design

The repository separates source-data preparation, exploratory analysis and integrated municipality benchmarking. The main data flow is:

```text
OMI quotations -> 01_01 preparation -> omi_quotations.parquet -> 01_02 exploration
OMI transactions -> 02 harmonisation -> omi_transactions_panel.parquet
ISTAT POSAS -> 03 population analysis (independent stream)

Processed quotations + transaction panel -> 04 municipality peer benchmark
```

The objective is a transparent and reproducible analytical workflow, not a causal model. Source fields, derived variables and interpretations should remain distinguishable.

## 2. OMI quotation preparation

Notebook 01_01 loads semiannual, semicolon-separated quotation releases and derives temporal metadata from the filename:

- reference year;
- reference semester;
- reference period.

It inspects schema and coverage, measures missingness, converts quotation fields to numeric types, checks zero values and range consistency, identifies candidate-key duplicates, standardises text, and validates temporal and geographic coverage. The cleaning logic is conservative: data-quality issues are surfaced rather than hidden through broad imputation.

### Quotation midpoint and spread

Where the minimum and maximum purchase quotations are available:

```text
Compr_mid = (Compr_min + Compr_max) / 2
spread_eur_m2 = Compr_max - Compr_min
```

The midpoint is a derived descriptive proxy for the centre of an official quotation range. It is not an observed transaction-level sale price. The exact set of fields available can differ by source release.

## 3. Quotation exploration

Notebook 01_02 reads the processed quotation dataset and focuses on residential observations. It explores national, regional, provincial and municipal quotation levels, trends and geographic differences, including a Lombardy/Province of Brescia deep dive.

Descriptive summaries use statistics such as median and mean quotation midpoint and observation counts. Growth comparisons depend on the reference periods available in each series. Results can be affected by changes in coverage, property mix and the set of locations observed; they should not be interpreted as quality-adjusted price indices unless an explicit adjustment has been implemented.

For a time series (P_t), common descriptive transformations include:

```text
period_growth_pct = (P_t / P_(t-1) - 1) * 100
overall_change_pct = (P_last / P_first - 1) * 100
CAGR = (P_last / P_first)^(1 / years) - 1
```

Growth volatility can be summarised by the standard deviation of observed period growth rates. Such statistics describe historical behaviour and do not forecast future returns.

## 4. OMI transaction harmonisation

Notebook 02 discovers annual OMI transaction files and harmonises schema drift across historical releases. It standardises source column names, maps year-specific NTN fields, builds residential and non-residential components, and assembles a municipality-year panel.

Residential NTN is retained by official property-size band where available. The sum of size-band values is compared with the reported residential total; any reconciliation gap should be inspected. Non-residential measures are aggregated from the relevant source tables without assuming that every year has identical column names.

The final panel is checked for non-empty input catalogues, multi-year coverage, unique municipality-year keys, non-negative residential NTN and missing key values. The notebook also reports unmatched records and other data-quality diagnostics before downstream use.

### Transaction totals

The benchmark derives total transaction volume as:

```text
vol_tot = ntn_res + ntn_non_res
```

Residential and non-residential NTN are distinct measures and should be reviewed separately as well as in aggregate.

## 5. Statistical analysis of transaction activity

Notebook 02 combines descriptive statistics with distributional and cross-sectional analysis. NTN values can be strongly right-skewed because a relatively small number of municipalities account for high transaction volumes. A (log(1 + NTN)) transformation is used where a transformed distribution is more informative.

The notebook also compares distributions across macro-areas, including non-parametric Kruskal–Wallis testing and Dunn post-hoc comparisons with multiple-testing correction where implemented. These tests evaluate distributional differences within the analysed sample; they do not establish why differences occur or imply causal effects.

Other analyses include size-band composition, national growth decomposition, concentration of transactions among municipalities, regional heterogeneity and municipality-level growth dynamics. Interpret results alongside coverage, missingness and the treatment of zero or undefined growth.

## 6. Population analysis

Notebook 03 processes ISTAT POSAS releases by year and geography. The main municipality population measure uses the official `Età = 999` total row rather than summing detailed age rows. It checks the uniqueness of municipality-year keys and missing or negative population values.

The notebook examines national population trends, municipality-level changes, population concentration, regional and provincial context, and regional age structure. Age structure is grouped into 0–14, 15–64 and 65+.

Population is not currently an input to the Notebook 04 peer definition. Future integration requires aligning the temporal dimensions and creating a validated crosswalk between ISTAT and OMI geographic identifiers.

## 7. Municipality peer benchmark

Notebook 04 consumes the processed quotation and transaction Parquet files; it does not reload raw source releases. It normalises municipality labels, aggregates semester-level residential quotations to municipality-year statistics, and joins them to the annual transaction panel.

The current default target is `BRESCIA`. If `LATEST_YEAR` is `None`, the notebook selects the latest year available in the integrated panel.

### Peer-group definitions

| Group | Construction |
|---|---|
| Provincial | Municipalities in the target's province |
| Dimensional (region) | Municipalities in the target's region with total NTN within the configured tolerance of the target |
| Top-N provincial | Highest-volume municipalities in the target's province, up to the configured N |

Current defaults are `PEER_VOL_TOL = 0.50`, `MIN_PEERS = 8`, `TOP_N_PROV = 30`, and `MIN_OBS_FOR_RISK = 3`. If the dimensional peer group has fewer than the minimum number of peers, the notebook falls back to the Top-N provincial group for that comparison.

### Benchmark statistics and analysis

For the target and peer samples, the notebook calculates comparisons such as peer medians, quartiles, target percentiles and peer counts. It explores residential and non-residential NTN, total volume, quotation levels and spreads, target-versus-peer time series, indexed quotation evolution, quotation versus transaction volume, residential share, and residential size-band composition.

For sufficiently long quotation histories, the notebook calculates CAGR, annualised volatility and maximum drawdown. Series with fewer than `MIN_OBS_FOR_RISK` observations do not meet the configured minimum for risk-style statistics.

Benchmark exports are written as CSV files to `data/processed/benchmark/` and include peer-group tables, risk/return metrics, a summary and metadata. Results depend on target, reference year, peer definitions and data availability.

## 8. Comparability and limitations

- OMI quotations are reference ranges, not realised sale prices.
- `Compr_mid` is derived from quoted minimum and maximum values.
- Quotation observations may differ in geographic, temporal and property-type coverage.
- Annual transaction data and semiannual quotation data must be aligned explicitly.
- Historical transaction files can differ in schema and coverage.
- Geographic identifiers across OMI and ISTAT are not interchangeable by assumption.
- The current benchmark uses transaction volume as a market-size proxy, not population.
- Peer-group definitions and fallbacks affect percentiles and comparisons.
- Growth, volatility, CAGR and drawdown are descriptive historical measures, not forecasts.
- Statistical differences and correlations do not establish causal relationships.

The project is intended for reproducible descriptive market analysis and exploratory research. Results should be interpreted with their source coverage, definitions and data-quality diagnostics.
