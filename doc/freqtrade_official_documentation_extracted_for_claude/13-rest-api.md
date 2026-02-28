# FreqTrade Documentation: REST API

> Source: https://www.freqtrade.io/en/stable/rest-api/

## Key Concepts Covered

- API server configuration, authentication (Basic, JWT, WebSocket), comprehensive endpoint reference, client library, WebSocket streaming, CORS, and security.

## Configuration

```json
{
  "api_server": {
    "enabled": true,
    "listen_ip_address": "127.0.0.1",
    "listen_port": 8080,
    "username": "user",
    "password": "pass",
    "jwt_secret_key": "random_32char_string",
    "ws_token": "websocket_token",
    "verbosity": "error",
    "enable_openapi": false,
    "CORS_origins": ["http://localhost:8080"]
  }
}
```

## Authentication

- **Basic Auth** -- Username/password for most endpoints.
- **JWT Tokens** -- Obtained via `POST /token/login`; access tokens expire in 15 minutes; refresh tokens extend sessions. Passed as `Authorization: Bearer {token}`.
- **WebSocket** -- Uses `ws_token` as query parameter.

## Key Endpoint Groups

| Group | Endpoints |
|-------|-----------|
| **Bot Control** | `POST /start`, `/pause`, `/stop`, `/stopbuy`, `/reload_config`; `GET /ping` (no auth) |
| **Trades** | `GET /trades`, `/trade/<id>`; `DELETE /trades/<id>`; `POST /forceenter`, `/forceexit` |
| **Analytics** | `GET /performance`, `/profit`, `/stats`, `/daily`, `/weekly`, `/monthly`, `/entries`, `/exits`, `/mix_tags` |
| **Account** | `GET /balance`, `/whitelist`, `/blacklist`; `POST /blacklist`; `DELETE /blacklist` |
| **Data (Alpha)** | `GET /pair_candles`, `/pair_history`; `POST` variants with column filtering |
| **System** | `GET /status`, `/count`, `/logs`, `/version`, `/sysinfo`, `/health` |
| **Locks** | `GET /locks`; `POST /locks`; `DELETE /locks/<id>` |

## Client Library

```bash
pip install freqtrade-client
freqtrade-client --config rest_config.json <command>
```

Programmatic usage:

```python
from freqtrade_client import FtRestClient
client = FtRestClient(server_url, username, password)
```

## WebSocket Streaming

```
ws://localhost:8080/api/v1/message/ws?token={ws_token}
```

Subscribe with:

```json
{"type": "subscribe", "data": ["whitelist", "analyzed_df"]}
```

## Practical Notes

- Never expose the API directly to the internet; use SSH tunnels, VPNs, or reverse proxies (Nginx, Traefik, Caddy).
- Docker deployments should set `listen_ip_address` to `"0.0.0.0"` with proper port mapping.
- `enable_openapi: true` provides Swagger UI at `/docs`.
- Pair candle/history endpoints are marked alpha and may change.
- Generate secrets with `python -c "import secrets; print(secrets.token_hex())"`.
