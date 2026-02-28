# FreqTrade Documentation: Data Download

> Source: https://www.freqtrade.io/en/stable/data-download/

## Key Concepts Covered

- Downloading historical OHLCV and trade data, format conversion, data listing, supported formats, and exchange-specific considerations.

## Primary Commands

| Command | Purpose |
|---------|---------|
| `freqtrade download-data` | Download historical candle/trade data |
| `freqtrade convert-data` | Convert between storage formats |
| `freqtrade convert-trade-data` | Convert tick-level data formats |
| `freqtrade trades-to-ohlcv` | Convert tick data to candle data |
| `freqtrade list-data` | List available downloaded datasets |

## Key `download-data` Flags

| Flag | Purpose |
|------|---------|
| `--exchange NAME` | Target exchange |
| `--pairs PAIR [PAIR ...]` | Specific pairs (supports regex like `.*/USDT`) |
| `--pairs-file FILE` | Load pairs from JSON file |
| `--days INT` | History depth (default 30) |
| `--timeframes 1m 5m 1h ...` | Candle intervals (default 1m 5m) |
| `--timerange 20200101-` | Absolute date range |
| `--dl-trades` | Download tick-level data |
| `--convert` | Auto-convert trades to OHLCV |
| `--trading-mode {spot,futures}` | Market type |
| `--erase` | Remove existing data first |
| `--prepend` | Add data before current range |

## Supported Data Formats

| Format | Size (6 pairs) | Speed | Notes |
|--------|----------------|-------|-------|
| **feather** (default) | 72 MB | 3.5s | Recommended; best balance |
| **json** | 149 MB | 25.6s | Largest, slowest |
| **jsongz** | 39 MB | 27s | Smallest, slow |
| **parquet** | 83 MB | 3.8s | OHLCV only; comparable to feather |

Config persistence:

```json
{
  "dataformat_ohlcv": "feather",
  "dataformat_trades": "feather"
}
```

## Practical Notes

- Downloads are incremental; FreqTrade auto-detects missing timeranges and fills gaps.
- Pairs file format: `["ETH/BTC", "ETH/USDT", "BTC/USDT"]`
- Kraken requires `--dl-trades` as it lacks historic OHLCV data.
- Docker-created `user_data` directories may need ownership fix: `sudo chown -R $UID:$GID user_data`.
- The download command does not account for strategy startup periods; download extra data manually if needed.
- Feather format is the recommended default for its performance-to-size ratio.
