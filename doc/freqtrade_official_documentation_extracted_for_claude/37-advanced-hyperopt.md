# FreqTrade Documentation: Advanced Hyperopt

> Source: https://www.freqtrade.io/en/stable/advanced-hyperopt/

## Custom Loss Functions

Create a class with `hyperopt_loss_function` method returning a float (lower = better). Activate with `--hyperopt-loss CustomClassName`.

```python
from freqtrade.optimize.hyperopt import IHyperOptLoss

class MyLoss(IHyperOptLoss):
    @staticmethod
    def hyperopt_loss_function(results, trade_count, min_date, max_date,
                                config, processed, backtest_stats,
                                starting_balance, *args, **kwargs):
        # return float, lower is better
        return -results['profit_ratio'].sum()
```

### Available Parameters

- `results` -- DataFrame of trade results
- `trade_count` -- number of trades
- `min_date` / `max_date` -- date range of the backtest
- `config` -- bot configuration
- `processed` -- processed dataframes
- `backtest_stats` -- full backtest statistics
- `starting_balance` -- initial balance

Called once per epoch -- optimize for performance.

## Overriding Pre-defined Spaces

Define a nested `HyperOpt` class in your strategy to override search spaces:

- `stoploss_space()` -- custom stop-loss ranges
- `roi_space()` -- ROI table parameters
- `generate_roi_table(params)` -- construct ROI dict from params
- `trailing_space()` -- trailing stop parameters
- `max_open_trades_space()` -- trade limit ranges

All overrides are optional and can be mixed/matched.

## Dynamic Parameters

Create parameters at runtime in `bot_start()`:

```python
def bot_start(self, **kwargs) -> None:
    self.buy_adx = IntParameter(20, 30, default=30, optimize=True)
```

**Limitation:** These will not appear in `list-strategies` parameter count.

## Custom Samplers (Estimators)

Via `generate_estimator()` method. Available Optuna samplers:

- `TPESampler` -- Tree-structured Parzen Estimator
- `GPSampler` -- Gaussian Process
- `CmaEsSampler` -- CMA-ES
- `NSGAIISampler` -- NSGA-II
- `NSGAIIISampler` -- NSGA-III (recommended)
- `QMCSampler` -- Quasi-Monte Carlo
- External: `AutoSampler` from Optunahub (`pip install optunahub cmaes torch scipy`)

## Space Types

| Type | Use Case | Note |
|------|----------|------|
| `Categorical` | Discrete categories | `Categorical(['a', 'b'], name="cat")` |
| `Integer` | Whole numbers | `Integer(1, 10, name='rsi')` |
| `SKDecimal` | Limited-precision decimals | **Recommended over Real** |
| `Real` | Full-precision decimals | Slower due to unnecessary precision |

```python
from freqtrade.optimize.space import Categorical, Integer, SKDecimal, Real
```

## Practical Notes

- Always keep `*args` and `**kwargs` in signatures for forward compatibility.
- `SKDecimal` is recommended over `Real` in almost all cases for faster optimization.
