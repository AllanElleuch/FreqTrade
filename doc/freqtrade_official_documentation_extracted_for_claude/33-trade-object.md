# FreqTrade Documentation: Trade Object

> Source: https://www.freqtrade.io/en/stable/trade-object/

## Key Concepts

The Trade object is the fundamental data structure representing a position that FreqTrade enters. It is persisted to the database and passed to strategy callbacks, but it **cannot be modified directly** -- modifications happen based on callback return values.

## Important Properties

### Basic Info
- `pair` -- trading pair
- `exchange` -- exchange name
- `strategy` -- strategy name
- `timeframe` -- candle timeframe
- `enter_tag` -- entry signal tag
- `exit_reason` -- reason for exit

### Price Data
- `open_rate` -- entry price (averaged if position adjusted)
- `close_rate` -- exit price
- `open_rate_requested` -- requested entry price
- `close_rate_requested` -- requested exit price
- `safe_close_rate` -- safe variant of close rate

### Position Info
- `is_open` -- whether the trade is currently open
- `is_short` -- whether the trade is a short position
- `stake_amount` -- amount of stake currency invested
- `amount` -- amount of the traded asset
- `leverage` -- leverage multiplier (default 1.0)
- `max_rate` -- highest rate seen during the trade
- `min_rate` -- lowest rate seen during the trade

### Financial Metrics
- `close_profit` -- relative profit (0.01 = 1%)
- `close_profit_abs` -- absolute profit in stake currency
- `realized_profit` -- profit realized while trade is still open

### Order Tracking
- `orders` -- all orders associated with the trade
- `open_orders` -- non-stoploss active orders
- `open_sl_orders` -- stoploss active orders
- `nr_of_successful_entries` -- count of successful entry orders
- `nr_of_successful_exits` -- count of successful exit orders
- `has_open_position` -- whether there is an open position
- `has_open_orders` -- whether there are open orders

### Stop Loss
- `stop_loss` -- current stop loss price
- `stop_loss_pct` -- stop loss as percentage
- `initial_stop_loss` -- initial stop loss price
- `initial_stop_loss_pct` -- initial stop loss as percentage
- `stoploss_last_update_utc` -- last time stop loss was updated
- `stoploss_or_liquidation` -- the most restrictive level (whichever is closer)

### Futures/Margin
- `liquidation_price` -- liquidation price for futures
- `interest_rate` -- interest rate for margin trading
- `funding_fees` -- accumulated funding fees

### Timestamps
- `open_date_utc` -- trade open time (always use the UTC variants)
- `close_date_utc` -- trade close time (always use the UTC variants)

## Class Methods (available in backtesting and live)

```python
from freqtrade.persistence import Trade

# Query trades with filters
trade_hist = Trade.get_trades_proxy(pair='ETH/USDT', is_open=False,
                                     open_date=current_date - timedelta(days=2))
open_trades = Trade.get_open_trade_count()
profit = Trade.get_total_closed_profit()
total_stake = Trade.total_open_trades_stakes()
```

## Live/Dry-Run Only Methods

These methods are **not** available in backtesting/hyperopt:

```python
if self.config['runmode'].value in ('live', 'dry_run'):
    performance = Trade.get_overall_performance()  # per-pair stats
    volume = Trade.get_trading_volume()
```

## Order Object Properties

- `order_id` -- exchange order ID
- `ft_pair` -- trading pair
- `ft_order_side` -- buy or sell
- `order_type` -- market, limit, etc.
- `status` -- order status
- `price` -- order price
- `average` -- average fill price
- `amount` -- order amount
- `filled` -- amount filled
- `remaining` -- amount remaining
- `cost` -- total cost
- `stop_price` -- stop price (for stop orders)
- `stake_amount` -- stake amount

### Safe Variants

"Safe" variants guarantee non-null returns by falling through multiple fields:
- `safe_filled`
- `safe_price`
- `safe_cost`
- `safe_amount_after_fee`

## Practical Notes

- Class methods work in callbacks but **not** in `populate_*()` methods.
- Trade attributes are read-only; your callbacks return values that Freqtrade uses to modify trades.
- Prefer the `safe_` property variants when dealing with exchange data.
