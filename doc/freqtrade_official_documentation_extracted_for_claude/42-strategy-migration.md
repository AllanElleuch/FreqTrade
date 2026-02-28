# FreqTrade Documentation: Strategy Migration

> Source: https://www.freqtrade.io/en/stable/strategy_migration/

## Overview

The V2-to-V3 migration is primarily driven by the need to support short trades and leveraged trading. The core change is a terminology shift from "buy/sell" to "entry/exit" throughout the entire codebase.

## Method Renamings

| Old (V2) | New (V3) |
|---|---|
| `populate_buy_trend()` | `populate_entry_trend()` |
| `populate_sell_trend()` | `populate_exit_trend()` |
| `custom_sell()` | `custom_exit()` |
| `check_buy_timeout()` | `check_entry_timeout()` |
| `check_sell_timeout()` | `check_exit_timeout()` |

## DataFrame Column Changes

| Old | New |
|---|---|
| `buy` | `enter_long` |
| `sell` | `exit_long` |
| `buy_tag` | `enter_tag` |
| (new) | `enter_short`, `exit_short` |

## Important Configuration Changes

- `order_time_in_force`: `{"buy": "gtc", "sell": "gtc"}` becomes `{"entry": "GTC", "exit": "GTC"}` (note: uppercase values)
- `order_types`: `"buy"`, `"sell"`, `"emergencysell"`, `"forcesell"`, `"forcebuy"` become `"entry"`, `"exit"`, `"emergency_exit"`, `"force_exit"`, `"force_entry"`
- `use_sell_signal` becomes `use_exit_signal`; `sell_profit_only` becomes `exit_profit_only`; `sell_profit_offset` becomes `exit_profit_offset`; `ignore_roi_if_buy_signal` becomes `ignore_roi_if_entry_signal`
- `unfilledtimeout`: `{"buy": 10, "sell": 10}` becomes `{"entry": 10, "exit": 10}`
- `bid_strategy` becomes `entry_pricing`; `ask_strategy` becomes `exit_pricing`
- Webhook keys: `webhookbuy` becomes `entry`, `webhooksell` becomes `exit`, etc.
- Telegram notifications: `buy` becomes `entry`, `sell` becomes `exit`, etc.

## Code Patterns (Old vs New)

```python
# Old V2
def populate_buy_trend(self, dataframe, metadata):
    dataframe.loc[conditions, ['buy', 'buy_tag']] = (1, 'rsi_cross')

# New V3
def populate_entry_trend(self, dataframe, metadata):
    dataframe.loc[conditions, ['enter_long', 'enter_tag']] = (1, 'rsi_cross')
```

## New Method Signatures

New method signatures add a `side: str` parameter (value: `"long"` or `"short"`) to:
- `custom_stake_amount()`
- `confirm_trade_entry()`
- `custom_entry_price()`

## Helper Method Changes

`stoploss_from_open()` and `stoploss_from_absolute()` now require `is_short=trade.is_short` and optionally `leverage=trade.leverage`.

## FreqAI Migration

The `populate_any_indicators()` method was split into four dedicated methods:

1. `feature_engineering_expand_all()` -- features auto-expanded across indicator periods
2. `feature_engineering_expand_basic()` -- basic features expanded across timeframes
3. `feature_engineering_standard()` -- standard features without auto-expansion
4. `set_freqai_targets()` -- target variable definitions

Features prefixed with `%` are auto-expanded; targets prefixed with `&` are recognized as labels. The `{pair}` and `{timeframe}` placeholders in feature names are no longer needed.

## Exit Reason Changes

| Old | New |
|---|---|
| `sell_signal` | `exit_signal` |
| `custom_sell` | `custom_exit` |
| `force_sell` | `force_exit` |
| `emergency_sell` | `emergency_exit` |

Both `sell_reason` and `exit_reason` are provided temporarily in `confirm_trade_exit()` for backward compatibility.

## Practical Notes

Set `INTERFACE_VERSION = 3` in your strategy class to indicate V3 compatibility.
