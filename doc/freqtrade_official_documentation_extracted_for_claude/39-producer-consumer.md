# FreqTrade Documentation: Producer/Consumer Mode

> Source: https://www.freqtrade.io/en/stable/producer-consumer/

## Key Concepts

A distributed architecture where a "producer" instance broadcasts analyzed dataframes via WebSocket to "consumer" instances. Consumers reuse computed indicators without recalculating, reducing computational overhead. Enables running consumers on low-power devices while the producer handles heavy computation.

## Consumer Configuration

```json
{
  "external_message_consumer": {
    "enabled": true,
    "producers": [
      {
        "name": "default",
        "host": "192.168.1.100",
        "port": 8080,
        "ws_token": "secretToken123"
      }
    ]
  }
}
```

## Optional Parameters

- `wait_timeout` (300s) -- reconnection interval
- `ping_timeout` (10s) -- WebSocket ping timeout
- `sleep_time` (10s) -- retry delay
- `remove_entry_exit_signals` (false) -- strip signal columns from producer data
- `initial_candle_limit` (1500) -- historical candle count on initial connection
- `message_size_limit` (8 MB) -- max message size

## Consumer Strategy Pattern

```python
process_only_new_candles = False  # MANDATORY for consumers

def populate_indicators(self, dataframe, metadata):
    pair = metadata['pair']
    producer_df, _ = self.dp.get_producer_df(pair)
    dataframe = merge_informative_pair(dataframe, producer_df,
                                        self.timeframe, self.timeframe,
                                        append_timeframe=False, suffix="default")
    return dataframe
```

**IMPORTANT:** `process_only_new_candles = False` is mandatory on the consumer side.

## Data Access Methods

- `self.dp.get_producer_pairs()` -- list available pairs from the default producer
- `self.dp.get_producer_pairs("producer_name")` -- list available pairs from a specific producer
- `self.dp.get_producer_df(pair)` -- get analyzed dataframe from the default producer
- `self.dp.get_producer_df(pair, timeframe="1h", producer_name="other")` -- get dataframe with specific timeframe from a named producer

## Practical Notes

- Producer requires `api_server` with WebSocket enabled.
- Set `ws_token` to a random secret to prevent unauthorized access.
- Multiple producers are supported; specify producer name in config and API calls.
- Consumer can use producer entry/exit signals directly via suffixed columns (e.g., `enter_long_default`).
