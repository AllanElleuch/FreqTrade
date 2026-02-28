# FreqTrade Configuration & Infrastructure Reference

## Docker Setup

### Directory Structure
```
ft_userdata/
├── docker-compose.yml          # Main trading bot
├── docker-compose-jupyter.yml  # Jupyter Lab for analysis
├── start-freqtrade.sh         # Quick start script
├── start-jupyter.sh           # Quick Jupyter start
└── user_data/
    ├── config.json            # Main configuration
    ├── strategies/            # Strategy files
    ├── data/                  # OHLCV data cache
    ├── logs/                  # Bot logs
    ├── models/                # FreqAI models
    └── docker/                # Custom Dockerfiles
```

### Docker Commands
```bash
# Start bot
docker compose up -d

# Stop bot
docker compose down

# View logs
docker compose logs -f
docker compose logs --tail 100

# Run one-off command
docker compose run --rm freqtrade <command>

# Rebuild after Dockerfile changes
docker compose build --no-cache

# Start Jupyter
docker compose -f docker-compose-jupyter.yml up -d
```

### docker-compose.yml Key Settings
```yaml
services:
  freqtrade:
    image: freqtradeorg/freqtrade:stable
    restart: unless-stopped
    container_name: freqtrade
    volumes:
      - ./user_data:/freqtrade/user_data
    ports:
      - "8082:8080"
    command: >
      trade
      --logfile /freqtrade/user_data/logs/freqtrade.log
      --db-url sqlite:////freqtrade/user_data/tradesv3.sqlite
      --config /freqtrade/user_data/config.json
      --strategy LSv3_F
```

## Configuration (config.json)

### Essential Parameters
```json
{
    "max_open_trades": 10,
    "stake_currency": "USDT",
    "stake_amount": "unlimited",
    "tradable_balance_ratio": 0.99,
    "dry_run": true,
    "dry_run_wallet": 2000,
    "trading_mode": "futures",
    "margin_mode": "isolated",
    "cancel_open_orders_on_exit": false
}
```

### Exchange Configuration
```json
{
    "exchange": {
        "name": "bybit",
        "key": "YOUR_API_KEY",
        "secret": "YOUR_API_SECRET",
        "ccxt_config": {},
        "ccxt_sync_config": {},
        "pair_whitelist": ["BTC/USDT:USDT", "ETH/USDT:USDT"],
        "pair_blacklist": ["BNB/.*"]
    }
}
```

### Pairlist Configuration
```json
{
    "pairlists": [
        {
            "method": "VolumePairList",
            "number_assets": 30,
            "sort_key": "quoteVolume",
            "refresh_period": 1800
        }
    ]
}
```
Other methods: `StaticPairList`, `PerformanceFilter`, `PriceFilter`, `SpreadFilter`, `AgeFilter`, `ShuffleFilter`

### Order Types
```json
{
    "order_types": {
        "entry": "market",
        "exit": "market",
        "stoploss": "market",
        "stoploss_on_exchange": false
    },
    "unfilledtimeout": {
        "entry": 10,
        "exit": 10,
        "unit": "minutes"
    }
}
```

### API Server
```json
{
    "api_server": {
        "enabled": true,
        "listen_ip_address": "0.0.0.0",
        "listen_port": 8080,
        "verbosity": "error",
        "enable_openapi": false,
        "jwt_secret_key": "your_secret",
        "ws_token": "your_ws_token",
        "CORS_origins": [],
        "username": "freqtrader",
        "password": "your_password"
    }
}
```

### Telegram
```json
{
    "telegram": {
        "enabled": true,
        "token": "YOUR_BOT_TOKEN",
        "chat_id": "YOUR_CHAT_ID",
        "notification_settings": {
            "status": "silent",
            "warning": "on",
            "startup": "on",
            "entry": "on",
            "exit": "on"
        }
    }
}
```

## Database
- Default: SQLite (`sqlite:///tradesv3.sqlite`)
- PostgreSQL: `postgresql+psycopg2://user:pass@host:5432/freqtrade`
- MySQL: `mysql+pymysql://user:pass@host:3306/freqtrade`
- Specified via `--db-url` flag or config

## Exchange Notes (Bybit)
- Bybit futures uses USDT-settled perpetual contracts
- Pair format: `BTC/USDT:USDT` (base/quote:settle)
- `trading_mode: "futures"` + `margin_mode: "isolated"`
- Supports leverage (configurable per-trade via `leverage()` callback)
- Stoploss on exchange: supported but disabled by default
- Rate limits: be careful with many pairs + short timeframes

## Environment Variables
Config values can use env vars: `"key": "${EXCHANGE_KEY}"`

## Multiple Configs
```bash
freqtrade trade --config config1.json --config config2.json
```
Later configs override earlier ones. Useful for separating secrets from strategy config.

## This Repo's Config Files
| File | Purpose |
|------|---------|
| `config.json` | Live/dry-run with VolumePairList (30 pairs) |
| `config-backtest.json` | Backtesting with StaticPairList |
| `config-backtest-freqai.json` | FreqAI backtesting (BTC/USDT only) |

## Detailed Documentation
See files in `/mnt/nvme/FreqTrade/repo/doc/freqtrade_official_documentation_extracted_for_claude/`:
- `01-docker-quickstart.md` - Docker setup
- `04-configuration.md` - Full config reference
- `25-leverage.md` - Futures/leverage setup
- `28-exchanges.md` - Exchange-specific notes
- `32-advanced-setup.md` - Advanced installation
