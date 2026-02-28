# FreqTrade Documentation: Webhook Config

> Source: https://www.freqtrade.io/en/stable/webhook-config/

## Key Concepts Covered

- Webhook setup, payload formats, message types and their parameters, retry logic, Discord integration, and custom messages.

## Basic Configuration

```json
{
  "webhook": {
    "enabled": true,
    "url": "https://your-webhook-url",
    "format": "json",
    "retries": 3,
    "retry_delay": 0.2,
    "timeout": 10
  }
}
```

## Payload Formats

- `"form"` (default) -- Form-encoded POST body
- `"json"` -- JSON-encoded; allows nested structures (ideal for Mattermost, Slack)
- `"raw"` -- Raw data; requires a single field named `"data"`

## Message Types and Available Parameters

| Event | Key Parameters |
|-------|---------------|
| **entry / entry_fill** | `{trade_id}`, `{pair}`, `{exchange}`, `{open_rate}`, `{amount}`, `{stake_amount}`, `{direction}`, `{leverage}`, `{enter_tag}` |
| **entry_cancel** | Same as entry events |
| **exit / exit_fill** | All entry params plus `{close_rate}`, `{profit_amount}`, `{profit_ratio}`, `{exit_reason}` |
| **exit_cancel** | Exit params with profit tracking |
| **status** | `{status}` only |

## Discord Integration

A dedicated `"discord"` config section with a Discord webhook URL. Default templates provided for `entry_fill` and `exit_fill` events; customizable or disableable with empty arrays.

## Advanced Options

- `"allow_custom_messages": true` enables strategy-level webhook calls via `self.dp.send_msg()`.
- Template strings use Python format syntax: `{pair}`, `{profit_ratio:.2%}`.

## Practical Notes

- Retry logic is important for unreliable webhook endpoints (set `retries` > 0).
- JSON format is recommended for services like Mattermost and Slack that expect structured payloads.
- Discord has its own dedicated configuration path separate from generic webhooks.
