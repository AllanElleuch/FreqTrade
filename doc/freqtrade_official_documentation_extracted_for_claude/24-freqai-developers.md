# FreqTrade Documentation: FreqAI Developer Guide

> Source: https://www.freqtrade.io/en/stable/freqai-developers/

## Key Concepts

- Internal architecture of FreqAI with three core components and the file/data structure for model persistence.

## Three Core Architectural Components

### 1. IFreqaiModel

A singular persistent object containing all logic to collect, store, process data, engineer features, run training, and perform inference. Built-in models extend this class. Developers override `fit()`, `train()`, `predict()`, and `data_cleaning_train/predict()` methods.

### 2. FreqaiDataKitchen

A non-persistent object created uniquely per asset/model. Contains metadata and data processing utilities. Destroyed and recreated for each training/inference cycle.

### 3. FreqaiDataDrawer

A singular persistent object holding all historical predictions, models, and save/load methods. Manages crash resilience through automatic data reloading and corruption detection.

## File Structure Under `user_data_dir/models/`

Each model gets a subdirectory based on the `identifier` config parameter, containing:

- `config_*.json` -- model configuration snapshot
- `historic_predictions.pkl` -- live prediction history with automatic backup/corruption detection
- `pair_dictionary.json` -- training queue and model locations
- `sub-train-*_TIMESTAMP/` folders with:
  - `*_metadata.json` -- normalization parameters and feature metadata
  - `*_model.*` -- serialized model (joblib, zip, hd5 formats)
  - `*_pca_object.pkl` -- PCA transformation object
  - `*_svm_model.pkl` -- SVM outlier detection model
  - `*_trained_df.pkl` -- training feature dataframe
  - `*_trained_dates_df.pkl` -- associated training dates

## Practical Usage Notes

- The inheritance pattern means custom models only need to override the specific methods they want to change.
- The file structure auto-regenerates from configuration, enabling crash resilience.
- `historic_predictions.pkl` has automatic backup and corruption detection.
- The `identifier` parameter is the key organizational unit -- all files for a model run are grouped under it.
- `FreqaiDataKitchen` being non-persistent means each training cycle starts fresh, preventing state leakage between assets.
- Custom models placed in `user_data/freqaimodels/` should use unique names to avoid overriding built-in prediction models.
