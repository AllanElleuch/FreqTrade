# FreqTrade Documentation: Plugins

> Source: https://www.freqtrade.io/en/stable/plugins/

## Key Concepts Covered

- **Pairlist Handlers** -- Define which trading pairs the bot trades. They are configured in the `pairlists` section and can be chained sequentially, where each handler filters or modifies the list passed to it.
- **Pair Blacklist** -- Pairs excluded via `exchange.pair_blacklist`, supporting regex patterns (e.g., `BNB/.*`).
- **Protections** -- Mechanisms that temporarily halt trading (per-pair or globally) based on conditions like drawdown, stoploss triggers, or low profit.

## Available Pairlist Handlers

| Handler | Purpose | Key Parameters |
|---------|---------|----------------|
| **StaticPairList** | Uses statically defined pairs from `exchange.pair_whitelist` | `allow_inactive` |
| **VolumePairList** | Selects top pairs by trading volume | `number_assets`, `sort_key`, `min_value`, `max_value`, `refresh_period`, `lookback_days` |
| **PercentChangePairList** | Filters by 24h price change percentage | `number_assets`, `min_value`, `max_value`, `sort_direction` |
| **ProducerPairList** | Reuses pairlists from Producer instances (consumer mode) | `number_assets`, producer name |
| **RemotePairList** | Fetches from remote URL or local JSON file | `pairlist_url`, `mode` (whitelist/blacklist), `processing_mode` (filter/append), `bearer_token`, `read_timeout` |
| **MarketCapPairList** | Filters by CoinGecko market cap rank | `max_rank`, `categories`, `mode` |

## Available Filters

| Filter | Purpose |
|--------|---------|
| **AgeFilter** | Removes pairs listed fewer than `min_days_listed` days |
| **DelistFilter** | Removes pairs delisting within `max_days_from_now` days |
| **FullTradesFilter** | Shrinks whitelist to in-trade pairs when slots are full |
| **OffsetFilter** | Offsets pairlist by specified amount |
| **PerformanceFilter** | Sorts by historical trade performance |
| **PrecisionFilter** | Removes low-value coins with rounding issues |
| **PriceFilter** | Filters by `min_price`, `max_price`, `max_value`, `low_price_ratio` |
| **ShuffleFilter** | Randomizes order; supports `seed` for reproducibility in backtesting |
| **SpreadFilter** | Removes pairs where bid-ask spread exceeds `max_spread_ratio` |
| **RangeStabilityFilter** | Removes pairs where high-low range is outside min/max rate of change |
| **VolatilityFilter** | Filters by standard deviation of log daily returns |

## Protections

| Protection | Behavior |
|------------|----------|
| **StoplossGuard** | Locks trading when too many stoplosses occur within lookback period |
| **MaxDrawdown** | Stops trading when drawdown exceeds `max_allowed_drawdown` (supports `equity` or `ratios` calculation) |
| **LowProfitPairs** | Locks individual pairs when profit falls below `required_profit` |
| **CooldownPeriod** | Prevents re-entry for `stop_duration` after a trade exit |

## Practical Notes

- Use `freqtrade test-pairlist` or FreqUI in webserver mode to validate pairlist configurations.
- Protections support backtesting with the `--enable-protections` flag.
- All lock times round up to the next candle boundary.
- Handlers can be chained for sophisticated filtering pipelines (e.g., VolumePairList -> DelistFilter -> AgeFilter -> PrecisionFilter -> PriceFilter -> SpreadFilter -> RangeStabilityFilter).
