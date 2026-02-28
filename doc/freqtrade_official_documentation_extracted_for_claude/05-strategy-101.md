# FreqTrade Documentation: Strategy 101

> Source: https://www.freqtrade.io/en/stable/strategy-101/

## Key Concepts

- A strategy is a Python class that defines buy/sell logic using "entry" and "exit" terminology (supporting both long and short positions).
- **Candles** = OHLCV data; **Indicators** = derived technical analysis values; **Signals** = entry/exit conditions.

## Three Essential Methods

1. `populate_indicators(self, dataframe, metadata)` -- Adds technical analysis columns (RSI, EMA, etc.)
2. `populate_entry_trend(self, dataframe, metadata)` -- Sets `enter_long` (or `enter_short`) column to 1
3. `populate_exit_trend(self, dataframe, metadata)` -- Sets `exit_long` (or `exit_short`) column to 1

## Minimal Strategy Example

```python
from freqtrade.strategy import IStrategy
import talib.abstract as ta

class MyStrategy(IStrategy):
    timeframe = '15m'
    stoploss = -0.10
    minimal_roi = {"0": 0.01}

    def populate_indicators(self, dataframe, metadata):
        dataframe['rsi'] = ta.RSI(dataframe, timeperiod=14)
        return dataframe

    def populate_entry_trend(self, dataframe, metadata):
        dataframe.loc[dataframe['rsi'] < 30, 'enter_long'] = 1
        return dataframe

    def populate_exit_trend(self, dataframe, metadata):
        dataframe.loc[dataframe['rsi'] > 70, 'exit_long'] = 1
        return dataframe
```

## Key Configuration Variables

- `timeframe`: Candle interval (e.g., '5m', '15m', '1h')
- `stoploss`: Maximum loss threshold (e.g., -0.10 = -10%)
- `minimal_roi`: Profit-taking thresholds (e.g., `{"0": 0.01}` = exit at 1% any time)
- `max_open_trades`: Maximum concurrent positions

## Testing

- **Backtesting:** Fast historical analysis, but assumes perfect fills and candle-close entries.
- **Dry Run:** Real-time simulation with actual market data, no real money.
- `lookahead-analysis` and `recursive-analysis` commands detect strategy bugs.
- Significant backtesting vs. dry-run differences suggest signal consistency issues.

## Practical Notes

- Signals do not always generate trades due to: insufficient funds, no available slots, existing open position on the pair, colliding simultaneous entry/exit signals, or strategy rejection via callbacks.
- Six bot control mechanisms: FreqUI, Telegram, FTUI, freqtrade-client, REST API, Webhooks.
- "Most public strategies are not good performers" -- always test thoroughly.
- Only invest what you are willing to lose.
