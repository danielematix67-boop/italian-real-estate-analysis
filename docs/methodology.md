# Methodology

## 1. Analytical philosophy

The project separates **data preparation**, **exploratory analysis** and **municipality benchmarking**.

The objective is to make transformations explicit and reproducible before using the resulting datasets for comparison.

Core principles are:

- inspect source data before transformation;
- validate structural assumptions before joins or aggregation;
- use conservative cleaning rules;
- avoid silent imputation of quotation values;
- preserve geographic and temporal identifiers;
- report material transformations and validation outcomes;
- distinguish source measures from derived indicators;
- treat benchmark results as descriptive comparisons rather than causal estimates.

## 2. OMI quotation dataset

The quotation ingestion workflow processes semiannual releases and derives:

- `reference_year`;
- `reference_semester`;
- `reference_period`.

The source quotation range is represented by `Compr_min` and `Compr_max`. Where a midpoint is required:

```text
Compr_mid = (Compr_min + Compr_max) / 2
```

`Compr_mid` is an analytical proxy for the centre of the quoted range; it is **not an observed transaction price**.

## 3. Municipality–Zone aggregation

For each reference period and Municipality–Zone pair, the exploration workflow calculates:

```text
median_compr_mid
mean_compr_mid
observations
```

The median is used as the primary descriptive statistic because it is less sensitive to extreme observations than the mean. The observation count provides a basic coverage indicator.

## 4. Growth metrics

### Semester-over-semester growth

```text
growth_pct = (P_t / P_(t-1) - 1) × 100
```

### Year-over-year growth

The preferred comparison is the same semester in the previous year:

```text
YoY = (P_t / P_(t-1 year) - 1) × 100
```

### Overall variation

```text
overall_change_pct = (last_price / first_price - 1) × 100
```

### CAGR

```text
CAGR = (last_price / first_price)^(1 / years) - 1
```

### Growth volatility

The standard deviation of calculable period-to-period growth rates is used as a descriptive measure of growth variability.

### Growth persistence

Positive and negative growth periods are counted separately. Periods where growth cannot be calculated are excluded from the denominator.

## 5. Drawdown analysis

For each Municipality–Zone series:

```text
rolling_max = cumulative maximum of median_compr_mid
drawdown_pct = median_compr_mid / rolling_max - 1
```

The main descriptive measures are:

- `max_drawdown_pct`;
- `latest_drawdown_pct`.

## 6. Distribution and spatial-dispersion analysis

Cross-sectional quotation distributions can be summarised with P10, P25, P50, P75 and P90.

For municipality-level zone dispersion:

```text
spread_eur_m2 = max_zone_price - min_zone_price
spread_pct = (max_zone_price / min_zone_price - 1) × 100
```

These measures describe dispersion in OMI quotation ranges rather than realised transaction-price dispersion.

## 7. OMI transaction analysis

The transaction workflow constructs an annual municipality-level panel from OMI releases covering 2011–2025.

The pipeline:

1. inventories annual releases;
2. validates the expected OMI table set;
3. harmonises year-specific schemas;
4. constructs explicit municipality-year keys;
5. validates join cardinality;
6. reports unmatched municipality-year records;
7. reconciles residential NTN size bands where possible;
8. exposes the resulting municipality-year panel for downstream analysis.

Residential and non-residential NTN are retained separately. Total transaction volume is derived as:

```text
vol_tot = ntn_res + ntn_non_res
```

## 8. Population analysis

The population workflow constructs a municipality-year panel from ISTAT POSAS releases covering 2019–2026.

For the principal municipality total, the official `Età = 999` row is used instead of unnecessarily summing age-detail rows.

Population is currently a separate analytical stream. It is not required for the current peer-group construction in Notebook 04.

## 9. Municipality peer benchmarking

Notebook `04_omi_comune_peer_benchmark.ipynb` provides a reproducible municipality deep-dive based on processed quotation and transaction panels.

### 9.1 Municipality-year quotation aggregation

Residential quotation observations are filtered and aggregated to municipality-year level using:

- median `Compr_mid`;
- mean `Compr_mid`;
- median `Compr_min`;
- median `Compr_max`;
- median quotation spread;
- number of quotation observations.

This converts the semiannual quotation data into a frequency compatible with annual transaction data.

### 9.2 Target municipality

The notebook has a configurable target municipality. The current default is:

```text
TARGET_COMUNE = "BRESCIA"
```

The reference year defaults to the latest year available in the integrated municipality-year panel.

### 9.3 Peer groups

Three complementary peer groups are constructed:

| Peer group | Definition |
|---|---|
| Provincial | All municipalities in the target's province |
| Dimensional (region) | Same region and total transaction volume within ±50% of the target |
| Top-N provincial | Top 30 municipalities in the target province by total transaction volume |

If the dimensional peer group contains fewer than 8 municipalities, the notebook falls back to the top-30 provincial peer group.

The peer definitions are descriptive and depend on the selected reference year.

### 9.4 Benchmark statistics

For each metric, the benchmark reports:

- target value;
- peer median;
- peer P25;
- peer P75;
- target percentile;
- peer count.

The benchmark currently covers:

- residential NTN;
- non-residential NTN;
- total transaction volume;
- median quotation €/m²;
- median minimum quotation €/m²;
- median maximum quotation €/m²;
- median quotation spread %.

### 9.5 Time-series comparison

The notebook compares the target municipality with the median of the selected dimensional peer group for:

- residential NTN;
- median quotation;
- non-residential NTN;
- indexed quotation evolution.

The indexed quotation series is rebased to the first available year.

### 9.6 Market composition

Residential transaction composition is examined using the available size bands:

- ≤50 m²;
- 50–85 m²;
- 85–115 m²;
- 115–145 m²;
- >145 m².

Residential share is defined as:

```text
share_res = ntn_res / (ntn_res + ntn_non_res)
```

### 9.7 Risk/return-style descriptive metrics

For the target and selected peer median quotation series, the notebook calculates:

- CAGR;
- annualised volatility;
- maximum drawdown.

These are descriptive time-series statistics and should not be interpreted as investment forecasts.

## 10. Statistical analysis

The repository includes SciPy, statsmodels and scikit-posthocs for distributional analysis, statistical testing and post-hoc comparisons where appropriate.

The benchmark notebook uses percentile comparisons and distributional visualisations. Statistical association does not establish causality.

Any future cross-dataset analysis involving population should explicitly distinguish cross-sectional relationships from within-municipality temporal relationships.

## 11. Comparability and limitations

Key limitations are explicit:

- OMI quotations are reference values, not realised sale prices.
- `Compr_mid` is a derived midpoint.
- Municipality–Zone series have different observation histories and coverage.
- Quotation statistics are not automatically quality-adjusted.
- Transaction and quotation frequencies differ and therefore require explicit temporal aggregation.
- Municipality identifiers differ across source systems.
- Peer groups depend on the selected reference year and peer-definition parameters.
- The dimensional peer group uses transaction volume as a market-size proxy rather than population.
- Small peer groups trigger a documented fallback rule.
- Benchmark percentiles are descriptive and sample-dependent.
- Transaction activity and quotation levels can be affected by structural, demographic, supply, demand and macroeconomic factors that are not controlled for by the benchmark.

The current analysis is therefore intended for **descriptive market intelligence and exploratory research**, not causal identification or investment advice.
