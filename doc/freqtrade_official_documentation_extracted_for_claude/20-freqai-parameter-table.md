# FreqTrade Documentation: FreqAI Parameter Table

> Source: https://www.freqtrade.io/en/stable/freqai-parameter-table/

## Key Concepts

Exhaustive reference of every configuration parameter in FreqAI, organized by category.

## General Parameters (`freqai.*`)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `train_period_days` | Days of training data (sliding window width) | **Required** |
| `backtest_period_days` | Days to inference before sliding window | **Required** |
| `identifier` | Unique model ID | **Required** |
| `live_retrain_hours` | Retraining frequency in dry/live runs | 0 |
| `expiration_hours` | Max model age before stopping predictions | 0 |
| `purge_old_models` | Models retained on disk | 2 |
| `save_backtest_models` | Store models during backtests | False |
| `fit_live_predictions_candles` | Historical candles for target statistics | -- |
| `continual_learning` | Use last model as starting point for new | False |
| `write_metrics_to_disk` | Collect timing/CPU usage as JSON | False |
| `data_kitchen_thread_count` | Threads for data processing | max - 2 |
| `activate_tensorboard` | Enable TensorBoard | True |
| `wait_for_training_iteration_on_reload` | Wait for training to finish before shutdown | True |

## Feature Parameters (`freqai.feature_parameters.*`)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `include_timeframes` | Timeframes for indicator creation | -- |
| `include_corr_pairlist` | Correlated pairs as additional features | -- |
| `label_period_candles` | Candles into the future for labels | -- |
| `include_shifted_candles` | Historical candles to add as features | -- |
| `weight_factor` | Recency weighting (typically < 1) | -- |
| `indicator_periods_candles` | Time periods for indicators | -- |
| `principal_component_analysis` | Enable PCA dimensionality reduction | False |
| `plot_feature_importances` | Create feature importance plots | 0 |
| `DI_threshold` | Dissimilarity Index threshold (> 0 activates) | -- |
| `use_SVM_to_remove_outliers` | SVM-based outlier detection | -- |
| `svm_params` | SKlearn SGDOneClassSVM parameters | -- |
| `use_DBSCAN_to_remove_outliers` | DBSCAN clustering outlier detection | -- |
| `noise_standard_deviation` | Gaussian noise to prevent overfitting | 0 |
| `outlier_protection_percentage` | Max data discarded by outlier detection | 30 |
| `reverse_train_test_order` | Latest data for training, historical for testing | False |
| `shuffle_after_split` | Shuffle train/test sets after splitting | False |
| `buffer_train_data_candles` | Cut beginning/end of training data | 0 |

## Data Split Parameters (`freqai.data_split_parameters.*`)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `test_size` | Fraction used for testing | -- |
| `shuffle` | Shuffle training data | False |

## Model Training Parameters (`freqai.model_training_parameters.*`)

| Parameter | Description |
|-----------|-------------|
| `n_estimators` | Number of boosted trees |
| `learning_rate` | Boosting learning rate |
| `n_jobs` / `thread_count` / `task_type` | Threading and CPU/GPU |

## RL Config Parameters (`freqai.rl_config.*`)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `train_cycles` | Training cycles multiplied by data points | -- |
| `max_trade_duration_candles` | Max trade length guidance | -- |
| `model_type` | SB3 model (PPO, A2C, DQN, TRPO, ARS, RecurrentPPO, MaskablePPO) | -- |
| `policy_type` | SB3 policy type | -- |
| `max_training_drawdown_pct` | Max drawdown during training | 0.8 |
| `cpu_count` | CPUs for RL training | cores - 1 |
| `model_reward_parameters` | Custom reward function params | -- |
| `add_state_info` | Include state in features | False |
| `net_arch` | Hidden layer architecture | [128, 128] |
| `randomize_starting_position` | Random episode start | False |
| `drop_ohlc_from_features` | Exclude OHLC from features | False |
| `progress_bar` | Show training progress | False |

## PyTorch Parameters (`freqai.model_training_parameters.*`)

| Parameter | Default |
|-----------|---------|
| `learning_rate` | 3e-4 |
| `model_kwargs` | {} |
| `trainer_kwargs.n_epochs` | 10 |
| `trainer_kwargs.n_steps` | None |
| `trainer_kwargs.batch_size` | 64 |

## Miscellaneous Parameters

| Parameter | Default |
|-----------|---------|
| `freqai.keras` | False |
| `freqai.conv_width` | 2 |
| `freqai.reduce_df_footprint` | False |
| `freqai.override_exchange_check` | False |

## Practical Usage Notes

- Three parameters are **required**: `train_period_days`, `backtest_period_days`, `identifier`.
- `model_training_parameters` is a flexible dictionary accepting any parameter supported by the chosen ML library.
- Multiple outlier detection methods (SVM, DBSCAN, DI) can be combined but each has different characteristics.
- Thread counts, batch sizes, and learning rates significantly impact training performance.
