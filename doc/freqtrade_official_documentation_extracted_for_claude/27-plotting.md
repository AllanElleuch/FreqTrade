# FreqTrade Documentation: Plotting

> Source: https://www.freqtrade.io/en/stable/plotting/

## Overview

FreqTrade provides two primary plotting tools:

- **plot-dataframe**: Candlestick charts with volume, indicators, and trade signals.
- **plot-profit**: Profit analysis, drawdown visualization, and trade parallelism.

**Important:** Both tools are **deprecated and in maintenance mode**. FreqUI is the recommended replacement for visualization.

## Installation

```bash
pip install -U -r requirements-plot.txt
```

## Commands

### Plot Candlestick Data with Indicators

```bash
freqtrade plot-dataframe -p BTC/ETH --strategy AwesomeStrategy
```

### Plot Profit Analysis

```bash
freqtrade plot-profit -p BTC/ETH --timerange 20180801-20180805
```

## Key Parameters for `plot-dataframe`

| Parameter | Description |
|-----------|-------------|
| `-p/--pairs` | Specify trading pairs |
| `--indicators1` | Indicators to display on main chart (overlay on candles) |
| `--indicators2` | Indicators to display in subplot below main chart |
| `--plot-limit INT` | Cap number of plotting points (default: 750) |
| `--trade-source {DB,file}` | Choose trade data source |
| `--timerange` | Filter time period |
| `--no-trades` | Exclude trade markers from chart |

## Strategy `plot_config`

Strategies can define a `plot_config` property to customize visualization. The config uses `main_plot` and `subplots` dictionaries specifying:

- Per-indicator colors
- Fill areas between indicator pairs
- Scatter/bar plot types
- Custom Plotly parameters

**Note:** `plotly` parameters are **incompatible with FreqUI** -- they only work with the standalone plotting tools.

### Example `plot_config` Structure

```python
@property
def plot_config(self):
    return {
        "main_plot": {
            "sma": {"color": "red"},
            "ema": {"color": "blue"},
        },
        "subplots": {
            "RSI": {
                "rsi": {"color": "green"},
            },
            "MACD": {
                "macd": {"color": "blue"},
                "macdsignal": {"color": "orange"},
            },
        },
    }
```

## Visual Elements (Chart Markers)

| Marker | Meaning |
|--------|---------|
| Green triangles | Buy signals |
| Red triangles | Sell signals |
| Cyan circles | Trade entries |
| Red squares | Loss exits |
| Green squares | Profitable exits |

Bollinger Bands are auto-rendered when the corresponding columns exist in the DataFrame.

## Practical Usage Notes

- Position adjustments may show initial buy prices outside candle ranges.
- Indicator columns must exist in the strategy DataFrame or they will be **silently ignored**.
- The "store file and open browser" workflow is considered unintuitive by the developers -- this is one reason FreqUI is preferred.
- Output files are HTML files that can be opened in any web browser.
