# FreqTrade Documentation: FAQ

> Source: https://www.freqtrade.io/en/stable/faq/

## Key Concepts Covered

### Supported Markets

FreqTrade supports spot trading and futures trading on select exchanges. Short positions are only available in futures mode (`"trading_mode": "futures"`). Leveraged spot tokens (e.g., BTCUP/USD, ETHBULL/USD) can be used to simulate short positions in spot markets. Options trading is not supported.

### Position Management

FreqTrade is restricted to one position per pair at a time. The `adjust_trade_position()` callback allows modification of an existing position (DCA-style).

### Sandbox/Testing

Dry-run mode is the recommended way to test without real funds. Traditional exchange sandbox environments are unsupported because they have different order books, liquidity, and trading behavior.

### Configuration Reloading

The `/reload_config` command applies config changes without a full bot restart.

### Trade Management

- `/stopentry` prevents new entries while keeping existing positions open.
- `/forceexit all` closes all existing positions.

### Database Management

Delete the default database file (`tradesv3.sqlite` or `tradesv3.dryrun.sqlite`) to reset, or specify a different database with `--db-url sqlite:///mynewdatabase.sqlite`.

### Multiple Bot Instances

Supported via advanced-setup documentation.

## Important Configuration Options and Commands

| Configuration/Command | Purpose |
|---|---|
| `"initial_state": "running"` | Prevents bot from starting in STOPPED mode |
| `source .venv/bin/activate` | Activates virtual environment before starting |
| `--logfile /path/to/log.log` | Captures logs to a file |
| `freqtrade hyperopt --hyperopt-loss SharpeHyperOptLossDaily --strategy SampleStrategy -e 1000` | Runs hyperopt with 1000 epochs |
| `order_types` set to `"limit"` | Required for exchanges that do not support market orders |

## Practical Usage Notes

- If the bot starts but makes no trades after 5 minutes, check logs for buy signals -- patience is needed for favorable market conditions.
- Negative profit on a small number of trades is not statistically significant; thousands of trades are needed.
- "Fillup" log messages about missing candles indicate low-volume pairs or exchange downtime; FreqTrade fills gaps with empty candles.
- Coin dust remaining after trades is due to exchange fee structures; use exchange-native fee tokens (e.g., BNB on Binance) to mitigate.
- Strategy loading errors are commonly caused by mismatched filenames, class names (case-sensitive), incorrect file locations, or missing Docker volume mounts.
- Hyperopt default of 100 epochs is insufficient; 500-1000 per run, totaling 10,000+, is recommended.
- GPU support for hyperopt is not implemented because indicator libraries lack GPU compatibility.
- FreqTrade has no associated cryptocurrency token; any such offerings are scams.
