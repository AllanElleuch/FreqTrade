# FreqTrade Documentation: Deprecated Features

> Source: https://www.freqtrade.io/en/stable/deprecated/

## Key Deprecated/Removed Features

| Feature | Deprecated | Removed | Replacement |
|---|---|---|---|
| `--refresh-pairs-cached` CLI option | 2019.7-dev | 2019.9 | `freqtrade download-data` subcommand |
| `--dynamic-whitelist` CLI option | 2018 | 2019.6-dev | Pairlists plugin system |
| `--live` CLI option | -- | 2019.8 | `download-data` subcommand |
| `ticker_interval` config key | 2020.6 | 2022.3 | `timeframe` |
| Singular `pairlist` config section | 2019.11 | 2020.4 | `pairlists` array |
| `bidVolume`/`askVolume` | 2020.4 | 2020.9 | `quoteVolume` |
| Order book exit pricing (`order_book_min`/`order_book_max`) | -- | 2021.7 | Standard exit pricing |
| Separate hyperopt files | 2021.4 | 2021.9 | Parametrized strategies within strategy files |
| `populate_any_indicators()` (FreqAI) | -- | 2023.3 | Separate feature engineering methods |
| Configuration-based protections | -- | 2024.10 | Strategy-code-based protections |
| HDF5 data format | 2024.12 | 2025.1 | Feather format (migrate via `convert-data`) |
| Syslog/journald via `--logfile` | 2025.3 | -- | Configuration-based logging (advanced setup) |
| Edge module | 2023.9 | 2025.6 | Completely removed |
| CatBoost models (FreqAI) | -- | 2025.12 | LightGBM or XGBoost |

## Webhook/Telegram Terminology Changes (V2 to V3)

- `buy_tag` became `enter_tag`
- `webhookbuy`/`webhookentry` became `entry`
- `webhooksell`/`webhookexit` became `exit`
- Same pattern for fill and cancel variants

## Dynamic Funding Rate Changes (2025.12)

Mark and funding fee candles now standardize at 1h intervals. Strategies using `@informative("8h", candle_type="funding_rate")` must switch to 1h timeframes.

### Data Migration Options

```bash
rm user_data/data/<exchange>/futures/*-mark*
rm user_data/data/<exchange>/futures/*-funding_rate*
freqtrade download-data -t 1h --trading-mode futures --candle-types funding_rate mark [...]
```

Exception: Hyperliquid requires no action as it already uses regular 1h candles.
