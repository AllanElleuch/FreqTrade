# FreqTrade Documentation: SQL Cheatsheet

> Source: https://www.freqtrade.io/en/stable/sql_cheatsheet/

## Key Concepts

A reference guide for querying and modifying FreqTrade's SQLite database. **Always back up the database** before running write queries and **never** run write queries while the bot is connected to the database.

## Installation

```bash
# Ubuntu/Debian
sudo apt-get install sqlite3

# Docker
docker compose exec freqtrade /bin/bash sqlite3 <database-file>.sqlite
```

## Navigation Commands

- `.open <filepath>` -- open database
- `.tables` -- list tables
- `.schema <table_name>` -- view table structure

## Read Queries

```sql
SELECT * FROM trades;
```

## Fix Manually Closed Trade

```sql
UPDATE trades SET is_open=0,
  close_date='2020-06-20 03:08:45.103418',
  close_rate=0.19638016,
  close_profit=0.0496,
  close_profit_abs = (amount * 0.19638016 * (1 - fee_close) -
                     (amount * (open_rate * (1 - fee_open)))),
  exit_reason='force_exit'
WHERE id=31;
```

## Delete a Trade

```sql
DELETE FROM trades WHERE id = 31;
```

## Enable Foreign Keys (Ubuntu)

```sql
PRAGMA foreign_keys = ON;
```

## Practical Notes

- The documentation **strongly recommends** using RPC methods instead of direct SQL. Use `/delete <tradeid>` via Telegram or REST API, or `/forceexit` for closing trades.
- **Never** omit the `WHERE` clause on `DELETE` or `UPDATE` -- it would affect all rows.
- Supports PostgreSQL and MariaDB as alternatives to SQLite with identical SQL syntax.
- Force-exit orders close automatically on the bot's next iteration without manual DB updates.
