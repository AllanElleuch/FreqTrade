# Project Context: FreqTrade Trading Bot

## Repository Location
`/mnt/nvme/FreqTrade/repo/ft_userdata/`

## Deployment
- **Exchange:** Bybit (futures, isolated margin)
- **Runtime:** Docker (`docker compose up -d`)
- **Active Strategy:** LSv3_F (non-FreqAI version, in docker-compose.yml)
- **Mode:** Currently dry_run: true (paper trading)
- **Stake:** USDT, unlimited stake_amount, 99% tradable_balance_ratio
- **Max Open Trades:** 10
- **Dry Run Wallet:** 2000 USDT

## File Structure
```
ft_userdata/
├── docker-compose.yml          # Main bot (port 8082→8080)
├── docker-compose-jupyter.yml  # Jupyter Lab (port 8888)
├── start-freqtrade.sh          # docker compose up -d
├── start-jupyter.sh            # Jupyter launcher
└── user_data/
    ├── config.json                 # Live/dry-run config (VolumePairList, 30 pairs)
    ├── config-backtest.json        # Backtest config (StaticPairList)
    ├── config-backtest-freqai.json # FreqAI backtest (BTC/USDT only)
    ├── strategies/
    │   ├── SampleStrategy.py       # Default sample (5m, long-only, RSI/TEMA/BB)
    │   ├── LSv1.py                 # V1: Ichimoku+EMA fans (15m, hardcoded params)
    │   ├── LSv3.py                 # V3: Ichimoku+EMA+FreqAI (15m, hyperoptable)
    │   ├── LSv3_F.py               # V3 Full: Same as LSv3 but FreqAI disabled
    │   ├── LSv3.json               # Hyperopt results for LSv3
    │   ├── LSv3_F.json             # Hyperopt results for LSv3_F
    │   ├── FreqaiExampleStrategy.py      # Pure ML strategy
    │   └── FreqaiExampleHybridStrategy.py # TA + ML hybrid
    ├── docker/
    │   └── Dockerfile.jupyter      # Custom image with jupyterlab
    ├── logs/
    ├── data/                       # Downloaded OHLCV data
    └── models/                     # FreqAI trained models
```

## Strategy Evolution: LSv1 → LSv3 → LSv3_F
- **Core concept:** Ichimoku Cloud trend filter + multi-timeframe EMA "fan magnitude"
- **Fan magnitude:** ratio of 2h EMA to 8h EMA (>1 = bullish, <1 = bearish)
- **LSv1:** Hardcoded params, no trailing stop, 27.5% stoploss
- **LSv3:** Hyperoptable params (IntParameter/DecimalParameter), trailing stop, FreqAI optional, 12.5% stoploss
- **LSv3_F:** Production version without FreqAI, same logic as LSv3
- **Key difference:** LSv3 uses `fan_magnitude_gain >=` but LSv3_F uses `fan_magnitude >=` (potential bug or intentional)

## Strategy Summary Table
| Strategy | Short | FreqAI | TF | Stoploss | Trailing |
|----------|-------|--------|-----|----------|----------|
| SampleStrategy | No | No | 5m | -10% | No |
| LSv1 | Yes | No | 15m | -27.5% | No |
| LSv3 | Yes | Yes | 15m | -12.5% | Yes |
| LSv3_F | Yes | No | 15m | -12.5% | Yes |
| FreqaiExample | Yes | Yes | any | -5% | No |
| FreqaiHybrid | Yes | Yes | any | -5% | No |

## FreqAI Configuration (from config-backtest-freqai.json)
- Training: 30 days, backtest validation: 7 days
- Timeframes: 5m, 15m, 4h
- Correlated pair: BTC/USDT
- Label period: 24 candles
- Indicator periods: [10, 20]
- Data split: 75/25 train/test

## Docker Details
- Main image: `freqtradeorg/freqtrade:stable`
- Jupyter image: built from `freqtradeorg/freqtrade:develop_plot` + jupyterlab
- API: port 8082 (mapped to container 8080)
- Jupyter: port 8888

## Security Notes
- API keys are stored in config.json (review before sharing)
- Telegram bot tokens in config files
- Jupyter token: hardcoded in docker-compose
- Root password set in Dockerfile.jupyter

## Documentation Reference
Full extracted documentation: `/mnt/nvme/FreqTrade/repo/doc/freqtrade_official_documentation_extracted_for_claude/`
See `00-index.md` for complete file listing and quick reference commands.
