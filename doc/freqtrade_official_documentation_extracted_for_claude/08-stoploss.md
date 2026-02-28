# FreqTrade Documentation: Stoploss

> Source: https://www.freqtrade.io/en/stable/stoploss/

## Key Concepts

- Stoploss defines a loss ratio triggering automatic exits. Example: `-0.10` exits when loss reaches 10%.
- All stoploss calculations include fees.
- With leverage, stoploss represents trade risk: a 10% stoploss on 10x leverage triggers at just 1% price movement.

## Four Stoploss Types

### 1. Static Stoploss

Fixed percentage; triggers at defined loss level.

```python
stoploss = -0.10
```

### 2. Trailing Stoploss

Adjusts upward with price increases, maintaining a consistent loss percentage from the highest point.

```python
stoploss = -0.10
trailing_stop = True
```

### 3. Trailing with Positive Offset

Activates a different (tighter) trailing percentage after reaching a profitability threshold.

```python
stoploss = -0.10
trailing_stop = True
trailing_stop_positive = 0.02
trailing_stop_positive_offset = 0.03
```

### 4. Offset-Triggered Trailing

Remains completely static until the specified profit offset is reached, then begins trailing.

```python
stoploss = -0.10
trailing_stop = True
trailing_stop_positive = 0.02
trailing_stop_positive_offset = 0.03
trailing_only_offset_is_reached = True
```

## On-Exchange Stoploss Configuration

```python
order_types = {
    'stoploss_on_exchange': True,
    'stoploss_on_exchange_limit_ratio': 0.99,
    'stoploss_on_exchange_interval': 60,
    'stoploss_price_type': 'last',  # Futures only: "last", "mark", or "index"
}
```

## Supported Exchanges for On-Exchange Stoploss

Binance, Bingx, Bitget, Bybit, Gate.io, HTX, Hyperliquid, Kraken, OKX, Kucoin, and others (with varying spot/futures and margin mode support).

## Special Order Type Overrides

- `emergency_exit`: Defaults to "market"; used when on-exchange stoploss orders fail
- `force_exit` / `force_entry`: Optional overrides for `/forceexit` and `/forceentry` commands

## Practical Notes

- Do not set stoploss too tight when using on-exchange stoploss, as the limit order may not fill during fast price drops.
- Reload configuration via `/reload_config` to apply updated stoploss values to existing open trades.
- Once trailing stoploss adjustments activate, the stoploss level cannot be modified further (it only tightens).
- The `stoploss_on_exchange_limit_ratio` of 0.99 means the limit order is placed at 99% of the stop trigger price, giving 1% slippage room.
- The `stoploss_on_exchange_interval` (default 60 seconds) controls how frequently the bot checks and updates the on-exchange stoploss order.
