# FreqTrade Documentation: Lookahead Analysis

> Source: https://www.freqtrade.io/en/stable/lookahead-analysis/

## Key Concepts

Lookahead bias occurs when a strategy references future candle data during backtesting that would not be available in live trading. This is a common cause of "too good to be true" backtest results. The tool performs a baseline backtest, then runs verification backtests for individual entries/exits separately, comparing dataframes to detect value differences.

## Command

```
freqtrade lookahead-analysis -s StrategyName -i 5m --timerange 20230101-20231231
```

## Key Options

- `-s, --strategy NAME` -- strategy class
- `-i, --timeframe` -- candle interval
- `-p, --pairs` -- limit to specific pairs
- `--timerange` -- data range
- `--minimum-trade-amount INT` -- minimum trades required
- `--targeted-trade-amount INT` -- target trade count
- `--lookahead-analysis-exportfilename` -- CSV export path
- `--allow-limit-orders` -- override forced market orders

## Forced Configuration

The command automatically sets:
- Cache to "none"
- `max-open-trades` >= number of pairs
- `dry-run-wallet` to 1 billion
- `stake-amount` to 10,000
- Disables protections
- Forces market orders (unless `--allow-limit-orders` is used)

## Common Lookahead Bias Patterns to Watch For

- `shift(-10)` or any negative shift (looking forward)
- Direct `iloc[]` row access
- For-loops without proper index control
- Aggregation functions (`.mean()`, `.min()`, `.max()`) on entire columns instead of rolling windows
- `ta.MACD()` with `signalperiod=1`

## Results Columns

- `filename` -- strategy file
- `strategy` -- strategy name
- `has_bias` -- Yes/No
- `total_signals` -- total signal count
- `biased_entry_signals` -- count of biased entry signals
- `biased_exit_signals` -- count of biased exit signals
- `biased_indicators` -- list of biased indicators

## Practical Notes

- Only **triggered** signals are checked; untriggered signals may hide bias (false negatives).
- Limit orders with `custom_entry_price()`/`custom_exit_price()` can cause false positives.
- FreqAI target indicators from `set_freqai_targets()` are falsely flagged -- ignore them.
- Fix entry signal bias first, then exit signals, since biased entries cascade to biased exits.
- Removing biased components usually degrades performance significantly.
