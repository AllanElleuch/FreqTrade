# Download DATA
https://www.freqtrade.io/en/stable/data-download/


freqtrade list-data --userdir ~/.freqtrade/user_data/

## Download all pair future

freqtrade download-data --exchange bybit  --pairs BTC/USDT:USDT --timerange 20210101-20240620 -t 4h 1m 5m 15m   --user-data-dir /freqtrade/user_data/
freqtrade download-data --exchange bybit --timerange 20210101-20240620 -t 4h 1m 5m 15m   --user-data-dir /freqtrade/user_data/
rr


freqtrade download-data --exchange bybit   --trading-mode futures --timerange 20210420-20240620   --user-data-dir /freqtrade/user_data/


freqtrade download-data --exchange bybit   --trading-mode futures  --trading-mode futures   --timerange 20210101-20230101  -t  5m 15m --user-data-dir /freqtrade/user_data/

freqtrade download-data --exchange bybit   --trading-mode futures  --trading-mode futures   --timerange 20210101-20230101  -t  4h 8h --user-data-dir /freqtrade/user_data/

  
freqtrade download-data --exchange bybit   --trading-mode futures --data-format-ohlcv feather   --data-format-trades feather --trading-mode futures   --timerange 20210101-20230101  -t 15m --user-data-dir /freqtrade/user_data/


freqtrade download-data --exchange bybit  --pairs ETH/USDT ETH/USDT XRP/USDT BTC/USDT --trading-mode futures --data-format-ohlcv json   --data-format-trades json --trading-mode futures   --timerange 20210101-20220101  -t 1m 5m 15m --user-data-dir /freqtrade/user_data/


freqtrade download-data --exchange bybit --pairs ETH/USDT:USDT ETH/USDT XRP/USDT:USDT BTC/USDT:USDT  --timerange 20210101-20220101  -t 5m 15m --user-data-dir /freqtrade/user_data/

Whitelist PAIRS: Whitelist with 30 pairs: ['BTC/USDT:USDT', 'ETH/USDT:USDT', 'SOL/USDT:USDT', '1000PEPE/USDT:USDT', 'XRP/USDT:USDT', 'NOT/USDT:USDT', '1000BEER/USDT:USDT', 'DOGE/USDT:USDT', 'WIF/USDT:USDT', 'PEOPLE/USDT:USDT', 'ONDO/USDT:USDT', 'ORDI/USDT:USDT', 'FTM/USDT:USDT', 'TON/USDT:USDT', 'INJ/USDT:USDT', 'MEW/USDT:USDT', 'NEAR/USDT:USDT', 'LINK/USDT:USDT', 'ADA/USDT:USDT', 'MATIC/USDT:USDT', 'WLD/USDT:USDT', '1000BONK/USDT:USDT', 'ENA/USDT:USDT', 'GALA/USDT:USDT', 'LDO/USDT:USDT', 'AVAX/USDT:USDT', 'BNB/USDT:USDT', 'IO/USDT:USDT', 'SHIB1000/USDT:USDT', 'ARB/USDT:USDT']

# Back testing
https://www.freqtrade.io/en/stable/backtesting/

-c PATH, --config PATH
Specify configuration file (default: `userdir/config.json` or `config.json` whichever exists). Multiple --config options may be used. Can be set to `-` to read config from stdin.


For example, if the commission fee per order is 0.1% (i.e., 0.001 written as ratio), then you would run backtesting as the following:

## Freqai

CatBoost: https://catboost.ai/en/docs/
LightGBM: https://lightgbm.readthedocs.io/en/v3.3.2/#
XGBoost: https://xgboost.readthedocs.io/en/stable/#


freqtrade trade --config config_examples/config_freqai.example.json --strategy FreqaiExampleStrategy --freqaimodel LightGBMRegressor --strategy-path freqtrade/templates

freqtrade backtesting --strategy  LSv3_Full  --timeframe 5m   --config user_data/config-backtest-freqai.json --freqaimodel LightGBMRegressor --timerange 20230620-20240625
freqtrade backtesting --strategy-list  LSv3_Full  --timeframe 5m   --config user_data/config-backtest-freqai.json --freqaimodel LightGBMRegressor --timerange 20230620-20240625
freqtrade backtesting --strategy-list FreqaiExampleStrategy LSv1 SampleStrategy LSv3 LSv3_Full  --timeframe 5m   --config user_data/config-backtest-freqai.json --freqaimodel LightGBMRegressor --timerange 20230620-20240625


freqtrade backtesting --strategy FreqaiExampleStrategy  --timeframe 5m   --config user_data/config-backtest-freqai.json --freqaimodel LightGBMRegressor --timerange 20230620-20240625

freqtrade backtesting --strategy FreqaiExampleStrategy  --timeframe 5m   --config user_data/config-backtest-freqai.json --freqaimodel LightGBMRegressor --timerange 20210101-20230101


### Testing 

Be aware of this error:
2023-04-25 02:32:42,637 - freqtrade - ERROR - Trying to access pretrained model with `identifier` but found different features furnished by current strategy.Change `identifier` to train from scratch, or ensure thestrategy is furnishing the same features as the pretrainedmodel. In case of --strategy-list, please be aware that FreqAI requires all strategies to maintain identical feature_engineering_* functions

If it happen delete the model folder or give another id of the model.



## without time range
freqtrade backtesting --strategy-list LSv1 LSv3 --timerange=20210131-20240619  --fee 0.001  --timeframe 15m  --dry-run-wallet 2000 --user-data-dir /freqtrade/user_data/ --config /freqtrade/user_data/config-backtest.json



freqtrade backtesting -s LSv1   --fee 0.001  --timeframe 15m  --dry-run-wallet 2000 --user-data-dir /freqtrade/user_data/ --config /freqtrade/user_data/config-backtest.json


freqtrade backtesting -s LSv1 --timerange=20180131-20180301  --fee 0.001  --timeframe 15m  --dry-run-wallet 2000 --user-data-dir /freqtrade/user_data/ --config /freqtrade/user_data/config-backtest.json

You  can also backtest against other strategy
-s LSv1 SampleStrategy
--strategy-list LSv1 SampleStrategy

freqtrade backtesting --strategy-list LSv1 SampleStrategy --timerange=20210131-20220301  --fee 0.001  --timeframe 15m  --dry-run-wallet 2000 --user-data-dir /freqtrade/user_data/ --config /freqtrade/user_data/config-backtest.json



To visualize daily and weekly breakdowns, you can use the following:
freqtrade backtesting --strategy MyAwesomeStrategy --breakdown day week


# Hyperopt  
https://www.freqtrade.io/en/stable/hyperopt/


freqtrade hyperopt --hyperopt-loss SharpeHyperOptLossDaily --spaces roi stoploss trailing --strategy LSv1 --config config.json -e 100  --user-data-dir /freqtrade/user_data/ --config /freqtrade/user_data/config-backtest.json




# FreqAI
https://www.freqtrade.io/en/stable/freqai/


