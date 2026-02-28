# FreqTrade Documentation: Backtesting

> Source: https://www.freqtrade.io/en/stable/backtesting/

## Key Concepts Covered

- Strategy validation using historical data, CLI commands, output interpretation, accuracy improvements, assumptions/limitations, and multi-strategy comparison.

## Basic Command

```
freqtrade backtesting --strategy AwesomeStrategy
```

## Key Flags

| Flag | Purpose |
|------|---------|
| `-s / --strategy NAME` | Strategy to test |
| `--strategy-list S1 S2` | Compare multiple strategies |
| `-i / --timeframe` | Candle timeframe (1m, 5m, 1h, etc.) |
| `--timerange 20190501-` | Date range for test |
| `--dry-run-wallet / --starting-balance` | Initial capital |
| `-p / --pairs PAIR ...` | Limit tested pairs |
| `--max-open-trades N` | Concurrent position limit |
| `--stake-amount` | Override stake configuration |
| `--fee FLOAT` | Custom fee ratio (applied twice: entry + exit) |
| `--timeframe-detail 5m` | Smaller timeframe for intra-candle accuracy |
| `--export {none,trades,signals}` | Output format |
| `--breakdown {day,week,month,year,weekday}` | Period analysis |
| `--cache {none,day,week,month}` | Result caching behavior |
| `--enable-protections` | Include protection plugins |
| `--enable-dynamic-pairlist` | Dynamic pair refreshing |

## Output Metrics

The backtesting report includes:

- Per-pair statistics: trade count, avg profit %, total profit (stake currency), total profit %, avg duration, win/loss/draw
- Left open trades (force-exited at end)
- Enter tag breakdown and exit reason analysis
- Summary metrics: absolute profit, total %, CAGR, Sortino ratio, Sharpe ratio, Calmar ratio, SQN, profit factor, expectancy, max drawdown with dates, consecutive win/loss streaks, market change comparison

## Breakdown Reports

`--breakdown day week month`: Show daily/weekly/monthly/yearly profit tables with trade counts and win rates.

## Backtesting Assumptions (Critical)

- All orders fill at requested price (no slippage), as long as price is within the candle's high/low range.
- Entries occur at candle open (unless custom pricing is used).
- Exit-signal exits trigger at the next candle's open.
- ROI comparisons use candle high, but exits respect candle range.
- Stoploss triggers at exact price; trailing stoploss adjusts on candle high.
- **Evaluation order:** Exit-signal -> Stoploss -> ROI -> Trailing stoploss.

## Improved Accuracy with `--timeframe-detail`

```
freqtrade backtesting --strategy MyStrat --timeframe 1h --timeframe-detail 5m
```

This runs the strategy at 1h intervals but evaluates active trades at 5-minute granularity, providing much better simulation of intra-candle behavior at the cost of increased memory and runtime.

## Dynamic Stake Amounts

Setting `stake_amount` to `"unlimited"` distributes starting capital evenly across `max_open_trades`, allowing profit compounding.

## Result Caching

Backtesting caches identical runs within the last day by default. Disabled for open-ended timeranges. Controllable via `--cache`.

## Practical Notes

- Starting balance must exceed `stake_amount` for the simulation to work.
- Dynamic pairlists compromise backtesting reproducibility.
- Always validate with dry-run before going live -- "past results don't guarantee future success."
- Multi-strategy comparison via `--strategy-list` produces a summary table for side-by-side evaluation.
- The `--export signals` option saves detailed signal data for post-analysis.
- Exchange minimum/maximum trade amounts are enforced, but historical precision limits are unknown, potentially inflating minimums.
