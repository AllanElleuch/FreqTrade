# FreqTrade Documentation: FreqAI Configuration

> Source: https://www.freqtrade.io/en/stable/freqai-configuration/

## Key Concepts

- Full configuration setup including strategy implementation functions, DataFrame column naming conventions, model types (regressors, classifiers), PyTorch module integration, and custom model development.

## Minimum Config Requirements

- `enabled`: Boolean to activate FreqAI
- `purge_old_models`: Number of old models to retain
- `train_period_days` / `backtest_period_days`: Training and testing windows
- `identifier`: Unique ID for model tracking (critical for crash resilience)
- `feature_parameters`: Timeframes, correlated pairs, candle periods, shifted candles
- `data_split_parameters`: Test/train split ratio

## Strategy Functions Required

- `populate_indicators()` -- must call `self.freqai.start()` to initiate processing
- `feature_engineering_expand_all()` -- auto-expands features across timeframes and periods; features prefixed with `%`
- `feature_engineering_expand_basic()` -- features without period multiplication
- `feature_engineering_standard()` -- final custom feature engineering step
- `set_freqai_targets()` -- defines labels/targets prefixed with `&`

## DataFrame Column Naming Conventions

| Prefix | Purpose |
|--------|---------|
| `&*` | Training targets/labels; predictions returned under same key |
| `&*_std/mean` | Statistical measures of labels |
| `%*` | Training features; removed after processing |
| `%%*` | Training features retained for plotting |
| `do_predict` | Outlier indicator (-2 to 2) |
| `DI_values` | Dissimilarity Index confidence metric |

## Startup Candle Count

Multiplying `startup_candle_count` by 2 ensures a NaN-free training dataset. Verify via log: "dropped 0 training points due to NaNs."

## Dynamic Target Thresholds Using Z-Scores

```python
dataframe["target_roi"] = dataframe["&-s_close_mean"] + dataframe["&-s_close_std"] * 1.25
```

Alternatively, use `fit_live_predictions_candles` to base thresholds on historical predictions.

## Built-in Models

LightGBMRegressor, LightGBMClassifier, XGBoost Regressor/Classifier. CatBoost available but no longer actively supported since 2025.12.

## Regression Target Example

```python
df['&s-close_price'] = df['close'].shift(-100)
```

## Classification Target Example

```python
df['&s-up_or_down'] = np.where(df["close"].shift(-100) > df["close"], 'up', 'down')
```

## PyTorch Quick Start

```bash
freqtrade trade --config config_examples/config_freqai.example.json \
  --strategy FreqaiExampleStrategy --freqaimodel PyTorchMLPRegressor \
  --strategy-path freqtrade/templates
```

## PyTorch Architecture Hierarchy

1. `BasePyTorchModel` -- implements `train()`, handles normalization
2. `BasePyTorch*` -- implements `predict()` for classifiers/regressors
3. `PyTorch*Classifier/Regressor` -- implements `fit()` with trainer

## Custom PyTorch Model Pattern

```python
class LogisticRegression(nn.Module):
    def __init__(self, input_size: int):
        super().__init__()
        self.linear = nn.Linear(input_size, 1)
        self.activation = nn.Sigmoid()

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.activation(self.linear(x))
```

## Classifier Requirements

Classifiers require: `self.freqai.class_names = ["down", "up"]` in `set_freqai_targets()`.

## Practical Usage Notes

- Features must be defined in `feature_engineering_*()` functions, NOT in `populate_indicators()`.
- Use `print(dataframe.columns)` after `self.freqai.start()` to discover available features.
- Custom models go in `user_data/freqaimodels/` with unique names to avoid overriding builtins.
- PyTorch dropped macOS x64 (Intel) support in version 2.3.
- `torch.compile()` can optimize models but removes eager execution and hides error details.
