# FreqTrade Strategy Development Reference

## IStrategy Interface (v3)
```python
class MyStrategy(IStrategy):
    INTERFACE_VERSION = 3
    can_short = True  # Enable short trading (futures)

    # Core parameters
    timeframe = '15m'
    stoploss = -0.10
    minimal_roi = {"60": 0.01, "30": 0.02, "0": 0.04}
    trailing_stop = False
    trailing_stop_positive = 0.01
    trailing_stop_positive_offset = 0.02
    trailing_only_offset_is_reached = True
    startup_candle_count = 200
    process_only_new_candles = True
    use_exit_signal = True
    exit_profit_only = False
    ignore_roi_if_entry_signal = False

    # Order types
    order_types = {
        'entry': 'limit', 'exit': 'limit',
        'stoploss': 'market', 'stoploss_on_exchange': False
    }
```

## Required Methods

### populate_indicators(dataframe, metadata) → DataFrame
Add all TA indicators to the dataframe. Called once per pair per candle.
```python
def populate_indicators(self, dataframe, metadata):
    dataframe['rsi'] = ta.RSI(dataframe, timeperiod=14)
    dataframe['ema_20'] = ta.EMA(dataframe, timeperiod=20)
    return dataframe
```

### populate_entry_trend(dataframe, metadata) → DataFrame
Set entry signals by writing to 'enter_long', 'enter_short' columns (1 = enter).
```python
def populate_entry_trend(self, dataframe, metadata):
    dataframe.loc[
        (dataframe['rsi'] < 30) & (dataframe['volume'] > 0),
        'enter_long'] = 1
    dataframe.loc[
        (dataframe['rsi'] > 70) & (dataframe['volume'] > 0),
        'enter_short'] = 1
    return dataframe
```

### populate_exit_trend(dataframe, metadata) → DataFrame
Set exit signals by writing to 'exit_long', 'exit_short' columns.
```python
def populate_exit_trend(self, dataframe, metadata):
    dataframe.loc[
        (dataframe['rsi'] > 70),
        'exit_long'] = 1
    dataframe.loc[
        (dataframe['rsi'] < 30),
        'exit_short'] = 1
    return dataframe
```

## Key Callbacks

### confirm_trade_entry(pair, order_type, amount, rate, time_in_force, current_time, entry_tag, side) → bool
Return False to reject entry. Use for additional validation.

### confirm_trade_exit(pair, trade, order_type, amount, rate, time_in_force, exit_reason, current_time, **kwargs) → bool
Return False to reject exit. Can prevent unwanted exits.

### custom_stoploss(pair, trade, current_time, current_rate, current_profit, after_fill) → float
Return custom stoploss value (negative float, e.g., -0.05). Return None to use default.
Called on every tick. Can implement trailing logic, time-based stops, indicator-based stops.

### custom_exit(pair, trade, current_time, current_rate, current_profit, **kwargs) → Optional[str|bool]
Return exit reason string or True to exit, False/None to hold.

### custom_stake_amount(pair, current_time, current_rate, proposed_stake, min_stake, max_stake, leverage, entry_tag, side) → float
Return custom stake amount. Useful for position sizing.

### leverage(pair, current_time, current_rate, proposed_leverage, max_leverage, entry_tag, side) → float
Return leverage to use (1.0 = no leverage). Only for futures.

### adjust_trade_position(pair, trade, current_time, current_rate, current_profit, min_stake, max_stake, current_entry_rate, current_exit_rate, current_entry_profit, current_exit_profit) → Optional[float]
Return positive value to increase position (DCA), negative to decrease. Return None for no change.

### custom_entry_price(pair, trade, current_time, proposed_rate, entry_tag, side) → float
Return custom entry price (limit order price).

### custom_exit_price(pair, trade, current_time, proposed_rate, exit_tag, side) → float
Return custom exit price.

### bot_start(self) → None
Called once at bot startup. Initialize resources here.

### bot_loop_start(self) → None
Called at the beginning of every bot iteration loop.

## Hyperoptable Parameters
```python
from freqtrade.strategy import IntParameter, DecimalParameter, BooleanParameter, CategoricalParameter

class MyStrategy(IStrategy):
    buy_rsi = IntParameter(10, 40, default=30, space="buy", optimize=True)
    sell_rsi = IntParameter(60, 90, default=70, space="sell", optimize=True)
    buy_ema_period = CategoricalParameter([10, 20, 50], default=20, space="buy")
    use_trailing = BooleanParameter(default=True, space="buy")
    risk_factor = DecimalParameter(0.01, 0.10, default=0.05, decimals=2, space="buy")

    def populate_entry_trend(self, dataframe, metadata):
        dataframe.loc[
            (dataframe['rsi'] < self.buy_rsi.value),
            'enter_long'] = 1
        return dataframe
```

## Entry/Exit Tags
```python
# In populate_entry_trend:
dataframe.loc[condition, 'enter_tag'] = 'rsi_oversold'

# In populate_exit_trend:
dataframe.loc[condition, 'exit_tag'] = 'rsi_overbought'
```

## Informative Pairs (Multi-Timeframe)
```python
def informative_pairs(self):
    return [("BTC/USDT", "1h"), (f"{self.config['stake_currency']}/USDT", "1d")]

# Or use the decorator:
from freqtrade.strategy import informative
@informative('1h')
def populate_indicators_1h(self, dataframe, metadata):
    dataframe['rsi_1h'] = ta.RSI(dataframe, timeperiod=14)
    return dataframe
```

## Stoploss Types
1. **Fixed:** `stoploss = -0.10` (10% below entry)
2. **Trailing:** `trailing_stop = True` + `trailing_stop_positive` + offset
3. **Custom:** `custom_stoploss()` callback - can use indicators, time, profit
4. **Stoploss on exchange:** `order_types['stoploss_on_exchange'] = True`

## Common Patterns from This Repo
- **Ichimoku Cloud filter:** Price above/below senkou_a and senkou_b for trend
- **EMA fan magnitude:** Ratio of short EMA to long EMA for trend strength
- **Heikinashi candles:** Smoothed candles for trend detection
- **Volume guard:** Always include `volume > 0` in conditions

## Common Pitfalls
- Forgetting `volume > 0` guard (zero-volume candles cause issues)
- Not setting `startup_candle_count` high enough for indicators
- Using `.value` outside of entry/exit methods (breaks hyperopt)
- Modifying Trade objects directly (use callbacks instead)
- Lookahead bias: never use future data in indicators
- Repainting: some indicators change historical values

## Detailed Documentation
See files in `/mnt/nvme/FreqTrade/repo/doc/freqtrade_official_documentation_extracted_for_claude/`:
- `05-strategy-101.md` - IStrategy basics
- `06-strategy-customization.md` - Advanced patterns
- `07-strategy-callbacks.md` - All callbacks
- `08-stoploss.md` - Stoploss configuration
- `33-trade-object.md` - Trade object reference
- `36-strategy-advanced.md` - Advanced techniques
