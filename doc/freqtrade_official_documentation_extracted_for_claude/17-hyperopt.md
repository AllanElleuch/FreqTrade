# FreqTrade Documentation: Hyperopt

> Source: https://www.freqtrade.io/en/stable/hyperopt/

## Key Concepts

- Hyperparameter optimization using the Optuna library with Bayesian optimization (NSGAIIISampler) to find optimal strategy parameters through automated backtesting.
- Two conceptual types of parameters: **Guards** (conditions that restrict entry, e.g., "never enter if ADX < 10") and **Triggers** (signals that initiate trades, e.g., "enter when EMA5 crosses EMA10").
- Loss functions evaluate each parameter combination to guide the optimizer.

## Parameter Types

- `IntParameter` -- integer values with upper/lower bounds
- `DecimalParameter` -- floating-point with limited decimal precision (default 3)
- `RealParameter` -- floating-point without precision limits
- `CategoricalParameter` -- predetermined choices
- `BooleanParameter` -- shorthand for `CategoricalParameter([True, False])`

Each parameter supports `optimize` (include in optimization, default True) and `load` (use previous results as starting values, default True) options.

## Naming Conventions

Parameters named with `buy_*`, `sell_*`, `enter_*`, `exit_*`, or `protection_*` prefixes are automatically assigned to spaces. Explicit `space` parameter overrides this.

## Strategy Pattern

```python
class MyStrategy(IStrategy):
    buy_ema_short = IntParameter(3, 50, default=5)
    buy_ema_long = IntParameter(15, 200, default=50)

    def populate_indicators(self, dataframe, metadata):
        for val in self.buy_ema_short.range:
            dataframe[f'ema_short_{val}'] = ta.EMA(dataframe, timeperiod=val)
        return dataframe

    def populate_entry_trend(self, dataframe, metadata):
        conditions.append(qtpylib.crossed_above(
            dataframe[f'ema_short_{self.buy_ema_short.value}'],
            dataframe[f'ema_long_{self.buy_ema_long.value}']
        ))
```

## Built-in Loss Functions

ShortTradeDurHyperOptLoss, OnlyProfitHyperOptLoss, SharpeHyperOptLoss/Daily, SortinoHyperOptLoss/Daily, MaxDrawDownHyperOptLoss/Relative, MaxDrawDownPerPairHyperOptLoss, CalmarHyperOptLoss, ProfitDrawDownHyperOptLoss, MultiMetricHyperOptLoss.

## Primary Command

```bash
freqtrade hyperopt --config config.json --hyperopt-loss <lossname> \
  --strategy <strategyname> -e 500 --spaces all
```

## Key Flags

- `-e / --epochs INT` -- optimization iterations (default 100)
- `-j / --job-workers` -- parallel processes (-1 = all CPUs, -2 = all minus one)
- `--spaces` -- what to optimize: buy, sell, enter, exit, roi, stoploss, trailing, protection, trades, default, all, custom_space
- `--random-state INT` -- seed for reproducibility
- `--min-trades INT` -- minimum trades per evaluation (default 1)
- `--early-stop INT` -- stop if no improvement after N epochs
- `--timerange` -- data range (e.g., 20210101-20210201)
- `--analyze-per-epoch` -- recalculate indicators per epoch (memory saving)
- `--print-all`, `--print-json`, `--no-color`, `--eps`, `--enable-protections`

## Protection Optimization

```python
cooldown_lookback = IntParameter(2, 48, default=5, space="protection")

@property
def protections(self):
    return [{"method": "CooldownPeriod", "stop_duration_candles": self.cooldown_lookback.value}]
```

## Result Application

Hyperopt exports to `StrategyName.json`. Precedence: config file > parameter JSON file > strategy `*_params` > parameter defaults.

## ROI/Stoploss/Trailing

Default ROI has 4 steps. Stoploss range: -0.35 to -0.02. Trailing parameters include `trailing_stop`, `trailing_stop_positive`, `trailing_stop_positive_offset`, and `trailing_only_offset_is_reached`.

## Practical Usage Notes

- Recommended workflow: optimize entry first (`--spaces buy`), then exit, then ROI/stoploss/trailing separately.
- Hyperoptable parameters must NOT appear in `populate_indicators()` -- use them in `populate_entry_trend()` and `populate_exit_trend()`.
- The first 30 epochs are random to establish search direction; use `--random-state` for reproducibility.
- Out-of-memory mitigations: reduce pairs, narrow timerange, avoid `--timeframe-detail`, reduce `-j`, use `--analyze-per-epoch`.
- Crashes on single-CPU machines; not supported on Raspberry Pi.
- Utility commands: `hyperopt-list` and `hyperopt-show` for reviewing results.
