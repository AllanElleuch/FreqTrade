# Download DATA
https://www.freqtrade.io/en/stable/data-download/


freqtrade list-data --userdir ~/.freqtrade/user_data/

freqtrade download-data --exchange bybit  --pairs ETH/USDT ETH/USDT XRP/USDT BTC/USDT --trading-mode futures --data-format-ohlcv json   --data-format-trades json --trading-mode futures   --timerange 20210101-20220101  -t 15m --user-data-dir /freqtrade/user_data/


freqtrade download-data --exchange bybit --pairs ETH/USDT:USDT ETH/USDT XRP/USDT:USDT BTC/USDT:USDT  --timerange 20210101-20220101  -t 5m 15m --user-data-dir /freqtrade/user_data/


# Back testing
https://www.freqtrade.io/en/stable/backtesting/

-c PATH, --config PATH
Specify configuration file (default: `userdir/config.json` or `config.json` whichever exists). Multiple --config options may be used. Can be set to `-` to read config from stdin.


freqtrade backtesting -s LSv1 --timeframe 15m  --user-data-dir /freqtrade/user_data/ --config /freqtrade/user_data/config-backtest.json
 
