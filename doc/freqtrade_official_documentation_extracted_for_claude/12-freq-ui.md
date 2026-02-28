# FreqTrade Documentation: FreqUI

> Source: https://www.freqtrade.io/en/stable/freq-ui/

## Key Concepts Covered

- Installation, configuration via REST API settings, core features, and webserver mode.

## Installation

- Auto-installed during standard FreqTrade setup.
- Manual install/update: `freqtrade install-ui`
- Accessible at `http://127.0.0.1:8080` once the bot starts in trade or dry-run mode.

## Configuration

FreqUI piggybacks on the REST API configuration:

```json
{
  "api_server": {
    "enabled": true,
    "listen_ip_address": "127.0.0.1",
    "listen_port": 8080,
    "username": "user",
    "password": "pass",
    "jwt_secret_key": "random_secret",
    "CORS_origins": ["http://localhost:8080"]
  }
}
```

Note: No trailing slash in `CORS_origins` URLs.

## Core Features

- **Trade View** -- Visualize active trades, start/stop bot, force entries/exits.
- **Plot Configurator** -- Strategy-based or UI-configured chart customization.
- **Settings** -- Timezone, favicon trade visualization, candle colors, in-app notifications.
- **Themes** -- Light and dark mode.

## Webserver Mode

`freqtrade webserver`

Enables data downloading, pairlist testing, strategy backtesting with visualization, and loading/comparing previous backtest results -- all from the UI without a running trading bot.

## Practical Notes

- FreqUI is optional; FreqTrade operates fully without it.
- CORS configuration is critical for cross-origin API access.
- Developers should clone the source repository rather than using `install-ui`.
