# FreqTrade Documentation: Docker Quickstart

> Source: https://www.freqtrade.io/en/stable/docker_quickstart/

## Key Concepts

- Docker is the recommended deployment method, eliminating platform-specific installation complexity.
- The setup creates a `ft_userdata` directory containing all user configuration, strategies, logs, and databases.
- FreqUI (the web interface) is accessible on `localhost:8080` when enabled.

## Important Commands

```bash
mkdir ft_userdata && cd ft_userdata
curl https://raw.githubusercontent.com/freqtrade/freqtrade/stable/docker-compose.yml -o docker-compose.yml
docker compose pull
docker compose run --rm freqtrade create-userdir --userdir user_data
docker compose run --rm freqtrade new-config --config user_data/config.json
docker compose up -d
```

### Monitoring and Maintenance

- `docker compose ps` -- check running services
- `docker compose logs -f` -- stream logs in real time
- `docker compose pull && docker compose up -d` -- update to latest version
- Data download: `docker compose run --rm freqtrade download-data --pairs ETH/BTC --exchange binance --days 5 -t 1h`
- Backtesting: `docker compose run --rm freqtrade backtesting --config user_data/config.json --strategy SampleStrategy`

## Configuration Files

- `docker-compose.yml` -- primary orchestration file
- `user_data/config.json` -- main bot configuration
- `user_data/strategies/` -- custom strategy Python files
- `user_data/tradesv3.sqlite` -- trade history database
- `user_data/logs/freqtrade.log` -- application logs

## Practical Notes

- Always use `--rm` flag for non-trading commands (backtesting, data download) to clean up containers.
- Trading should use `docker compose up -d`, not `docker compose run`.
- For custom strategy dependencies, create a custom Dockerfile based on `docker/Dockerfile.custom`.
- Windows users may encounter timestamp sync issues (fix with `wsl --shutdown` and Docker restart). Linux VPS is recommended for production.
- Remote server deployments should use SSH tunnels or VPNs for FreqUI access since HTTPS is not natively supported.
