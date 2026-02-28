# FreqTrade Documentation: Strategy Callbacks

> Source: https://www.freqtrade.io/en/stable/strategy-callbacks/

## Key Concepts

- Callbacks are special functions invoked "whenever needed" during bot operation, as opposed to vectorized populate methods that run once over the full DataFrame in backtesting.
- Avoid heavy computations in callbacks to prevent operational delays.

## Complete Callback List

| Callback | Purpose |
|---|---|
| `bot_start()` | Runs once at strategy initialization |
| `bot_loop_start()` | Runs each bot iteration (~every 5 seconds live) |
| `custom_stake_amount()` | Dynamic position sizing before trade entry |
| `custom_exit()` | Custom exit signals per trade (profit thresholds, duration) |
| `custom_stoploss()` | Dynamic trailing stoploss logic |
| `custom_roi()` | Minimum ROI thresholds (works alongside `minimal_roi`) |
| `custom_entry_price()` / `custom_exit_price()` | Custom order pricing |
| `check_entry_timeout()` / `check_exit_timeout()` | Unfilled order timeout management |
| `adjust_order_price()` | Refresh limit order prices on new candles |
| `confirm_trade_entry()` / `confirm_trade_exit()` | Final boolean approval before order placement |
| `adjust_trade_position()` | DCA, position increases/decreases |
| `leverage()` | Per-trade leverage customization |
| `order_filled()` | React to filled orders |
| `plot_annotations()` | Add chart visualizations (area, line, point) |

## Exit Logic Hierarchy

1. `populate_exit_trend()` -- signal-based vectorized exits using indicators
2. `custom_exit()` -- per-trade exits using trade-specific data (profit, duration)
3. `custom_stoploss()` -- dynamic trailing stoploss; compatible with on-exchange stoploss
4. `custom_roi()` -- minimum profit thresholds; when both `minimal_roi` and `custom_roi` exist, the *lower* value triggers the exit

## Custom Stoploss Details

- Returns percentage relative to current price
- Values only move upward (stoploss can only tighten, never widen) unless `after_fill=True`
- The strategy-level `stoploss` is the absolute floor
- Helper: `stoploss_from_open(open_relative_stop, current_profit)` converts entry-relative stoploss to current-price-relative
- Helper: `stoploss_from_absolute(absolute_price, current_rate, is_short)` converts absolute price to percentage
- With leverage, the returned value represents the risk for the trade

## Position Adjustment (`adjust_trade_position`)

- Requires `position_adjustment_enable = True`
- Return positive values to increase position, negative to decrease
- Partial exit formula: `amount = negative_stake_amount * trade.amount / trade.stake_amount`
- Should implement `custom_stake_amount()` to reserve funds for DCA
- Increases memory usage; dozens of adjustments per trade are discouraged

## Confirmation Callbacks

- `confirm_trade_entry()` and `confirm_trade_exit()` return boolean (True = proceed, False = reject)
- WARNING: Rejecting exits in `confirm_trade_exit()` can prevent stoploss execution, risking significant losses

## Return Value Conventions

- `None` = "no action" or fall back to defaults
- `NaN` and `inf` treated as `None`
- Zero or `None` from `custom_stake_amount()` prevents trade placement
- Invalid custom prices are clamped to `custom_price_max_distance_ratio` (default 2%)

## Practical Notes

- Backtesting calls `adjust_trade_position()` once per candle, but live mode calls it every iteration, potentially causing discrepancies.
- `bot_loop_start()` is useful for pair-independent logic (e.g., global state updates).
- `order_filled()` is ideal for post-fill housekeeping.
