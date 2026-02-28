# FreqTrade Documentation: Advanced Orderflow

> Source: https://www.freqtrade.io/en/stable/advanced-orderflow/

## Key Concepts

Orderflow analysis uses public trade data to provide detailed breakdowns of buy/sell orders at different price levels. Currently in **beta** with known performance considerations.

## Important Limitations

- Experimental, subject to breaking changes
- Not tested with FreqAI
- Requires substantial resources (large raw trades data, increased memory, slower startup)
- Not all exchanges provide public trade data

## Configuration

Add the following to your `config.json`:

```json
{
  "exchange": {
    "use_public_trades": true
  },
  "orderflow": {
    "cache_size": 1000,
    "max_candles": 500,
    "scale": 0.5,
    "stacked_imbalance_range": 3,
    "imbalance_volume": 0,
    "imbalance_ratio": 3
  }
}
```

### Configuration Parameters

- `cache_size` -- number of cached entries (default 1000)
- `max_candles` -- maximum candles to process (default 500)
- `scale` -- price level granularity (default 0.5)
- `stacked_imbalance_range` -- range for stacked imbalance detection (default 3)
- `imbalance_volume` -- minimum volume for imbalance (default 0)
- `imbalance_ratio` -- ratio threshold for imbalance detection (default 3)

## Download Command

```
freqtrade download-data -p BTC/USDT:USDT --timerange 20230101- \
  --trading-mode futures --timeframes 5m --dl-trades
```

## Available Dataframe Columns

When orderflow is enabled, the following columns are added to the dataframe:

- `trades` -- raw trade data
- `orderflow` -- orderflow data structure
- `imbalances` -- detected imbalances
- `bid` -- total bid volume
- `ask` -- total ask volume
- `delta` -- net order flow (ask - bid)
- `min_delta` -- minimum delta in the candle
- `max_delta` -- maximum delta in the candle
- `total_trades` -- total number of trades
- `stacked_imbalances_bid` -- stacked bid imbalances
- `stacked_imbalances_ask` -- stacked ask imbalances

## Code Pattern

```python
def populate_indicators(self, dataframe: DataFrame, metadata: dict):
    dataframe["cum_delta"] = dataframe["delta"].cumsum()
    # Access orderflow dict per candle for bid_amount, ask_amount, delta, etc.
```
