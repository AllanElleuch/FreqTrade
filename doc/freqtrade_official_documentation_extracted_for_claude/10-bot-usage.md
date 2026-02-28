# FreqTrade Documentation: Bot Usage

> Source: https://www.freqtrade.io/en/stable/bot-usage/

## Key Concepts Covered

- Bot startup requirements, CLI commands, configuration file management, strategy deployment, and user data directory structure.

## Important Commands and Flags

```
freqtrade trade [options]
```

| Flag | Purpose |
|------|---------|
| `-c / --config PATH` | Specify config file(s) |
| `-s / --strategy NAME` | Select strategy class name |
| `--strategy-path PATH` | Custom strategy directory |
| `--recursive-strategy-search` | Search strategy directories recursively |
| `--db-url PATH` | Override database location |
| `--dry-run` | Force simulation mode |
| `--dry-run-wallet` | Set starting balance for dry run |
| `--fee FLOAT` | Apply custom transaction fee |
| `-v / --verbose` | Increase logging verbosity |
| `--logfile PATH` | Log to file |
| `--sd-notify` | Enable systemd notification |
| `--freqaimodel NAME` | Specify FreqAI model |

## Configuration Patterns

- **Single config:** `freqtrade trade -c config.json`
- **Multiple configs (override pattern):** `freqtrade trade -c ./config.json -c path/to/secrets/keys.config.json` -- later files override earlier settings, useful for separating credentials from default config.
- **Default behavior:** Loads `config.json` from the current directory if no `-c` flag is provided.

## User Data Directory

```
user_data/
  ├── backtest_results/
  ├── data/
  ├── hyperopts/
  ├── hyperopt_results/
  ├── plot/
  └── strategies/
```

Created with: `freqtrade create-userdir --userdir someDirectory`

## Practical Notes

- System clock must be NTP-synchronized to avoid exchange communication issues.
- Virtual environment must be activated: `source .venv/bin/activate`
- Strategies are placed in `user_data/strategies` and selected by class name, not filename.
- Version control is recommended for strategy tracking.
- Dry-run mode does not require exchange credentials.
