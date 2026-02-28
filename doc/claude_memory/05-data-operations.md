# FreqTrade Data & Operations Reference

## Data Download

### Download Historical Data
```bash
freqtrade download-data \
  --config user_data/config.json \
  --timerange 20230101-20240601 \
  --timeframe 5m 15m 1h 4h \
  --exchange bybit \
  --trading-mode futures
```

### Key Flags
- `--pairs BTC/USDT:USDT ETH/USDT:USDT` - Specific pairs
- `--pairs-file user_data/pairs.json` - Pairs from file
- `--timeframe 5m 15m 1h 4h 1d` - Multiple timeframes
- `--timerange YYYYMMDD-YYYYMMDD` - Date range
- `--exchange bybit` - Exchange name
- `--trading-mode futures` - futures or spot
- `--erase` - Delete existing data first
- `--data-format-ohlcv json|jsongz|hdf5|feather` - Storage format
- `--datadir user_data/data` - Data directory

### Data Storage
Data stored in: `user_data/data/<exchange>/`
Format: `<pair>-<timeframe>.json` (or .json.gz, .feather, .h5)

### List Downloaded Data
```bash
freqtrade list-data --config user_data/config.json --data-format-ohlcv json
```

### Convert Data Format
```bash
freqtrade convert-data --format-from json --format-to feather --datadir user_data/data
```

## Bot Operations

### CLI Commands
```bash
# Start trading
freqtrade trade --config config.json --strategy MyStrategy

# Show trades
freqtrade show-trades --db-url sqlite:///tradesv3.sqlite

# List strategies
freqtrade list-strategies --strategy-path user_data/strategies

# List exchanges
freqtrade list-exchanges

# List pairs for exchange
freqtrade list-pairs --exchange bybit --trading-mode futures

# Test pairlist
freqtrade test-pairlist --config config.json

# Create new strategy
freqtrade new-strategy --strategy MyNewStrategy --strategy-path user_data/strategies
```

## Telegram Bot Commands
```
/start           - Start the bot
/stop            - Stop the bot
/status          - Show open trades
/status table    - Show open trades as table
/profit          - Show profit summary
/profit [N]      - Show profit for last N days
/forceexit <id>  - Force exit a trade
/forceexit all   - Force exit all trades
/forceenter <pair> <side> - Force enter a trade
/balance         - Show balance
/daily [N]       - Show daily profit for N days
/weekly [N]      - Show weekly profit
/monthly [N]     - Show monthly profit
/performance     - Show pair performance
/trades [N]      - Show recent trades
/delete <id>     - Delete a trade
/count           - Show open trade count
/locks           - Show locked pairs
/unlock <id>     - Unlock a pair
/reload_config   - Reload configuration
/show_config     - Show running config
/logs [N]        - Show last N log entries
/edge            - Show Edge results
/health          - Show bot health
/version         - Show version
/help            - Show help
```

## REST API
Base URL: `http://localhost:8080/api/v1/`

### Key Endpoints
```
GET  /ping              - Health check
GET  /version           - Version info
GET  /balance           - Account balance
GET  /count             - Open trade count
GET  /profit            - Profit summary
GET  /trades            - List trades
GET  /status            - Open trade status
GET  /show_config       - Current config
GET  /performance       - Pair performance
GET  /daily             - Daily profit
POST /forcebuy          - Force enter trade
POST /forcesell         - Force exit trade
POST /start             - Start trading
POST /stop              - Stop trading
POST /reload_config     - Reload config
GET  /pair_candles      - OHLCV data
GET  /pair_history      - Historical data
GET  /strategies        - List strategies
GET  /strategy/<name>   - Strategy details
```

### Authentication
JWT-based. Get token: `POST /api/v1/token/login` with username/password.

## Plugins

### Pairlist Handlers (chained)
- `StaticPairList` - Fixed pair list
- `VolumePairList` - Top N by volume
- `ProducerPairList` - From external bot
- `PerformanceFilter` - Filter by past performance
- `PriceFilter` - Min/max price
- `SpreadFilter` - Max spread
- `AgeFilter` - Min pair listing age
- `ShuffleFilter` - Randomize order
- `OffsetFilter` - Skip first N pairs

### Protection Plugins
```json
{
  "protections": [
    {"method": "CooldownPeriod", "stop_duration": 5},
    {"method": "StoplossGuard", "trade_limit": 4, "stop_duration": 60, "only_per_pair": false},
    {"method": "MaxDrawdown", "trade_limit": 20, "stop_duration": 120, "max_allowed_drawdown": 0.2},
    {"method": "LowProfitPairs", "trade_limit": 5, "stop_duration": 60, "required_profit": 0.02}
  ]
}
```

## Webhook Configuration
```json
{
  "webhook": {
    "enabled": true,
    "url": "https://your-webhook-url.com",
    "webhookentry": {"value1": "Entering {pair}"},
    "webhookexit": {"value1": "Exiting {pair} at {profit_ratio:.2%}"},
    "webhookentrycancel": {"value1": "Entry cancelled for {pair}"},
    "webhookexitcancel": {"value1": "Exit cancelled for {pair}"},
    "webhookstatus": {"value1": "Status: {status}"}
  }
}
```

## Producer/Consumer Mode
Run multiple bots where a "producer" generates signals and "consumers" execute trades.
```json
{
  "external_message_consumer": {
    "enabled": true,
    "producers": [
      {
        "name": "producer1",
        "host": "192.168.1.100",
        "port": 8080,
        "ws_token": "token123"
      }
    ]
  }
}
```

## SQL Cheatsheet
```sql
-- Open trades
SELECT * FROM trades WHERE is_open = 1;

-- Profitable trades
SELECT * FROM trades WHERE close_profit > 0;

-- Profit by pair
SELECT pair, SUM(close_profit_abs) as profit FROM trades GROUP BY pair ORDER BY profit DESC;

-- Delete specific trade
DELETE FROM trades WHERE id = 123;

-- Reset all trades (DANGEROUS)
DELETE FROM trades;
```

## Detailed Documentation
See files in `/mnt/nvme/FreqTrade/repo/doc/freqtrade_official_documentation_extracted_for_claude/`:
- `09-plugins.md` - Plugins reference
- `10-bot-usage.md` - CLI usage
- `11-telegram-usage.md` - Telegram bot
- `13-rest-api.md` - REST API
- `14-webhook-config.md` - Webhooks
- `15-data-download.md` - Data download
- `39-producer-consumer.md` - Multi-bot setup
- `40-sql-cheatsheet.md` - SQL queries
