# FreqTrade Documentation: Exchanges

> Source: https://www.freqtrade.io/en/stable/exchanges/

## Overview

Exchange integration in FreqTrade relies on **CCXT** (100+ exchanges supported). The documentation covers tested vs. community-supported exchanges, stoploss-on-exchange features, rate limiting, and exchange-specific configuration quirks.

## Binance

- **Geographic API blocks:** Canada, Malaysia, Netherlands, United States.
- **Blacklist recommendation:** Add `"BNB/<STAKE>"` to your blacklist to avoid fee complications.
- **RSA key support:** Via environment variables or config.
- **Futures requirements:**
  - Must use orderbook for pricing.
  - Account must be set to **"One-way Mode"**.
  - Account must be set to **"Single-Asset Mode"**.
- **BNFCR futures:** Special configuration:
  ```json
  {
    "trading_mode": "futures",
    "margin_mode": "cross",
    "proxy_coin": "BNFCR"
  }
  ```

## Kraken

- Only **720 historic candles** available via API; use `--dl-trades` for backtesting.
- Supports quarterly trade zip downloads with `convert-trade-data` and `trades-to-ohlcv` commands.
- **Rate limit parameter** = milliseconds delay between requests (not requests/second).

## Kucoin

- Requires **API passphrase** beyond key/secret.
- **Blacklist recommendation:** Add `"KCS/<STAKE>"` to your blacklist.

## OKX

- Requires **passphrase** for API authentication.
- Use `"myokx"` for EAA-registered accounts.
- Only **100 candles per API call**.
- Supports both **"Buy/Sell"** and **"long/short" (hedge)** position modes.

## Hyperliquid (DEX)

- Uses **wallet private keys** instead of API keys.
- Requires:
  - `walletAddress`: Main wallet address
  - `privateKey`: API wallet private key
- **USDC collateral only**, deposits via Arbitrum One Layer 2.
- No native market orders (CCXT simulates with 5% max slippage).
- Only **5000 candles** available.
- Vault/subaccount support via `ccxt_config.options`.
- **Security warning:** Never use actual wallet private keys; always create separate API wallets.

## Other Exchanges

| Exchange | Key Notes |
|----------|-----------|
| **Bingx** | Supports stop-loss-limit and stop-market orders |
| **Bybit** | Futures-only stoploss, one subaccount per bot |
| **Bitget** | Passphrase required, isolated futures |
| **Gate.io** | `unknown_fee_rate` config needed for POINT token fees |
| **Bitmart** | UID/memo required, Verification Level 2 mandatory |

## Time-in-Force Support by Exchange

| TIF | Exchanges |
|-----|-----------|
| **GTC** (Good Till Cancelled) | All exchanges |
| **IOC** (Immediate Or Cancel) | Binance, Bingx, Kraken, Gate.io, Bybit, Bitget |
| **FOK** (Fill Or Kill) | Kucoin, Bybit, Bitget |
| **PO** (Post Only) | Bingx, Bybit, Bitget |

## General Practical Notes

- Most exchanges return **incomplete current candles**; FreqTrade removes them by default to prevent repainting.
- Regenerating API keys is better than resetting Nonce for `InvalidNonce` errors.
- Custom `_ft_has_params` settings may **break your bot** and are unsupported.
- Always check the exchange-specific documentation for required account settings before running a bot.
- Some exchanges have minimum order size requirements that may vary by pair.
