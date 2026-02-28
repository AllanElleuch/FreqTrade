# FreqTrade Documentation: Advanced Setup

> Source: https://www.freqtrade.io/en/stable/advanced-setup/

## Overview

This document covers running multiple bot instances, database configuration, systemd service setup, and advanced logging options for FreqTrade post-installation.

## Running Multiple Bots

Each bot instance needs:

- **Distinct database files** (to avoid data corruption)
- **Separate Telegram bots** (different bot tokens)
- **Different REST API ports** (to avoid port conflicts)

### Per-Instance Database Specification

```bash
freqtrade trade -c MyConfigBTC.json -s MyStrategy \
  --db-url sqlite:///user_data/tradesBTC.dryrun.sqlite
```

## Database Options

### SQLite (Default)

- Auto-creates `tradesv3.sqlite` (live) and `tradesv3.dryrun.sqlite` (dry-run).
- No additional setup required.
- Database tables are auto-created on startup.

### PostgreSQL

```bash
pip install "psycopg[binary]"
```

Connection URL format:
```
postgresql+psycopg://<user>:<pass>@localhost:5432/<db>
```

### MariaDB/MySQL

```bash
pip install pymysql
```

Connection URL format:
```
mysql+pymysql://<user>:<pass>@localhost:3306/<db>
```

**Warning:** The FreqTrade team provides **no support** for PostgreSQL/MariaDB setup or maintenance. Use at your own risk.

## Docker Multiple Instances

Docker-compose template supports running multiple services (e.g., `freqtrade1`, `freqtrade2`) with:

- Distinct configuration files
- Separate databases
- Different port mappings
- Individual log files

## Systemd Service Setup

### Installation

```bash
# Copy service file to user systemd directory
cp freqtrade.service ~/.config/systemd/user/

# Start the service
systemctl --user start freqtrade

# Enable the service to start on boot
systemctl --user enable freqtrade

# Enable persistence when logged out (important!)
sudo loginctl enable-linger "$USER"
```

### Key systemd Notes

- The `sd_notify` protocol enables systemd watchdog functionality.
- The service file should be customized to point to your FreqTrade installation and config files.
- Use `systemctl --user status freqtrade` to check service status.

## Advanced Logging Options

### Default Logging

- Colored terminal output.
- Adding `--logfile` enables `RotatingFileHandler` for automatic log rotation.

### Syslog Logging

Via `log_config` option with handler class `logging.handlers.SysLogHandler`.

Addresses by platform:
| Platform | Address |
|----------|---------|
| Linux | `/dev/log` |
| macOS | `/var/run/syslog` |
| UDP (cross-platform) | `localhost:514` |

### Journald Logging

```bash
pip install cysystemd
```

- Handler class: `cysystemd.journal.JournaldLogHandler`
- **Not available on Windows.**

### JSON Logging

- Custom formatter: `freqtrade.loggers.json_formatter.JsonFormatter`
- Useful for log aggregation systems (e.g., ELK stack, Datadog).

## Practical Usage Notes

- Logging configs should be **extracted to separate config files** to avoid duplication across instances.
- Database tables are **auto-created on startup** -- no manual schema setup needed.
- **Always keep separate databases per bot instance** to avoid data corruption.
- When running multiple bots, consider using a naming convention for databases that includes the strategy or pair information.
- The `enable-linger` command is critical for systemd services -- without it, services stop when the user logs out.
