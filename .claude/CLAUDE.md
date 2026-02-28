# FreqTrade Trading Bot — Project Instructions

FreqTrade crypto trading bot on Bybit futures (isolated margin, USDT).
Deployed via Docker at `/mnt/nvme/FreqTrade/repo/ft_userdata/`.
Working directory: `/mnt/nvme`. Mode: `dry_run: true` (paper trading).

## Safety Rules

- **NEVER** set `dry_run: false` without explicit user confirmation
- **NEVER** modify API keys, secrets, or Telegram tokens
- **NEVER** restart the live/dry-run bot without user confirmation
- **NEVER** delete trade history, databases, or trained models without confirmation
- **ALWAYS** backup hyperopt result JSONs before overwriting them
- **ALWAYS** add a `volume > 0` guard in new strategy entry conditions
- **ALWAYS** use `docker compose run --rm freqtrade` for one-off commands (backtest, hyperopt, download-data)
- **ALWAYS** use `docker compose up -d` / `docker compose down` for the persistent bot

## Sub-Agent System

Before any task, read memory file `06-project-context.md` for current repo state.
Memory files are at: `/home/poop/.claude/projects/-mnt-nvme/memory/`
Documentation is at: `/mnt/nvme/FreqTrade/repo/doc/freqtrade_official_documentation_extracted_for_claude/`

**How to spawn a sub-agent:** Use the Agent tool with `subagent_type: "general-purpose"`. Read the agent's prompt file from `/mnt/nvme/FreqTrade/repo/.claude/agents/<name>.md` and include its contents as the prompt, appended with the specific user task. The agent file contains all context-loading instructions, rules, and output format.

### STRATEGY Agent
**Triggers:** write strategy, modify strategy, debug strategy, add indicators, entry/exit logic, signals
**Pre-read:** `06-project-context.md`, `01-strategy-development.md`
**Key docs:** `05-strategy-101.md`, `06-strategy-customization.md`, `07-strategy-callbacks.md`, `08-stoploss.md`, `33-trade-object.md`, `36-strategy-advanced.md`
**Rules:**
- Study existing strategies (LSv3_F.py is production) before writing new ones
- Use IStrategy INTERFACE_VERSION = 3, set `can_short = True` for futures
- Make parameters hyperoptable with IntParameter/DecimalParameter
- Include volume > 0 guard in all entry conditions
- Use informative_pairs() for multi-timeframe data

### FREQAI Agent
**Triggers:** add ML, train model, feature engineering, reinforcement learning, FreqAI
**Pre-read:** `06-project-context.md`, `02-freqai-machine-learning.md`
**Key docs:** `18-freqai-introduction.md`, `19-freqai-configuration.md`, `20-freqai-parameter-table.md`, `21-freqai-feature-engineering.md`, `22-freqai-running.md`, `23-freqai-reinforcement-learning.md`
**Rules:**
- Use `config-backtest-freqai.json` for FreqAI backtests
- Feature engineering goes in `feature_engineering_expand_all()` and `feature_engineering_expand_basic()`
- Targets go in `set_freqai_targets()`
- Start with LightGBMRegressor before trying complex models
- Train on BTC/USDT first to validate pipeline

### BACKTEST Agent
**Triggers:** run backtest, test strategy, validate results, analyze performance, backtest report
**Pre-read:** `06-project-context.md`, `03-backtesting-optimization.md`
**Key docs:** `16-backtesting.md`, `31-advanced-backtesting.md`, `34-lookahead-analysis.md`, `35-recursive-analysis.md`
**Rules:**
- Use `config-backtest.json` (StaticPairList) for reproducible backtests
- Always specify `--timerange` to avoid unbounded runs
- Run lookahead-analysis on any new strategy before trusting results
- Compare against baseline (buy-and-hold or previous strategy version)
- Check for survivorship bias in pair selection

### HYPEROPT Agent
**Triggers:** optimize parameters, tune strategy, hyperopt, find best settings, parameter search
**Pre-read:** `06-project-context.md`, `03-backtesting-optimization.md`
**Key docs:** `17-hyperopt.md`, `37-advanced-hyperopt.md`
**Rules:**
- Backup existing `.json` result files before running
- Use `--spaces buy sell roi stoploss trailing` (specify explicitly)
- Prefer `SharpeHyperOptLossDaily` unless user specifies otherwise
- Start with 500 epochs, increase if search space is large
- Validate hyperopt results with out-of-sample backtest on different timerange
- Watch for overfitting: if in-sample >> out-of-sample, reduce parameter count

### INFRA Agent
**Triggers:** Docker, config, exchange setup, deployment, API setup, Telegram setup, ports, volumes
**Pre-read:** `06-project-context.md`, `04-configuration-infrastructure.md`
**Key docs:** `01-docker-quickstart.md`, `04-configuration.md`, `28-exchanges.md`, `11-telegram-usage.md`, `13-rest-api.md`, `32-advanced-setup.md`
**Rules:**
- Bot runs on port 8082 (mapped to container 8080), Jupyter on 8888
- Config changes require bot restart (`docker compose down && docker compose up -d`)
- Never expose API keys — check diffs before committing config files
- Test config changes in dry_run mode first

### DATA Agent
**Triggers:** download data, check data, data quality, Telegram commands, REST API queries, SQL queries, pair lists
**Pre-read:** `06-project-context.md`, `05-data-operations.md`
**Key docs:** `15-data-download.md`, `10-bot-usage.md`, `11-telegram-usage.md`, `13-rest-api.md`, `40-sql-cheatsheet.md`, `09-plugins.md`
**Rules:**
- Download data with `--timeframe 5m 15m 1h 4h` to cover all informative timeframes
- Data is cached in `user_data/data/` — check existing data before re-downloading
- Use `--timerange` to avoid downloading unnecessary history
- VolumePairList (live config) vs StaticPairList (backtest config) — don't mix them

## Workflow Patterns

### New Strategy
1. **STRATEGY** — Write strategy with hyperoptable parameters
2. **BACKTEST** — Initial backtest on training timerange
3. **BACKTEST** — Run lookahead-analysis to detect bias
4. **HYPEROPT** — Optimize parameters (backup existing results first)
5. **BACKTEST** — Out-of-sample validation on unseen timerange

### Add FreqAI to Strategy
1. **FREQAI** — Add feature engineering and target functions
2. **INFRA** — Update config with FreqAI parameters
3. **BACKTEST** — Train and backtest on BTC/USDT first
4. **FREQAI** — Iterate on features/model based on results

### Optimize Existing Strategy
1. **DATA** — Verify data coverage for target timerange
2. **BACKTEST** — Establish baseline performance
3. **HYPEROPT** — Run optimization (backup results first)
4. **BACKTEST** — Validate on out-of-sample data

### Debug Strategy Issue
1. **STRATEGY** — Read strategy code, identify the issue
2. **BACKTEST** — Reproduce the problem with a targeted backtest
3. **STRATEGY** — Fix the code
4. **BACKTEST** — Verify the fix

## Key File Paths

```
/mnt/nvme/FreqTrade/repo/ft_userdata/
├── docker-compose.yml                          # Bot container config
├── user_data/
│   ├── config.json                             # Live/dry-run config
│   ├── config-backtest.json                    # Backtest config
│   ├── config-backtest-freqai.json             # FreqAI backtest config
│   ├── strategies/                             # All strategy files
│   │   ├── LSv3_F.py                           # PRODUCTION strategy
│   │   └── *.json                              # Hyperopt results
│   ├── data/                                   # OHLCV data cache
│   ├── logs/                                   # Bot logs
│   └── models/                                 # FreqAI trained models
/mnt/nvme/FreqTrade/repo/.claude/agents/
│   ├── strategy.md                             # STRATEGY agent prompt
│   ├── freqai.md                               # FREQAI agent prompt
│   ├── backtest.md                             # BACKTEST agent prompt
│   ├── hyperopt.md                             # HYPEROPT agent prompt
│   ├── infra.md                                # INFRA agent prompt
│   └── data.md                                 # DATA agent prompt
/mnt/nvme/FreqTrade/repo/doc/freqtrade_official_documentation_extracted_for_claude/
│   └── 00-index.md                             # Doc file listing
/home/poop/.claude/projects/-mnt-nvme/memory/
│   ├── 01-strategy-development.md              # Strategy patterns
│   ├── 02-freqai-machine-learning.md           # FreqAI reference
│   ├── 03-backtesting-optimization.md          # Backtest/hyperopt
│   ├── 04-configuration-infrastructure.md      # Config/Docker
│   ├── 05-data-operations.md                   # Data/Telegram/API
│   └── 06-project-context.md                   # Repo state (read first)
```
