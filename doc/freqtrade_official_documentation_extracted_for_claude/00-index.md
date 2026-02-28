# FreqTrade Documentation Index

> Source: https://www.freqtrade.io/en/stable/
> Extracted: 2026-02-28 for Claude sub-agent consumption

## Overview

FreqTrade is a free and open-source crypto trading bot written in Python. It supports:
- Multiple exchanges (Binance, Bybit, OKX, etc.)
- Spot and futures trading (long and short)
- Backtesting with historical data
- Hyperparameter optimization (Hyperopt via Optuna)
- Machine learning integration (FreqAI)
- Telegram bot notifications
- REST API / FreqUI web interface
- Docker deployment

## Documentation File Index

### Getting Started
| # | File | Topic |
|---|------|-------|
| 01 | `01-docker-quickstart.md` | Docker deployment quickstart |
| 02 | `02-installation.md` | Installation from source |
| 03 | `03-bot-basics.md` | Core concepts, order flow, terminology |
| 04 | `04-configuration.md` | config.json structure, all parameters |

### Strategy Development
| # | File | Topic |
|---|------|-------|
| 05 | `05-strategy-101.md` | IStrategy interface basics |
| 06 | `06-strategy-customization.md` | Advanced indicator/signal patterns |
| 07 | `07-strategy-callbacks.md` | All callback functions |
| 08 | `08-stoploss.md` | Stoploss types and configuration |
| 33 | `33-trade-object.md` | Trade object properties and methods |
| 36 | `36-strategy-advanced.md` | Advanced strategy techniques |
| 42 | `42-strategy-migration.md` | Migrating between strategy versions |

### Plugins & Operations
| # | File | Topic |
|---|------|-------|
| 09 | `09-plugins.md` | Pairlist handlers, protections |
| 10 | `10-bot-usage.md` | CLI commands and bot operation |
| 11 | `11-telegram-usage.md` | Telegram bot setup and commands |
| 12 | `12-freq-ui.md` | Web UI setup and features |
| 13 | `13-rest-api.md` | REST API endpoints |
| 14 | `14-webhook-config.md` | Webhook notifications |
| 15 | `15-data-download.md` | Historical data downloading |

### Backtesting & Optimization
| # | File | Topic |
|---|------|-------|
| 16 | `16-backtesting.md` | Backtesting commands and output |
| 17 | `17-hyperopt.md` | Hyperparameter optimization |
| 31 | `31-advanced-backtesting.md` | Advanced backtesting analysis |
| 34 | `34-lookahead-analysis.md` | Lookahead bias detection |
| 35 | `35-recursive-analysis.md` | Recursive analysis tools |
| 37 | `37-advanced-hyperopt.md` | Custom hyperopt spaces/losses |

### FreqAI (Machine Learning)
| # | File | Topic |
|---|------|-------|
| 18 | `18-freqai-introduction.md` | FreqAI overview and concepts |
| 19 | `19-freqai-configuration.md` | FreqAI config parameters |
| 20 | `20-freqai-parameter-table.md` | Complete parameter reference |
| 21 | `21-freqai-feature-engineering.md` | Feature engineering functions |
| 22 | `22-freqai-running.md` | Running FreqAI models |
| 23 | `23-freqai-reinforcement-learning.md` | RL integration |
| 24 | `24-freqai-developers.md` | FreqAI developer guide |

### Exchange & Infrastructure
| # | File | Topic |
|---|------|-------|
| 25 | `25-leverage.md` | Leverage/futures/short trading |
| 26 | `26-utility-subcommands.md` | CLI utility commands |
| 27 | `27-plotting.md` | Chart plotting tools |
| 28 | `28-exchanges.md` | Exchange-specific notes |
| 29 | `29-data-analysis.md` | Jupyter notebook analysis |
| 30 | `30-strategy-analysis-example.md` | Strategy analysis walkthrough |
| 32 | `32-advanced-setup.md` | Advanced installation/setup |

### Advanced & Reference
| # | File | Topic |
|---|------|-------|
| 38 | `38-advanced-orderflow.md` | Orderflow analysis |
| 39 | `39-producer-consumer.md` | Multi-bot producer/consumer |
| 40 | `40-sql-cheatsheet.md` | Database SQL queries |
| 41 | `41-faq.md` | Frequently asked questions |
| 43 | `43-updating.md` | Updating FreqTrade |
| 44 | `44-deprecated.md` | Deprecated features |
| 45 | `45-developer-guide.md` | Contributing to FreqTrade |

## Quick Reference

### Essential CLI Commands
```bash
# Live trading
freqtrade trade --config config.json --strategy MyStrategy

# Backtesting
freqtrade backtesting --config config.json --strategy MyStrategy --timerange 20240101-20240601

# Hyperopt
freqtrade hyperopt --config config.json --strategy MyStrategy --hyperopt-loss SharpeHyperOptLossDaily --epochs 500

# Data download
freqtrade download-data --config config.json --timerange 20240101-20240601 --timeframe 5m 15m 1h

# List strategies
freqtrade list-strategies --strategy-path user_data/strategies
```

### Docker Commands
```bash
# Start bot
docker compose up -d

# View logs
docker compose logs -f

# Run backtest in container
docker compose run --rm freqtrade backtesting --config /freqtrade/user_data/config.json --strategy MyStrategy
```
