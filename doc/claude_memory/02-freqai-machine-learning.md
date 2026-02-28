# FreqAI Machine Learning Reference

## Overview
FreqAI is FreqTrade's built-in ML framework. It handles feature engineering, model training, prediction, and retraining automatically. Models are trained on historical data and retrained periodically.

## Enabling FreqAI
1. Set `"freqai": {"enabled": true}` in config.json
2. Strategy must call `self.freqai.start(dataframe, metadata, self)` in `populate_indicators()`
3. Implement feature engineering methods and target function

## Config Structure
```json
{
  "freqai": {
    "enabled": true,
    "purge_old_models": 2,
    "train_period_days": 30,
    "backtest_period_days": 7,
    "identifier": "my_model",
    "live_retrain_hours": 1,
    "expiration_hours": 2,
    "fit_live_predictions_candles": 100,
    "feature_parameters": {
      "include_timeframes": ["5m", "15m", "1h", "4h"],
      "include_corr_pairlist": ["BTC/USDT:USDT", "ETH/USDT:USDT"],
      "label_period_candles": 24,
      "include_shifted_candles": 2,
      "indicator_periods_candles": [10, 20],
      "DI_threshold": 0.9,
      "weight_factor": 0.9,
      "noise_standard_deviation": 0.05
    },
    "data_split_parameters": {
      "test_size": 0.25,
      "shuffle": false
    },
    "model_training_parameters": {
      "n_estimators": 800,
      "max_depth": 10,
      "learning_rate": 0.02
    }
  }
}
```

## Feature Engineering Methods (Strategy)

### feature_engineering_expand_all(dataframe, period, metadata, **kwargs)
Called for each period in `indicator_periods_candles` and each timeframe. Indicators here get auto-expanded.
```python
def feature_engineering_expand_all(self, dataframe, period, metadata, **kwargs):
    dataframe["%-rsi-period"] = ta.RSI(dataframe, timeperiod=period)
    dataframe["%-mfi-period"] = ta.MFI(dataframe, timeperiod=period)
    dataframe["%-adx-period"] = ta.ADX(dataframe, timeperiod=period)
    dataframe["%-sma-period"] = ta.SMA(dataframe, timeperiod=period)
    dataframe["%-ema-period"] = ta.EMA(dataframe, timeperiod=period)
    return dataframe
```
**Convention:** Feature columns MUST start with `%` prefix. The `-period` suffix gets auto-replaced.

### feature_engineering_expand_basic(dataframe, metadata, **kwargs)
Called once per timeframe (no period expansion).
```python
def feature_engineering_expand_basic(self, dataframe, metadata, **kwargs):
    dataframe["%-pct-change"] = dataframe["close"].pct_change()
    dataframe["%-raw_volume"] = dataframe["volume"]
    dataframe["%-raw_price"] = dataframe["close"]
    return dataframe
```

### feature_engineering_standard(dataframe, metadata, **kwargs)
Called once on the base timeframe only. For non-expandable features.
```python
def feature_engineering_standard(self, dataframe, metadata, **kwargs):
    dataframe["%-day_of_week"] = dataframe["date"].dt.dayofweek
    dataframe["%-hour_of_day"] = dataframe["date"].dt.hour
    return dataframe
```

### set_freqai_targets(dataframe, metadata, **kwargs)
Define training targets. Column names MUST start with `&`.
```python
def set_freqai_targets(self, dataframe, metadata, **kwargs):
    # Regression: predict future price change
    dataframe["&-s_close"] = (
        dataframe["close"].shift(-self.freqai_info["feature_parameters"]["label_period_candles"])
        / dataframe["close"] - 1
    )
    # Classification: predict direction
    dataframe["&s-up_or_down"] = np.where(
        dataframe["close"].shift(-24) > dataframe["close"], "up", "down"
    )
    return dataframe
```

## Available Models
### Regression
- `LightGBMRegressor` - Default, fast, good accuracy
- `XGBoostRegressor` - Alternative gradient boosting
- `CatboostRegressor` - Handles categorical features
- `PyTorchMLPRegressor` - Neural network (MLP)
- `PyTorchTransformerRegressor` - Transformer architecture

### Classification
- `LightGBMClassifier`
- `XGBoostClassifier`
- `CatboostClassifier`
- `PyTorchMLPClassifier`

### Reinforcement Learning
- `ReinforcementLearner` (base)
- `ReinforcementLearner_multiproc` - Multi-process training
- `ReinforcementLearner_test_3ac` - 3-action (long/short/neutral)
- `ReinforcementLearner_test_4ac` - 4-action (long/short/neutral/exit)

## Using Predictions in Strategy
```python
def populate_entry_trend(self, dataframe, metadata):
    # Regression model
    dataframe.loc[
        (dataframe['do_predict'] == 1) &  # Model confidence
        (dataframe['&-s_close'] > 0.01),   # Predicted 1% up
        'enter_long'] = 1

    # Classification model
    dataframe.loc[
        (dataframe['do_predict'] == 1) &
        (dataframe['&s-up_or_down'] == 'up'),
        'enter_long'] = 1
    return dataframe
```

## Key Columns Added by FreqAI
- `do_predict` - 1 if model is confident, 0 if outlier detected
- `&-s_close` (or custom target) - Model prediction value
- `DI_values` - Dissimilarity Index (outlier score)

## Outlier Detection
- **DI_threshold:** Dissimilarity Index threshold. Points with DI > threshold get `do_predict = 0`
- **use_SVM_to_remove_outliers:** SVM-based outlier detection during training
- **use_DBSCAN_to_remove_outliers:** DBSCAN clustering for outlier detection

## Reinforcement Learning
Config additions:
```json
{
  "freqai": {
    "rl_config": {
      "train_cycles": 25,
      "max_trade_duration_candles": 300,
      "model_type": "PPO",
      "policy_type": "MlpPolicy",
      "max_training_drawdown_pct": 0.02,
      "net_arch": [128, 128],
      "randomize_starting_position": true
    }
  }
}
```
RL models learn to take actions (enter_long, enter_short, exit, hold) based on reward functions.

## This Repo's FreqAI Setup
- Config: `config-backtest-freqai.json`
- Identifier: "LSV3"
- Training: 30 days, validation: 7 days
- Timeframes: 5m, 15m, 4h
- Correlated pair: BTC/USDT
- Strategies with FreqAI: LSv3 (hybrid), FreqaiExampleStrategy (pure ML), FreqaiExampleHybridStrategy (TA+ML)

## Detailed Documentation
See files in `/mnt/nvme/FreqTrade/repo/doc/freqtrade_official_documentation_extracted_for_claude/`:
- `18-freqai-introduction.md` - Overview
- `19-freqai-configuration.md` - Configuration details
- `20-freqai-parameter-table.md` - All parameters
- `21-freqai-feature-engineering.md` - Feature engineering
- `22-freqai-running.md` - Running models
- `23-freqai-reinforcement-learning.md` - RL guide
- `24-freqai-developers.md` - Developer guide
