# FreqTrade Documentation: Utility Sub-commands

> Source: https://www.freqtrade.io/en/stable/utils/

## Overview

FreqTrade provides a broad set of CLI utilities beyond core trading and backtesting: directory setup, config creation, strategy management, exchange/market listing, data conversion, and results analysis.

## Command Reference

| Command | Purpose |
|---------|---------|
| `freqtrade create-userdir` | Create directory structure with sample files. `--reset` restores defaults (risks data loss). |
| `freqtrade new-config` | Interactive wizard for config generation (dry-run, stake currency, exchange, timeframe). |
| `freqtrade show-config` | Displays merged config with secrets redacted; use `--show-sensitive` to reveal. |
| `freqtrade new-strategy` | Generate strategy templates: `"minimal"`, `"full"` (default), or `"advanced"` (all optional functions). |
| `freqtrade list-strategies` | List available strategies; red = load failure, yellow = duplicate names. |
| `freqtrade strategy-updater` | Convert strategies to v3; originals saved to `strategies_orig_updater/`. |
| `freqtrade list-exchanges` | Show exchanges with support levels and trading modes. `-a` for all CCXT exchanges. |
| `freqtrade list-timeframes` | Show supported timeframes for a given exchange. |
| `freqtrade list-pairs` | Filter pairs by base/quote with table, JSON, CSV, or list output. |
| `freqtrade test-pairlist` | Validate dynamic pairlist configs; useful for generating static pairlists for backtesting. |
| `freqtrade convert-db` | Migrate trade data between database systems (target must be empty). |
| `freqtrade show-trades` | Retrieve trades from DB with JSON output option. |
| `freqtrade backtesting-show` | Display previous backtest results with pair list generation. |
| `freqtrade backtesting-analysis` | Advanced analysis with grouping (0-5), signal filtering, indicator analysis, CSV export. |
| `freqtrade hyperopt-list` | List hyperopt epochs with filtering (best, profitable, trade counts, profit ranges). |
| `freqtrade hyperopt-show` | Detailed info for specific epochs with JSON/parameter export. |

## Common CLI Arguments

| Argument | Description |
|----------|-------------|
| `-c/--config` | Specify configuration file |
| `-d/--datadir` | Specify data directory |
| `--userdir` | Specify user data directory |
| `-v/--verbose` | Enable verbose output |
| `--logfile` | Specify log file path |
| `--no-color` | Disable colored output |
| `-V/--version` | Show version information |

## Practical Usage Notes

- The `--reset` flag overwrites sample files **without confirmation** -- use with caution.
- Commands loading Python files from directories carry **security risks** with untrusted files.
- Selecting only winning pairs for backtesting leads to **overfitting**; always do extensive dry-run testing.
- The webserver mode (experimental) allows FreqUI to control backtesting without reloading data.

## Key Command Examples

### Create user directory
```bash
freqtrade create-userdir --userdir user_data
```

### Generate a new config interactively
```bash
freqtrade new-config -c config.json
```

### Create a new strategy
```bash
freqtrade new-strategy --strategy MyStrategy --template advanced
```

### List available exchanges
```bash
freqtrade list-exchanges
freqtrade list-exchanges -a  # all CCXT exchanges
```

### List pairs with filtering
```bash
freqtrade list-pairs --exchange binance --quote USDT
```

### Test pairlist configuration
```bash
freqtrade test-pairlist -c config.json
```

### Backtesting analysis with grouping
```bash
freqtrade backtesting-analysis -c config.json --analysis-groups 0 1 2 3 4 5
```

### Hyperopt results
```bash
freqtrade hyperopt-list -c config.json --profitable
freqtrade hyperopt-show -c config.json --best -n 5
```
