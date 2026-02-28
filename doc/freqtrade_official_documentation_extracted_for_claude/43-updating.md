# FreqTrade Documentation: Updating

> Source: https://www.freqtrade.io/en/stable/updating/

## Overview

Regular updates are essential because exchange APIs change frequently. Breaking changes are documented in release changelogs. Users on the development branch should monitor pull requests for upcoming changes.

## Update Commands by Installation Method

### Docker

```bash
docker compose pull
docker compose up -d
```

Note: Legacy `master` images should be migrated to `stable`.

### Setup Script

```bash
./setup.sh --update
```

Important: Deactivate the virtual environment before running this command.

### Native/pip Installation

```bash
git pull
pip install -U -r requirements.txt
pip install -e .
freqtrade install-ui
```

This updates dependencies and ensures FreqUI is at the latest version.

## Practical Usage Notes

- Common update issues stem from missing or failed dependency installations.
- Platform-specific problems (especially Windows) are covered in the general installation troubleshooting sections.
- Always check the changelog for breaking changes before updating.
