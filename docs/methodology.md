# Methodology

## 1. Analytical philosophy

The project treats data preparation and exploratory analysis as separate stages. The objective is to make every transformation explicit, reproducible and auditable before using the resulting data for market comparisons.

The main principles are:

- inspect source data before transformation;
- validate structural assumptions;
- use conservative cleaning rules;
- avoid silent imputation of quotation values;
- preserve geographic and temporal identifiers;
- report the effect of material transformations;
- distinguish source measures from derived indicators.

## 2. OMI quotation dataset

The quotation pipeline reads one semiannual release at a time and derives:

- `reference_year`;
- `reference_semester`;
- `reference_period`.

The quotation range is represented by `Compr_min` and `Compr_max`. Where a midpoint is used for analysis, it is a derived analytical measure and should not be confused with an observed transaction price.

The main residential analysis operates at the `Comune_descrizione` + `Zona` level. This allows the analysis to retain the OMI zoning structure rather than collapsing every municipality into a single value.

## 3. Municipality–Zone aggregation

For each reference period and Municipality–Zone pair, the exploration notebook calculates:

```text
median_compr_mid
mean_compr_mid
observations
```

The **median** is the primary comparison statistic because it reduces the influence of unusually high or low observations across property categories and OMI zones. The mean is retained as a complementary descriptive statistic.

## 4. Growth metrics

### Semester-over-semester growth

For a Municipality–Zone pair:

```text
growth_pct = (P_t / P_(t-1) - 1) × 100
```

where `P` is the median quotation midpoint.

### Year-over-year growth

The intended comparison is the same semester in the previous year:

```text
YoY = (P_t / P_(t-1 year) - 1) × 100
```

When implementing this metric for irregular or incomplete series, period alignment should be preferred to relying only on row position.

### Overall variation

```text
overall_change_pct = (last_price / first_price - 1) × 100
```

This measures the total change over the available observation window.

### CAGR

```text
CAGR = (last_price / first_price)^(1 / years) - 1
```

CAGR normalises long-run growth for the length of the available observation window.

### Growth volatility

The standard deviation of semester-over-semester growth rates is used as a descriptive measure of growth variability.

### Growth persistence

The analysis counts positive and negative growth periods and derives:

```text
positive_growth_share_pct =
    positive_periods / (positive_periods + negative_periods) × 100
```

Periods with no calculable growth are not classified as negative periods.

## 5. Drawdown

For each Municipality–Zone series, the notebook maintains a cumulative historical maximum:

```text
rolling_max = cumulative maximum of median_compr_mid
```

Drawdown is then:

```text
drawdown_pct = (median_compr_mid / rolling_max - 1) × 100
```

`max_drawdown_pct` identifies the largest decline from a previous observed peak in the available series, while `latest_drawdown_pct` describes the position of the latest observation relative to the historical maximum.

## 6. National distribution

The national quotation analysis can be summarised through cross-sectional percentiles for each semester, including P10, P25, P50, P75 and P90.

The P90–P10 spread is used as a descriptive measure of the dispersion between the upper and lower parts of the Municipality–Zone quotation distribution.

## 7. Intra-municipality dispersion

For each municipality and period, the analysis can compare the highest and lowest zone-level median quotation:

```text
spread_eur_m2 = max_zone_price - min_zone_price

spread_pct = (max_zone_price / min_zone_price - 1) × 100
```

This captures the internal spatial differentiation of OMI quotations within a municipality.

## 8. Transaction analysis

The transaction notebook builds an annual municipality-level panel from the OMI transaction tables. The pipeline first inventories available releases and validates the expected table set, then harmonises schemas before joining data.

The resulting panel supports analysis of NTN and related transaction-volume indicators by municipality and year.

## 9. Population analysis

The population notebook builds a municipality-year panel from ISTAT POSAS releases. For the main population measure it uses the official `Età = 999` total row.

The resulting data can support demographic trend analysis and future contextual joins with real-estate indicators.

## 10. Comparability and limitations

Several analytical limitations should remain explicit:

- OMI quotations are reference quotation ranges and are not equivalent to realised sale prices.
- Municipality–Zone series can have different observation histories and coverage.
- Aggregate quotation statistics do not control for changes in the composition of properties represented in the underlying OMI categories.
- Transaction volumes and population have different frequencies and coverage periods from OMI quotations.
- Cross-dataset relationships should be analysed only after explicit key, frequency and coverage alignment.

These limitations are part of the analytical methodology rather than reasons to exclude the datasets.
