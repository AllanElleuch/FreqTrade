# FreqTrade Documentation: Configuration

> Source: https://www.freqtrade.io/en/stable/configuration/

## Key Concepts

- Configuration is JSON-based (`config.json`), supporting single-line (`//`) and multi-line (`/* */`) comments plus trailing commas.
- Environment variables override config files using `FREQTRADE__` prefix with `__` as level separators (e.g., `FREQTRADE__EXCHANGE__KEY`).
- Multiple config files can be merged via `add_config_files` to separate public settings from private credentials.

## Essential Configuration Parameters

| Parameter | Description |
|---|---|
| `max_open_trades` | Simultaneous positions (-1 = unlimited) |
| `stake_currency` | Trading currency (USDT, BTC, etc.) |
| `stake_amount` | Per-trade amount, or `"unlimited"` for dynamic sizing |
| `dry_run` | Boolean: simulation vs. live |
| `dry_run_wallet` | Starting balance for simulation (default: 1000) |
| `tradable_balance_ratio` | Fraction of balance available (default: 0.99) |
| `available_capital` | Limits capital when running multiple bots |
| `stoploss` | Maximum loss ratio per trade |
| `minimal_roi` | Profit targets with time-based exits |
| `trailing_stop` | Dynamic stoploss adjustment |

## Pricing Configuration

- `entry_pricing.price_side`: "ask", "bid", "same", or "other"
- `entry_pricing.use_order_book`: Use orderbook vs. ticker data
- `entry_pricing.order_book_top`: Orderbook depth level (1 = best)
- `entry_pricing.check_depth_of_market.enabled`: Filter on orderbook depth
- Same structure exists for `exit_pricing`
- Market orders should use "other" price side for realistic execution pricing

## Exchange Setup

- `exchange.name`, `exchange.key`, `exchange.secret`, `exchange.password`
- `exchange.pair_whitelist` / `exchange.pair_blacklist`
- `exchange.ccxt_config` for additional CCXT library parameters
- `exchange.enable_ws` for websocket toggle (default: true)
- `trading_mode`: "spot", leverage, or contract trading
- `margin_mode`: "cross" or "isolated"

## Integrations

- **Telegram:** `telegram.enabled`, `telegram.token`, `telegram.chat_id`, `telegram.notification_settings`
- **Webhooks:** HTTP callbacks for trade events (entry, exit, fills, cancellations)
- **REST API / FreqUI:** `api_server.listen_ip_address`, `api_server.listen_port`, `api_server.username`, `api_server.password`

## Configuration Precedence (highest to lowest)

CLI arguments > Environment variables > Configuration files > Strategy defaults

## Practical Notes

- Use `freqtrade show-config` to display the final merged configuration for debugging.
- JSON schema validation available via `https://schema.freqtrade.io/schema.json`.
- Never hardcode API keys in primary config; use environment variables or separate private config files.
- `stake_amount: "unlimited"` divides available balance equally across allowed trades, enabling compounding.
- Transition to production: set `dry_run: false`, use fresh database, start with small position sizes.
