# FreqTrade Documentation: FreqAI Introduction

> Source: https://www.freqtrade.io/en/stable/freqai/

## Key Concepts

- FreqAI is an open-source ML automation framework integrated with Freqtrade. It automates training predictive models to generate market forecasts. It is non-profit, has no tokens, and sells no signals.
- **Self-adaptive retraining:** Models retrain during live deployments to adjust to evolving market conditions.
- **Feature engineering:** Generates large feature sets (10,000+ features) from user-defined strategies.
- **High performance:** Threading for parallel retraining + GPU support.
- **Realistic backtesting:** Historical testing with automated periodic retraining to avoid look-ahead bias.
- **Extensibility:** Supports classifiers, regressors, CNNs, reinforcement learning across multiple ML libraries.
- **Outlier detection, crash resilience, automatic normalization, PCA, fleet deployment** (consumer/producer bot architecture).

## ML Terminology Definitions

- **Features** -- input vectors
- **Labels** -- prediction targets
- **Training** -- the process of fitting a model
- **Train data vs. Test data** -- data splits for model fitting and evaluation
- **Inferencing** -- generating predictions from a trained model

## Quick Start

```bash
freqtrade trade --config config_examples/config_freqai.example.json \
  --strategy FreqaiExampleStrategy \
  --freqaimodel LightGBMRegressor \
  --strategy-path freqtrade/templates
```

## Installation

```bash
pip install -r requirements-freqai.txt
```

## Docker Images

- `freqtradeorg/freqtrade:stable_freqai` -- core FreqAI
- `freqtradeorg/freqtrade:stable_freqaitorch` -- PyTorch support
- `freqtradeorg/freqtrade:stable_freqairl` -- reinforcement learning

## Template Files

- Strategy: `freqtrade/templates/FreqaiExampleStrategy.py`
- Model: `freqtrade/freqai/prediction_models/LightGBMRegressor.py`
- Config: `config_examples/config_freqai.example.json`

## Practical Usage Notes

- Dynamic VolumePairlists are incompatible -- FreqAI requires all training data pre-downloaded at startup. Use static VolumePairlist or ShufflePairlist with constant pair count.
- CatBoost unavailable on ARM devices (Raspberry Pi).
- The example strategy is for benchmarking/testing, not production.
- Published in Journal of Open Source Software (2022), DOI: 10.21105/joss.04864.
