# Methodology

## 1. Analytical philosophy

The project separates **data preparation** from **exploratory analysis**. The objective is to make transformations explicit, reproducible and auditable before using the resulting datasets for market comparisons.

Core principles are:

- inspect source data before transformation;
- validate structural assumptions before joins or aggregation;
- use conservative cleaning rules;
- avoid silent imputation of quotation values;
- preserve geographic and temporal identifiers;
- report material transformations and validation outcomes;
- distinguish source measures from derived indicators.

## 2. OMI quotation dataset

The quotation ingestion workflow processes semiannual releases and derives:

- `reference_year`;
- `reference_semester`;
- `reference_period`.

The source quotation range is represented by `Compr_min` and `Compr_max`. Where a midpoint is required for descriptive analysis, it is derived from the quotation range and represented by `Compr_mid`.

`Compr_mid` is an analytical proxy for the centre of the quoted range; it is **not an observed transaction price**.

The residential exploration retains the OMI geographic hierarchy and supports analysis at national, regional, municipality and Municipality–Zone levels.

## 3. Municipality–Zone aggregation

For each reference period and Municipality–Zone pair, the exploration workflow calculates:

```text
median_compr_mid
mean_compr_mid
observations
```

The **median** is the primary comparison statistic because it is less sensitive to unusually high or low observations than the mean. The mean is retained as a complementary descriptive statistic, while `observations` provides a basic measure of data coverage.

This aggregation preserves the OMI zoning structure instead of collapsing each municipality into a single quotation value.

## 4. Growth metrics

### Semester-over-semester growth

For a Municipality–Zone series:

```text
growth_pct = (P_t / P_(t-1) - 1) × 100
```

where `P` is the period median quotation midpoint.

### Year-over-year growth

The preferred comparison is the same semester in the previous year:

```text
YoY = (P_t / P_(t-1 year) - 1) × 100
```

Period alignment is preferred to simple row-position comparisons, particularly for series with missing or incomplete observations.

### Overall variation

```text
overall_change_pct = (last_price / first_price - 1) × 100
```

This measures the total change between the first and last available observations in a series.

### CAGR

```text
CAGR = (last_price / first_price)^(1 / years) - 1
```

CAGR expresses the equivalent annualised growth rate over the available observation window.

### Growth volatility

The standard deviation of calculable semester-over-semester growth rates is used as a descriptive measure of growth variability.

### Growth persistence

The analysis counts positive and negative growth periods and derives:

```text
positive_growth_share_pct =
    positive_periods / (positive_periods + negative_periods) × 100
```

Periods for which growth cannot be calculated are excluded from this denominator rather than being classified as negative growth.

## 5. Drawdown analysis

For each Municipality–Zone series, the workflow maintains a cumulative historical maximum:

```text
rolling_max = cumulative maximum of median_compr_mid
```

Drawdown is defined as:

```text
drawdown_pct = (median_compr_mid / rolling_max - 1) × 100
```

Two descriptive measures are particularly useful:

- `max_drawdown_pct` — the largest decline from a previous observed peak;
- `latest_drawdown_pct` — the latest observation relative to the historical maximum.

Both are calculated over the available history of each series.

## 6. National and regional distributions

Cross-sectional quotation distributions can be summarised by semester using percentiles such as P10, P25, P50, P75 and P90.

The P90–P10 spread provides a descriptive measure of the distance between the upper and lower parts of the quotation distribution.

Regional analysis additionally compares quotation levels, growth and observation counts across regions and over time.

## 7. Intra-municipality spatial dispersion

For each municipality and reference period, the analysis can compare the highest and lowest zone-level median quotation:

```text
spread_eur_m2 = max_zone_price - min_zone_price

spread_pct = (max_zone_price / min_zone_price - 1) × 100
```

This captures the internal spatial differentiation of OMI quotations within a municipality.

## 8. OMI transaction analysis

The transaction workflow constructs an annual municipality-level panel from OMI releases covering 2011–2025.

The pipeline first inventories available annual releases and validates the expected table set. It then harmonises year-specific schemas before joining the municipality dimension to transaction measures.

The municipality dimension is explicitly keyed by **year + municipality code (`codcom`)**. Join cardinality is validated before the final panel is accepted, and unmatched municipality-year records are reported rather than silently discarded.

For residential NTN data, the workflow also checks the reconciliation between size-class components and the reported total where the source structure allows that validation.

The resulting panel supports analysis of NTN and transaction-volume dynamics by municipality and year.

## 9. Population analysis

The population workflow constructs a municipality-year panel from ISTAT POSAS releases covering 2019–2026.

For the principal municipality population measure, the workflow uses the official `Età = 999` total row rather than unnecessarily summing all age-detail records.

The resulting panel supports demographic and geographic analysis and provides a controlled analytical input for potential future integration with real-estate indicators.

## 10. Statistical analysis

The repository includes statistical-analysis dependencies such as SciPy, statsmodels and scikit-posthocs. These tools are available for distributional analysis, statistical testing and post-hoc comparisons where the analytical question requires them.

Statistical tests should be interpreted in conjunction with sample size, data structure, multiple-comparison considerations and the observational nature of the underlying market data. A statistically significant difference is not, by itself, evidence of an economically material effect.

## 11. Comparability and limitations

Several limitations remain explicit throughout the project:

- OMI quotations are reference quotation ranges and are not equivalent to realised sale prices.
- `Compr_mid` is a derived midpoint and not a transaction observation.
- Municipality–Zone series can have different observation histories and coverage.
- Aggregate quotation statistics do not automatically control for changes in the composition of properties represented in the underlying OMI categories.
- Transaction volumes and population have different definitions, frequencies and coverage periods from OMI quotations.
- Cross-dataset relationships require explicit alignment of geographic keys, frequency and observation windows.

These limitations are part of the analytical design and should be considered when interpreting results.
