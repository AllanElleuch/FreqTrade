# FreqTrade Documentation: FreqAI Feature Engineering

> Source: https://www.freqtrade.io/en/stable/freqai-feature-engineering/

## Key Concepts

- Feature definition through strategy functions, data pipeline architecture, outlier detection methods (DI, SVM, DBSCAN, PCA), and custom data transformations.

## Core Feature Engineering Functions

- `feature_engineering_expand_all()` -- expands across indicator periods, timeframes, shifted candles, and correlated pairs
- `feature_engineering_expand_basic()` -- expands across timeframes, shifted candles, correlated pairs (excludes indicator periods)
- `feature_engineering_standard()` -- called once for final custom features; processes all previously created features
- `set_freqai_targets()` -- establishes model targets (must use `&` prefix)

Features for model training must use `%` prefix. Intermediate calculations (not model inputs) omit prefixes.

## Configuration Parameters in `feature_parameters`

- `include_timeframes` -- e.g., `["5m", "15m", "4h"]`
- `include_corr_pairlist` -- correlated pairs
- `label_period_candles` -- candles for target calculation
- `include_shifted_candles` -- historical candle count
- `indicator_periods_candles` -- e.g., `[10, 20]`
- `weight_factor` -- exponential recency weighting: `W_i = exp(-i / (alpha * n))`

## Total Feature Count Formula

`timeframes x features x pairs x shifted_candles x indicator_periods`

Example: 3 timeframes x 3 features x 3 pairs x 2 shifted_candles x 2 indicator_periods = 108 features.

## Metadata Dictionary

All functions receive `metadata` with `pair`, `tf` (timeframe), and `period`, enabling conditional feature application per timeframe.

## Extra Returns Per Training

```python
dk.data['extra_returns_per_train']['my_new_value'] = some_value
```

Requires config: `"extra_returns_per_train": {"total_profit": 4}`

## Data Pipeline Architecture

Default pipeline: `MinMaxScaler(-1, 1)` then `VarianceThreshold` (removes zero-variance columns). Customizable via `define_data_pipeline()` and `define_label_pipeline()` using the `DataSieve` library (sklearn-compatible API).

Custom transforms inherit from `datasieve.transforms.base_transform.BaseTransform` and implement:
- `fit(X, y=None, sample_weight=None, feature_list=None)`
- `transform(X, y=None, sample_weight=None, feature_list=None, outlier_check=False)`
- `inverse_transform(X, y=None, sample_weight=None, feature_list=None)`

## Outlier Detection Methods

### 1. Dissimilarity Index (DI)

Activated via `"DI_threshold": 1`. Formula: `DI_k = d_k / d_bar` where `d_k` is minimum distance to training data. Higher thresholds allow more extrapolation. Returned in `DI_values` column.

### 2. SVM

`"use_SVM_to_remove_outliers": true`. Configurable via `svm_params` with `shuffle` (default False) and `nu` (0-1, outlier proportion).

### 3. DBSCAN

`"use_DBSCAN_to_remove_outliers": true`. Uses sklearn's implementation with `min_samples = candle_count / 4` and auto-computed `eps`.

### 4. PCA

`"principal_component_analysis": true`. Reduces dimensionality maintaining >= 0.999 explained variance.

### 5. Noise Addition

`"noise_standard_deviation": 0.1`.

## Practical Usage Notes

- Start from `templates/FreqaiExampleStrategy.py` to ensure proper naming conventions.
- The `%` prefix is critical -- features without it are not fed to the model.
- Use metadata conditionals to create timeframe-specific features.
- Migration note: users with custom `IFreqaiModel` classes using old `data_cleaning_train/predict()` methods should consult migration docs.
