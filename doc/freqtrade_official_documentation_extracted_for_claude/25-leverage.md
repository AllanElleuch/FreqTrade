# FreqTrade Documentation: Leverage (Short / Leverage Trading)

> Source: https://www.freqtrade.io/en/stable/leverage/

## Key Concepts

FreqTrade supports three trading modes:

- **Spot**: Long only, no leverage.
- **Margin**: Currently unavailable.
- **Futures**: Perpetual swap contracts with leverage.

Leverage trading amplifies both profits and losses; asset value can drop to 0 rapidly.

To enable short trading, strategies must set `can_short = True` as a class variable and trading mode must be `margin` or `futures`.

## Configuration Options

| Option | Values | Description |
|--------|--------|-------------|
| `"trading_mode"` | `"spot"` (default), `"margin"`, `"futures"` | Sets the trading mode |
| `"margin_mode"` | `"isolated"`, `"cross"` | `"isolated"` = separate collateral per pair (lower risk); `"cross"` = shared collateral (higher liquidation risk) |
| `"liquidation_buffer"` | `0.05` default (range 0.0-0.99) | Controls artificial liquidation threshold distance from the real liquidation price |
| `"futures_funding_rate"` | optional | Config for backtesting when exchange funding rate data is unavailable |

## Pair Naming Convention

Futures pair naming follows the CCXT convention:

```
base/quote:settle
```

Example: `ETH/USDT:USDT`

## Liquidation Formula

```
freqtrade_liquidation_price = liquidation_price +/- (abs(open_rate - liquidation_price) * liquidation_buffer)
```

- The `+/-` depends on whether the position is long or short.
- The buffer creates an artificial liquidation price closer to the entry than the real one, providing a safety margin.

## Leverage Configuration

- Leverage defaults to **1x**.
- Dynamic adjustment is possible through a strategy leverage callback.
- The leverage callback allows the strategy to set leverage per-trade based on conditions.

## Important Warnings and Practical Notes

- You **cannot** run multiple bots on one leveraged account simultaneously, as liquidation calculations assume exclusive account control.
- Cross-margin influence is **not** fully simulated in dry-run or backtesting.
- Setting false funding rates for missing historical data produces **inaccurate** backtests.
- FreqTrade does **not** currently account for liquidation fees. Using low buffer values without `stoploss_on_exchange` risks inaccurate P/L tracking.
- Funding fees and margin interest are included in `current_profit` calculations.
- When using futures, ensure your exchange account is set to the correct position mode (typically "One-way Mode" and "Single-Asset Mode" for Binance).
