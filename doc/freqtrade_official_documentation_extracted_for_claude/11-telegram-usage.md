# FreqTrade Documentation: Telegram Usage

> Source: https://www.freqtrade.io/en/stable/telegram-usage/

## Key Concepts Covered

- Bot creation via BotFather, authentication, command reference, notification control, custom keyboard configuration, and message formats.

## Setup Requirements

- Create bot with `@BotFather` (`/newbot`), obtain API token.
- Get your `chat_id` from `@userinfobot`.
- For groups, use `/tg_info` to retrieve group and topic IDs.

## Configuration

```json
{
  "telegram": {
    "enabled": true,
    "token": "YOUR_TOKEN",
    "chat_id": "YOUR_CHAT_ID"
  }
}
```

## Notification Granularity

Three levels per notification type: `"on"` (with sound), `"silent"` (no sound), `"off"` (suppressed). Configurable categories: status, warning, startup, entry, entry_fill, exit (with sub-reasons), balance_dust_level, and strategy messages.

## Key Command Categories

| Category | Commands |
|----------|----------|
| **Bot Control** | `/start`, `/stop`, `/pause` (`/stopentry`), `/reload_config`, `/show_config` |
| **Trade Status** | `/status`, `/status table`, `/count`, `/trades [limit]`, `/order <id>`, `/locks`, `/unlock` |
| **Trade Actions** | `/forceexit <id>`, `/forceexit all`, `/forcelong <pair>`, `/forceshort <pair>`, `/delete <id>`, `/cancel_open_order <id>` |
| **Performance** | `/profit [n]`, `/profit_long`, `/profit_short`, `/performance`, `/balance`, `/daily`, `/weekly`, `/monthly`, `/stats`, `/entries`, `/exits` |
| **Pair Management** | `/whitelist`, `/blacklist [pair]`, `/marketdir` |
| **Utility** | `/logs [limit]`, `/help`, `/version`, `/list_custom_data`, `/tg_info` |

## Custom Keyboard

```json
"keyboard": [
    ["/daily", "/stats", "/balance", "/profit"],
    ["/status table", "/performance"],
    ["/reload_config", "/count", "/logs"]
]
```

## Practical Notes

- Group usage gives every group member access to all bot commands -- security warning.
- The `/pause` signal is not persisted across bot restarts.
- `/marketdir` resets after restart and is not backtesting-compatible.
- `entry_fill` and `exit_fill` notifications default to `"off"` and must be explicitly enabled.
