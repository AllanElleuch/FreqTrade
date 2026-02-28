# FreqTrade Documentation: FreqAI Reinforcement Learning

> Source: https://www.freqtrade.io/en/stable/freqai-reinforcement-learning/

## Key Concepts

- RL agents learn trading policies by making autonomous decisions (5 actions: long entry, long exit, short entry, short exit, neutral) and receiving reward signals.
- Unlike regressors/classifiers, RL does not require explicit target labels -- agents develop their own entry/exit policies.
- State information (current profit, position, trade duration) reinforces learning in live/dry runs.
- RL adds "adaptivity and market reactivity that Classifiers and Regressors cannot match."

## Installation

`./setup.sh -i` or Docker image `_freqairl`. Dependencies include `torch` (~700MB).

## RL Config Block

```json
{
    "rl_config": {
        "train_cycles": 25,
        "add_state_info": true,
        "max_trade_duration_candles": 300,
        "max_training_drawdown_pct": 0.02,
        "cpu_count": 8,
        "model_type": "PPO",
        "policy_type": "MlpPolicy",
        "model_reward_parameters": {"rr": 1, "profit_aim": 0.025}
    }
}
```

## Command

```bash
freqtrade trade --freqaimodel ReinforcementLearner \
  --strategy MyRLStrategy --config config.json
```

## Required Strategy Functions

### `set_freqai_targets()`

```python
dataframe["&-action"] = 0
return dataframe
```

### `feature_engineering_standard()`

```python
dataframe[f"%-raw_close"] = dataframe["close"]
dataframe[f"%-raw_open"] = dataframe["open"]
dataframe[f"%-raw_high"] = dataframe["high"]
dataframe[f"%-raw_low"] = dataframe["low"]
```

### `populate_entry_trend()` / `populate_exit_trend()`

```python
# Action mapping: 0=Neutral, 1=Enter Long, 2=Exit Long, 3=Enter Short, 4=Exit Short
enter_long_conditions = [df["do_predict"] == 1, df["&-action"] == 1]
```

## Custom Reward Function

```python
class MyCoolRLModel(ReinforcementLearner):
    class MyRLEnv(Base5ActionRLEnv):
        def calculate_reward(self, action: int) -> float:
            if not self._is_valid(action):
                return -2
            pnl = self.get_unrealized_profit()
            # Custom reward logic
```

## Environment Variables in Reward Functions

- `self._position` -- current position state
- `self._current_tick` -- current candle index
- `self._last_trade_tick` -- last trade timing
- `self.raw_features` -- access to engineered features (naming: `f"%-{feature}_{pair}_{timeframe}"`)
- `self.rl_config` -- configuration parameters

## Three Base Environments

| Environment | Actions |
|-------------|---------|
| `Base3ActionRLEnv` | Hold, Long, Short (supports long-only when `can_short = False`) |
| `Base4ActionRLEnv` | Enter Long, Enter Short, Hold, Exit |
| `Base5ActionRLEnv` | Enter Long, Enter Short, Exit Long, Exit Short, Hold |

## TensorBoard with Custom Logging

```python
self.tensorboard_log("event_name")        # Incremented metrics
self.tensorboard_log("float_metric", 0.23) # Float values
```

```bash
tensorboard --logdir user_data/models/unique-id  # Access at 127.0.0.1:6006
```

## Practical Usage Notes

- The default reward function is for testing only -- not designed for production. Users must create custom reward functions saved to `user_data/freqaimodels`.
- Reward function design principles: continuously differentiable, well-scaled, small penalties for common events rather than large penalties for rare ones, scale penalties with severity.
- The RL training environment is simplified and does NOT incorporate strategy logic like `custom_exit`, `custom_stoploss`, leverage controls, etc.
- State information creates live/dry adaptivity unavailable during backtesting.
- `continual_learning: true` continues training from previous model instead of retraining from scratch.
- Only `Base3ActionRLEnv` supports long-only strategies.
