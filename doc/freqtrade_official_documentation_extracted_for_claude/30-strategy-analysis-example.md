# FreqTrade Documentation: Strategy Analysis Example

> Source: https://www.freqtrade.io/en/stable/strategy_analysis_example/

## Overview

A step-by-step guide for debugging strategies using Jupyter notebooks with visualization and statistical tools. The guide assumes SampleStrategy with 5-minute Binance candle data.

## Code Patterns

### Load Historical Data

```python
from freqtrade.data.history import load_pair_history

candles = load_pair_history(
    datadir=datadir,
    timeframe=timeframe,
    pair=pair,
    data_format=data_format,
    candle_type=candle_type
)
```

### Run Strategy Analysis

```python
from freqtrade.resolvers import StrategyResolver

strategy = StrategyResolver.load_strategy(config)
df = strategy.analyze_ticker(candles, {"pair": pair})
```

### Load Backtest Results

```python
from freqtrade.data.btanalysis import load_backtest_data, load_backtest_stats

# Load individual trades
trades = load_backtest_data(backtest_dir)

# Load full statistics (results per pair, drawdown, market change)
stats = load_backtest_stats(backtest_dir)
```

### Load Live/Dry-Run Trades from Database

```python
from freqtrade.data.btanalysis import load_trades_from_db

trades = load_trades_from_db("sqlite:///tradesv3.sqlite")
```

### Analyze Trade Parallelism

```python
from freqtrade.data.btanalysis import analyze_trade_parallelism

parallel = analyze_trade_parallelism(trades, timeframe)
```

### Generate Candlestick Chart

```python
from freqtrade.plot.plotting import generate_candlestick_graph

fig = generate_candlestick_graph(
    pair=pair,
    data=data,
    trades=trades,
    indicators1=indicators1,
    indicators2=indicators2
)
```

## Key API Functions

| Function | Module | Purpose |
|----------|--------|---------|
| `load_pair_history()` | `freqtrade.data.history` | Load OHLCV candle data for a pair |
| `StrategyResolver.load_strategy()` | `freqtrade.resolvers` | Instantiate a strategy from config |
| `strategy.analyze_ticker()` | Strategy instance | Run indicator calculation and signal generation |
| `load_backtest_data()` | `freqtrade.data.btanalysis` | Load trade list from backtest results |
| `load_backtest_stats()` | `freqtrade.data.btanalysis` | Load full statistics from backtest |
| `load_trades_from_db()` | `freqtrade.data.btanalysis` | Load trades from SQLite database |
| `analyze_trade_parallelism()` | `freqtrade.data.btanalysis` | Calculate concurrent open trades over time |
| `generate_candlestick_graph()` | `freqtrade.plot.plotting` | Create interactive Plotly candlestick chart |

## Practical Usage Notes

- **"Startup" data** at the beginning of the DataFrame may contain NaN values in indicator columns. This is expected behavior as indicators need a warmup period.
- Indicator columns may use **different units**, which can affect `crossed*()` function accuracy (e.g., comparing RSI values with price values).
- **Multiple consecutive buy signals** do not guarantee corresponding trades; execution depends on `max_open_trades` slots being available.
- Filter to **specific date ranges** for better performance when visualizing large datasets.
- Statistics include: results per pair, pairlist performance, market change, and maximum drawdown.
- The `load_backtest_stats()` function returns a dictionary with comprehensive backtest metrics including strategy-level and pair-level results.
