---
name: analyzing-time-series
description: >
  Diagnose and analyze time series data from a CSV file — stationarity, seasonality,
  trend, forecastability, and transform recommendations. Use this skill whenever a user
  shares time series data or wants to understand, explore, or prepare a series before
  forecasting. Trigger on phrases like "analyze my sales/temperature/metrics data",
  "does my data have seasonality?", "is this forecastable?", "should I difference this?",
  "what model should I use for this data?", "my data looks weird — can you check it?",
  or any request to understand patterns in a CSV with a date and a value column.
---

# Time Series Diagnostics

Comprehensive diagnostic toolkit to analyze time series data characteristics before forecasting.

## Input Format

The input CSV file should have two columns:
- **Date column** - Timestamps or dates (e.g., `date`, `timestamp`, `time`)
- **Value column** - Numeric values to analyze (e.g., `value`, `sales`, `temperature`)


## Workflow

**Step 1: Run diagnostics**

```bash
python scripts/diagnose.py data.csv --output-dir results/
```

This runs all statistical tests and produces `diagnostics.json` (raw metrics) and `summary.txt` (human-readable findings). Run this first — it also writes `diagnostics_state.json`, which Step 2 uses to keep plots synchronized with the same differencing order used here.

**Step 2: Generate plots (optional)**

```bash
python scripts/visualize.py data.csv --output-dir results/
```

Creates diagnostic plots in `results/plots/`. Must run after Step 1 so ACF/PACF plots reflect the stationarity results already computed. Column names are auto-detected, or can be specified with `--date-col` and `--value-col` options.

**Step 3: Report to user**

Read `references/interpretation.md` before writing your report — it contains the threshold tables needed to translate raw test statistics (ADF/KPSS p-values, seasonal strength scores, Ljung-Box results) into plain-language conclusions. Then summarize findings from `summary.txt` (prefer this over `diagnostics.json` — it is pre-formatted for direct communication) and present relevant plots. Cover:
- Is the data forecastable?
- Is it stationary? How much differencing is needed?
- Is there seasonality? What period?
- Is there a trend? What direction?
- Is a transform needed?

## Script Options

Both scripts accept:
- `--date-col NAME` - Date column (auto-detected if omitted)
- `--value-col NAME` - Value column (auto-detected if omitted)
- `--output-dir PATH` - Output directory (default: `diagnostics/`)
- `--seasonal-period N` - Seasonal period (auto-detected if omitted)

## Output Files

```
results/
├── diagnostics.json       # All test results and statistics
├── summary.txt            # Human-readable findings
├── diagnostics_state.json # Internal state for plot synchronization
└── plots/
    ├── timeseries.png
    ├── histogram.png
    ├── rolling_stats.png
    ├── box_by_dayofweek.png  # By day of week (if applicable)
    ├── box_by_month.png      # By month (if applicable)
    ├── box_by_quarter.png    # By quarter (if applicable)
    ├── acf_pacf.png
    ├── decomposition.png
    └── lag_scatter.png
```

## References

See `references/interpretation.md` for:
- Statistical test thresholds and interpretation
- Seasonal period guidelines by data frequency
- Transform recommendations

## Scripts

- `scripts/diagnose.py` — runs all statistical tests; primary entry point
- `scripts/visualize.py` — generates diagnostic plots; depends on `diagnostics_state.json` from `diagnose.py`
- `scripts/ts_utils.py` — shared utilities (data loading, frequency detection, stationarity tests); required by both scripts, do not move or delete

## Dependencies

`pandas`, `numpy`, `matplotlib`, `statsmodels`, `scipy`