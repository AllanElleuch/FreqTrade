# FreqTrade Documentation: Bot Basics

> Source: https://www.freqtrade.io/en/stable/bot-basics/

## Key Concepts

- **Strategy:** Python class defining trading behavior.
- **Pair naming:** Spot uses `base/quote` (e.g., ETH/USDT); Futures uses `base/quote:settle` (e.g., ETH/USDT:USDT). Incorrect naming causes failures.
- **Fees:** All profit calculations include fees. Backtesting and dry-run use exchange defaults; live uses actual fees including rebates.
- **Order types:** Limit orders (execute at specified price or better) vs. market orders (guaranteed fill at current price).

## Bot Execution Loop (Live/Dry-Run)

Each cycle (default: every few seconds, controlled by `internals.process_throttle_secs`):

1. Retrieve active trades from storage
2. Calculate tradable pair list
3. Download OHLCV market data
4. Execute strategy callbacks: `bot_loop_start()`, `populate_indicators()`, `populate_entry_trend()`, `populate_exit_trend()`
5. Update open order states
6. Evaluate exit signals (stoploss, ROI, custom exits)
7. Check position adjustments
8. Verify trade slot availability (`max_open_trades`)
9. Evaluate entry signals and confirm new positions

## Backtesting Execution

1. Load historical OHLCV data
2. Execute `bot_start()` once
3. Calculate indicators and signals for all pairs
4. Simulate candle-by-candle: check timeouts, process signals, confirm entries/exits, evaluate stoploss

## Practical Notes

- Critical difference: backtesting calls callbacks once per candle, while live mode calls them roughly every 5 seconds. This can cause result mismatches.
- Custom fees can be set via the `--fee` argument for backtesting.
- `entry_pricing` and `exit_pricing` settings control order placement behavior.
