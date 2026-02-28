# FreqTrade Backtesting & Optimization Reference

## Backtesting Commands

### Basic Backtest
```bash
freqtrade backtesting \
  --config user_data/config.json \
  --strategy MyStrategy \
  --timerange 20240101-20240601 \
  --timeframe 15m
```

### With FreqAI
```bash
freqtrade backtesting \
  --config user_data/config-backtest-freqai.json \
  --strategy LSv3 \
  --timerange 20240101-20240601 \
  --freqaimodel LightGBMRegressor
```

### Key Flags
- `--timerange YYYYMMDD-YYYYMMDD` - Date range
- `--timeframe 5m|15m|1h|4h|1d` - Candle timeframe
- `--strategy-list Strategy1 Strategy2` - Compare strategies
- `--export trades` - Export trade data for analysis
- `--export-filename user_data/backtest_results/my_test.json`
- `--breakdown month|week|day` - Show results by period
- `--enable-position-stacking` - Allow multiple open trades per pair
- `--disable-max-market-positions` - Remove max_open_trades limit
- `--dry-run-wallet 1000` - Set starting balance
- `--fee 0.001` - Override fee (0.1%)
- `--eps` / `--enable-protections` - Enable protection plugins during backtest
- `--cache none` - Disable result caching

### Docker Backtest
```bash
docker compose run --rm freqtrade backtesting \
  --config /freqtrade/user_data/config-backtest.json \
  --strategy LSv3_F \
  --timerange 20240101-20240601
```

## Backtest Output Metrics
- **Total trades** - Number of completed trades
- **Win/Loss ratio** - Percentage of profitable trades
- **Avg profit** - Average profit per trade
- **Total profit** - Absolute profit
- **Profit factor** - Gross profit / Gross loss (>1 = profitable)
- **Sharpe ratio** - Risk-adjusted return
- **Sortino ratio** - Downside risk-adjusted return
- **Max drawdown** - Largest peak-to-trough decline
- **Calmar ratio** - Annual return / Max drawdown
- **Expectancy** - Average expected profit per trade
- **CAGR** - Compound annual growth rate

## Hyperopt (Hyperparameter Optimization)

### Basic Hyperopt
```bash
freqtrade hyperopt \
  --config user_data/config.json \
  --strategy MyStrategy \
  --hyperopt-loss SharpeHyperOptLossDaily \
  --epochs 500 \
  --spaces buy sell roi stoploss trailing \
  --timerange 20240101-20240601
```

### Loss Functions
- `ShortTradeDurHyperOptLoss` - Minimize trade duration + maximize profit
- `OnlyProfitHyperOptLoss` - Maximize profit only
- `SharpeHyperOptLoss` - Maximize Sharpe ratio
- `SharpeHyperOptLossDaily` - Maximize daily Sharpe ratio (recommended)
- `SortinoHyperOptLoss` / `SortinoHyperOptLossDaily` - Maximize Sortino ratio
- `MaxDrawDownHyperOptLoss` - Minimize drawdown
- `MaxDrawDownRelativeHyperOptLoss` - Minimize relative drawdown
- `CalmarHyperOptLoss` - Maximize Calmar ratio
- `ProfitDrawDownHyperOptLoss` - Balance profit and drawdown

### Hyperopt Spaces
- `buy` - Optimize entry parameters (IntParameter, DecimalParameter, etc.)
- `sell` - Optimize exit parameters
- `roi` - Optimize ROI table
- `stoploss` - Optimize stoploss value
- `trailing` - Optimize trailing stop parameters
- `protection` - Optimize protection plugin parameters
- `trades` - Optimize max_open_trades

### Key Flags
- `--epochs N` - Number of optimization rounds
- `--spaces buy sell` - Which parameter spaces to optimize
- `--min-trades N` - Minimum trades for a valid epoch
- `--print-all` - Print all epoch results
- `--no-color` - Disable colored output
- `--job-workers N` - Parallel workers (-1 = all cores)
- `--random-state N` - Reproducible results

### Apply Hyperopt Results
Results are saved to strategy JSON file (e.g., `LSv3.json`). Can also manually copy values to strategy class.
```bash
# List past results
freqtrade hyperopt-list --config user_data/config.json
# Show specific result
freqtrade hyperopt-show --config user_data/config.json --best
freqtrade hyperopt-show --config user_data/config.json -n 5
```

## Advanced Hyperopt
### Custom Loss Function
```python
from freqtrade.optimize.hyperopt import IHyperOptLoss
class MyLoss(IHyperOptLoss):
    @staticmethod
    def hyperopt_loss_function(results, trade_count, min_date, max_date, config, processed, backtest_stats, **kwargs):
        total_profit = results['profit_ratio'].sum()
        max_drawdown = backtest_stats['max_drawdown_abs']
        return -total_profit + max_drawdown * 2
```

### Custom Hyperopt Space
Override `generate_roi_table`, `roi_space`, `stoploss_space`, `trailing_space` methods.

## Analysis Tools

### Lookahead Analysis
```bash
freqtrade lookahead-analysis \
  --config user_data/config.json \
  --strategy MyStrategy \
  --timerange 20240101-20240301
```
Detects if strategy uses future data (lookahead bias).

### Recursive Analysis
```bash
freqtrade recursive-analysis \
  --config user_data/config.json \
  --strategy MyStrategy \
  --timerange 20240101-20240301
```
Checks for repainting indicators.

### Plotting
```bash
# Plot profit
freqtrade plot-profit --config user_data/config.json --strategy MyStrategy

# Plot pair with indicators
freqtrade plot-dataframe --config user_data/config.json --strategy MyStrategy --pairs BTC/USDT
```

## This Repo's Hyperopt Results
- **LSv3.json:** ROI {0: 0.221, 64: 0.113, 220: 0.022, 504: 0}, stoploss -0.132, trailing 0.198/0.211
- **LSv3_F.json:** ROI {0: 0.243, 70: 0.143, 115: 0.053, 242: 0}, stoploss -0.103, trailing 0.052/0.151

## Detailed Documentation
See files in `/mnt/nvme/FreqTrade/repo/doc/freqtrade_official_documentation_extracted_for_claude/`:
- `16-backtesting.md` - Backtesting guide
- `17-hyperopt.md` - Hyperopt guide
- `31-advanced-backtesting.md` - Advanced analysis
- `34-lookahead-analysis.md` - Lookahead detection
- `35-recursive-analysis.md` - Recursive analysis
- `37-advanced-hyperopt.md` - Custom hyperopt
