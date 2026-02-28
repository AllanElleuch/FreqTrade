# FreqTrade Documentation: Strategy Customization

> Source: https://www.freqtrade.io/en/stable/strategy-customization/

## Key Concepts

- Strategies must use **pandas vectorized operations** across entire DataFrames, not loops or scalar comparisons.
- Signals are generated at candle close and are "intentions to enter a trade"; actual trades execute on the next candle's open.
- `INTERFACE_VERSION = 3` is the current default.

## Critical Variables

- `startup_candle_count`: Historical candles needed for indicator warmup (e.g., EMA100 needs ~400 candles)
- `can_short = True`: Enables short trading with `enter_short`/`exit_short` columns
- FreqTrade limits OHLCV data requests to 5 calls per pair to avoid exchange overload

## Correct vs. Incorrect Pattern

```python
# WRONG - scalar comparison
if dataframe['rsi'] > 30:
    dataframe['enter_long'] = 1

# CORRECT - vectorized
dataframe.loc[dataframe['rsi'] > 30, 'enter_long'] = 1
```

## Entry Signal Pattern with Tags

```python
dataframe.loc[
    (condition1) & (condition2) & (dataframe['volume'] > 0),
    ['enter_long', 'enter_tag']] = (1, 'signal_name')
```

Always include `dataframe['volume'] > 0` to avoid trading in illiquid periods.

## Minimal ROI Configuration

```python
minimal_roi = {
    "40": 0.0,    # Break-even after 40 min
    "30": 0.01,   # 1% profit after 30 min
    "20": 0.02,   # 2% after 20 min
    "0": 0.04     # 4% immediately
}
```

Disable ROI with empty dict: `minimal_roi = {}`

## Informative Pairs (accessing other timeframes/pairs)

```python
@informative('1h')
def populate_indicators_1h(self, dataframe, metadata):
    dataframe['rsi'] = ta.RSI(dataframe, timeperiod=14)
    return dataframe
```

Use the `@informative` decorator or `merge_informative_pair()` helper to safely combine different-timeframe data without lookahead bias.

## DataProvider Methods

- `self.dp.current_whitelist()` -- dynamically updated trading pairs
- `self.dp.get_pair_dataframe(pair, timeframe)` -- historical candles
- `self.dp.get_analyzed_dataframe(pair, timeframe)` -- candles with signals
- `self.dp.orderbook(pair, maximum)` -- live order book (not reliable in backtesting)
- `self.dp.ticker(pair)` -- current price
- `self.dp.funding_rate(pair)` -- futures funding rates

## Wallet Access

```python
free_balance = self.wallets.get_free('ETH')
used_balance = self.wallets.get_used('ETH')
```

## Pair Locking

- `self.lock_pair(pair, until=datetime, reason='string')`
- `self.unlock_pair(pair)`
- Only protection-based locks work during backtesting; manual locks do not.

## Lookahead Bias Prevention

- Never use `shift(-1)` (negative shifts)
- Avoid `.iloc[-1]` within populate methods
- Use `.rolling().mean()` instead of `.mean()` on full DataFrame
- Use `.resample('1h', label='right')` instead of plain resample
- Always run `lookahead-analysis` and `recursive-analysis` before deployment

## Practical Notes

- Use class name (not filename) when running: `freqtrade trade --strategy ClassName`
- Custom strategy directory: `--strategy-path user_data/otherPath`
- `freqtrade list-strategies` shows available strategies
- Signal collision: when both entry and exit signals fire simultaneously, entry is ignored
- Trade history via `Trade.get_trades_proxy()` is unavailable during backtesting/hyperopt populate methods
