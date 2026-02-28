# FreqTrade Documentation: Advanced Strategy

> Source: https://www.freqtrade.io/en/stable/strategy-advanced/

## Persistent Data Storage

Store custom data that survives bot restarts using the trade's custom data API:

```python
trade.set_custom_data(key='my_key', value=my_value)
value = trade.get_custom_data(key='my_key', default=0)
entry = trade.get_custom_data_entry(key='my_key')  # full entry with metadata
```

Use simple types (bool, int, float, str) to avoid serialization issues and database bloat.

## Non-Persistent Storage (DEPRECATED)

Custom `custom_` prefixed dictionaries in the strategy class. Data does not survive bot restarts. This approach is deprecated in favor of the persistent data storage above.

## Dataframe Access in Callbacks

```python
dataframe, last_updated = self.dp.get_analyzed_dataframe(pair, self.timeframe)
current_candle = dataframe.iloc[-1]  # latest complete candle
```

Only works in callbacks, **not** in `populate_*()` functions.

## Enter Tags

Named signals defined in `populate_entry_trend()`:
- Limited to 255 characters
- Accessible via `trade.enter_tag`
- Multiple signals can concatenate tags

## Exit Tags

Defined in `populate_exit_trend()`:
- Limited to 100 characters
- Shown in backtest results

## Strategy Versioning

Implement `version()` returning a version string. FreqTrade does not keep historic versions.

## Derived Strategies

Inheritance pattern to override specific attributes/methods. Keep subclasses in separate files to avoid hyperopt parameter file issues.

## Strategy Embedding

BASE64 encode strategies for embedding in config:

```
"strategy": "StrategyName:BASE64String"
```

## Performance Optimization

Use `pd.concat(axis=1)` instead of sequential column assignment to avoid pandas fragmentation warnings, especially during hyperopt.

## Callback Methods Referenced

- `bot_loop_start()` -- called at the start of each bot loop iteration
- `adjust_entry_price()` -- adjust entry price for pending orders
- `custom_exit()` -- custom exit logic
- `confirm_trade_exit()` -- confirm or deny exits
- `populate_entry_trend()` -- define entry signals
- `populate_exit_trend()` -- define exit signals
- `populate_indicators()` -- compute indicators
- `version()` -- return strategy version string
