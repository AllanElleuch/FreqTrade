# FreqTrade Project Knowledge Base

## Project Overview
FreqTrade crypto trading bot at `/mnt/nvme/FreqTrade/repo/ft_userdata/`
- Exchange: Bybit futures (isolated margin, USDT)
- Runtime: Docker (`docker compose up -d`)
- Active strategy: LSv3_F (non-FreqAI, Ichimoku+EMA fans)
- Mode: dry_run (paper trading), 2000 USDT wallet

## Sub-Agent Routing Table

| Task | Read First | Memory File |
|------|-----------|-------------|
| Write/modify strategy | 06, 01 | `01-strategy-development.md` |
| Add ML/FreqAI | 06, 02 | `02-freqai-machine-learning.md` |
| Backtest or hyperopt | 06, 03 | `03-backtesting-optimization.md` |
| Config/Docker/infra | 06, 04 | `04-configuration-infrastructure.md` |
| Data/Telegram/API ops | 06, 05 | `05-data-operations.md` |
| Understand repo state | 06 | `06-project-context.md` |

Always read `06-project-context.md` first for repo layout and current state.

## Full Documentation
46 extracted doc files at:
`/mnt/nvme/FreqTrade/repo/doc/freqtrade_official_documentation_extracted_for_claude/`
See `00-index.md` for complete file listing with topics.

## Key File Paths
```
ft_userdata/user_data/
├── config.json                    # Live config (VolumePairList, 30 pairs)
├── config-backtest.json           # Backtest config (StaticPairList)
├── config-backtest-freqai.json    # FreqAI config (BTC/USDT only)
├── strategies/
│   ├── LSv3_F.py                  # ACTIVE: production strategy
│   ├── LSv3.py                    # With FreqAI support
│   ├── LSv1.py                    # Original version
│   ├── SampleStrategy.py          # Default sample
│   ├── FreqaiExampleStrategy.py   # Pure ML example
│   └── FreqaiExampleHybridStrategy.py  # TA+ML hybrid
├── data/                          # OHLCV cache
├── logs/                          # Bot logs
└── models/                        # FreqAI models
```

## Quick Commands (Docker)
```bash
# Start/stop bot
docker compose up -d && docker compose down

# Backtest
docker compose run --rm freqtrade backtesting \
  --config /freqtrade/user_data/config-backtest.json \
  --strategy LSv3_F --timerange 20240101-20240601

# Hyperopt
docker compose run --rm freqtrade hyperopt \
  --config /freqtrade/user_data/config-backtest.json \
  --strategy LSv3_F --hyperopt-loss SharpeHyperOptLossDaily \
  --epochs 500 --spaces buy sell roi stoploss trailing

# Download data
docker compose run --rm freqtrade download-data \
  --config /freqtrade/user_data/config.json \
  --timerange 20240101- --timeframe 5m 15m 1h 4h

# View logs
docker compose logs -f
```

## Strategy Architecture (LSv1 → LSv3 → LSv3_F)
- Ichimoku Cloud (senkou_a/b) as trend filter
- Multi-timeframe EMA fan: 5m to 8h EMAs on Heikinashi candles
- Fan magnitude = EMA_2h / EMA_8h (>1 bullish, <1 bearish)
- Hyperoptable trend levels (1-8), fan gain thresholds, exit indicators
- 15m timeframe, trailing stops, futures long+short

## IStrategy Quick Reference
```python
class MyStrategy(IStrategy):
    INTERFACE_VERSION = 3
    can_short = True
    timeframe = '15m'
    stoploss = -0.10
    minimal_roi = {"60": 0.01, "30": 0.02, "0": 0.04}

    # Required: populate_indicators(), populate_entry_trend(), populate_exit_trend()
    # Callbacks: custom_stoploss(), custom_exit(), leverage(), adjust_trade_position()
    # FreqAI: feature_engineering_expand_all(), set_freqai_targets()
    # Hyperopt: IntParameter, DecimalParameter, CategoricalParameter, BooleanParameter
```

## Security Reminders
- API keys in config.json - never commit to public repos
- Telegram tokens in config files
- Jupyter token hardcoded in docker-compose
- Always use `dry_run: true` when testing
