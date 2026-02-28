# FreqTrade Documentation: Advanced Backtesting Analysis

> Source: https://www.freqtrade.io/en/stable/advanced-backtesting/

## Overview

Advanced backtesting analysis provides detailed examination of entry/exit signals from backtesting runs, allowing inspection of performance across different buy/sell conditions with indicator value analysis.

## Commands

### Run Backtest with Signal Export

```bash
freqtrade backtesting -c config.json --timeframe 5m --strategy MyStrategy \
  --timerange=20210101-20210201 --export=signals
```

### Run Analysis with Grouping

```bash
freqtrade backtesting-analysis -c config.json --analysis-groups 0 1 2 3 4 5
```

## Analysis Group Levels

| Group | Description |
|-------|-------------|
| 0 | Overall winrate and profit by entry tag |
| 1 | Profit summaries grouped by entry tag |
| 2 | Profit summaries by entry AND exit tags |
| 3 | Profit summaries by pair AND entry tag |
| 4 | Profit summaries by pair, entry tag, AND exit tag |
| 5 | Profit summaries grouped by exit tag |

## Key Parameters

| Parameter | Description |
|-----------|-------------|
| `--export=signals` | Generates pickle files with strategy signals and trade data in `user_data/backtest_results/` |
| `--backtest-filename` | Specify particular result files for analysis |
| `--enter-reason-list` | Filter analysis by specific entry tags |
| `--exit-reason-list` | Filter analysis by specific exit tags |
| `--timerange` | Filter trades by date range |
| `--indicator-list` | Display indicator values (e.g., `rsi bb_lowerband ema_9 macd`); use `all` for all entry-point indicators |
| `--entry-only` | Show indicator values at entry points only |
| `--exit-only` | Show indicator values at exit points only |
| `--rejected-signals` | Print rejected trade signals |
| `--analysis-to-csv` | Export results to CSV format |
| `--analysis-csv-path` | Specify CSV output path |

## Auto-Accessible Fields

The following fields are automatically available in analysis output:

- `open_date` -- Trade open timestamp
- `close_date` -- Trade close timestamp
- `min_rate` -- Minimum price during trade
- `max_rate` -- Maximum price during trade
- `open` / `close` / `high` / `low` -- OHLC prices
- `volume` -- Trading volume
- `profit_ratio` -- Profit as a ratio
- `profit_abs` -- Absolute profit value

## Practical Usage Notes

- Signal export is **only available for backtesting, not hyperopt**.
- Generated pickle files can be **large**; clean `user_data/backtest_results` periodically.
- Use `--cache none` before new backtests to prevent stale cached results from being used.
- Indicator names are automatically displayed with `(entry)` and `(exit)` suffixes to distinguish values at different trade points.
- Combining multiple analysis groups in a single command provides a comprehensive view of strategy performance.
- Entry and exit tags must be set in the strategy code using `enter_tag` and `exit_tag` for the grouping to be meaningful.

## Example Workflow

```bash
# Step 1: Run backtest with signal export
freqtrade backtesting -c config.json --timeframe 5m --strategy MyStrategy \
  --timerange=20210101-20210201 --export=signals --cache none

# Step 2: Analyze overall performance by entry tag
freqtrade backtesting-analysis -c config.json --analysis-groups 0 1

# Step 3: Drill down into specific entry reasons with indicators
freqtrade backtesting-analysis -c config.json --analysis-groups 2 \
  --enter-reason-list "rsi_cross" --indicator-list rsi bb_lowerband

# Step 4: Export detailed results to CSV
freqtrade backtesting-analysis -c config.json --analysis-groups 0 1 2 3 4 5 \
  --analysis-to-csv --analysis-csv-path user_data/analysis_results/

# Step 5: Check rejected signals
freqtrade backtesting-analysis -c config.json --rejected-signals
```
