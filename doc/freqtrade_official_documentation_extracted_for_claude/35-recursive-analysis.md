# FreqTrade Documentation: Recursive Analysis

> Source: https://www.freqtrade.io/en/stable/recursive-analysis/

## Key Concepts

Recursive formulas (where each term depends on preceding terms) cause indicators to produce different values depending on how much historical data is available. An indicator calculated on 1000 candles gives different values than the same indicator on 500 candles. This creates discrepancies between backtesting (full history) and live trading (limited API data). The tool tests indicators across multiple startup candle counts to quantify these variances.

## Command

```
freqtrade recursive-analysis -s StrategyName -i 5m --timerange 20230101-20231231
```

## Key Options

- `-p, --pairs` -- defaults to first whitelist pair
- `-i, --timeframe` -- candle interval
- `--timerange` -- data range (minimum 5000 candles recommended)
- `--startup-candle` -- startup candle counts to test (e.g., 199, 499, 999, 1999)
- `--data-format-ohlcv` -- storage format
- `-s, --strategy` -- strategy class name

## How It Works (3 phases)

1. **Benchmark**: Compute indicators on full supplied timerange
2. **Multi-Run**: Recalculate with different startup candle counts
3. **Variance Report**: Compare last-row indicator values across runs

## Output Interpretation

- `nan%` -- insufficient data for calculation
- `-` -- zero variance (perfect consistency)
- Decimal percentages -- variance magnitude

## Practical Notes

- Only analyzes indicators in `populate_indicators()` and `@informative` decorators; indicators in entry/exit trend functions are ignored.
- Use volatile pairs (BTC, ETH) to minimize rounding errors.
- Minimum 5000 candles recommended for reliable benchmarking.
- Focus on whether variance affects trading decisions, not on achieving absolute zero.
- Cache is forced to "none" automatically.
