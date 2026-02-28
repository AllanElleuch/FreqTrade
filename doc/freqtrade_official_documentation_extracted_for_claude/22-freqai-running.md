# FreqTrade Documentation: FreqAI Running

> Source: https://www.freqtrade.io/en/stable/freqai-running/

## Key Concepts

- Two deployment modes (live/dry and backtesting), data management, model training control, hyperopt integration, and TensorBoard visualization.

## Live/Dry Deployment

```bash
freqtrade trade --strategy FreqaiExampleStrategy \
  --config config_freqai.example.json --freqaimodel LightGBMRegressor
```

## Backtesting

```bash
freqtrade backtesting --strategy FreqaiExampleStrategy \
  --config config_freqai.example.json --freqaimodel LightGBMRegressor \
  --timerange 20210501-20210701
```

## Key Runtime Parameters

- `identifier` -- model name for save/load
- `live_retrain_hours` -- minimum hours between retraining (default: continuous)
- `expiration_hours` -- max model age before stopping predictions
- `train_period_days` -- training window
- `backtest_period_days` -- testing window (supports fractional days)
- `startup_candle_count` -- initial candles before predictions
- `purge_old_models` -- number of recent models retained
- `continual_learning` -- sequential model training from previous state (automatically disables PCA)
- `save_backtest_models` -- preserve models for later live use
- `activate_tensorboard` -- training monitoring (default: True)

## Data Management

Live mode auto-downloads data. Backtesting requires pre-downloaded data covering `train_period_days + startup_candle_count` before the backtest range. Predictions stored in `historic_predictions.pkl` for crash recovery.

## Hyperopt Integration

```bash
freqtrade hyperopt --hyperopt-loss SharpeHyperOptLoss \
  --strategy FreqaiExampleStrategy
```

Restriction: Can only optimize entry/exit thresholds -- cannot optimize `feature_engineering_*()` or `set_freqai_targets()` functions. DI thresholds are an example of hyperoptable FreqAI parameters.

## TensorBoard

```bash
tensorboard --logdir user_data/models/unique-id
```

Accessible at `127.0.0.1:6060`.

## Practical Usage Notes

- Backtesting saves predictions by default; re-running with the same config reuses trained models.
- Fractional `backtest_period_days` exponentially increases model training load.
- Continual learning is incompatible with PCA (different feature spaces).
- Retraining simulates live behavior without look-ahead bias.
- `model_training_parameters` accepts any parameter from the chosen ML library (LightGBM, CatBoost, etc.).
