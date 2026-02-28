# FreqTrade Documentation: Developer Guide

> Source: https://www.freqtrade.io/en/stable/developer/

## Developer Setup Options

### 1. DevContainer (Recommended for Quick Setup)

Use VSCode with the Remote Container extension for the most efficient setup.

### 2. Setup Script with Dev Dependencies

Run the setup script with development dependency options.

### 3. Manual Installation

```bash
pip3 install -r requirements-dev.txt
pip3 install -e .[all]
pre-commit install
```

Required tools: `pytest`, `ruff`, `mypy`, and `coveralls`. Pre-commit hooks run automatically; manual execution via `pre-commit run -a`.

## Testing

```bash
pytest  # Run all tests
```

### Log Verification Patterns

```python
from tests.conftest import log_has, log_has_re

def test_method_to_test(caplog):
    method_to_test()
    assert log_has("This event happened", caplog)
    # or regex: assert log_has_re(r"pattern.*here", caplog)
```

## Debugging

- **VSCode:** Launch configuration in `.vscode/launch.json` uses `debugpy`.
- **PyCharm:** Requires virtual environment configuration in Run/Debug Configurations.

## Exception Hierarchy

- `FreqtradeException` (base)
  - `OperationalException` / `ConfigurationError`
  - `DependencyException`
  - `PricingError` / `ExchangeError`
  - `StrategyError`

## Plugin Development

### Pairlist Handlers

Pairlist handlers must implement:

- `short_desc()` -- Telegram-friendly description
- `gen_pairlist()` -- initial pairlist generation
- `filter_pairlist()` -- chain filtering logic
- Registration in `constants.py` under `AVAILABLE_PAIRLISTS`

### Protections

Protections must implement the `IProtection` interface:

- `short_desc()`, `global_stop()`, `stop_per_pair()`
- Return `ProtectionReturn` objects (lock status, duration, reason, side)
- Use `calculate_lock_end()` helper for timing
- Configure via `has_local_stop=True` or `has_global_stop=True`

## Exchange Implementation Testing

- Public endpoint validation: `pytest --longrun tests/exchange_online/test_ccxt_compat.py`
- OHLCV data verification and candle limit adjustments
- Incomplete candle detection via `ohlcv_partial_candle` property
- Binance leverage tiers update requires authenticated futures account, generates `freqtrade/exchange/binance_leverage_tiers.json`

## Release Process

### Version Format

Version format: `YYYY.M` (e.g., `2025.7`) per PEP0440; minor versions like `2025.7.1`.

### Release Steps

1. Create branch from ~1 week old commit
2. Cherry-pick bugfixes
3. Merge stable
4. Update version in `freqtrade/__init__.py`
5. Create PR against stable
6. Update develop to next version with `-dev` suffix

### Changelog Generation

```bash
git log --oneline --no-decorate --no-merges stable..new_release
```

## PyPI Distribution

Automated via GitHub Actions. Manual fallback:

```bash
pip install -U build
python -m build --sdist --wheel
twine upload dist/*
```

## CI/CD

- Multi-OS testing (Linux, macOS, Windows)
- Docker multiarch builds for stable/develop
- Weekly full rebuilds
- Commit hash stored in `/freqtrade/freqtrade_commit`

## Practical Notes

- All contributions require documentation.
- Tests must pass on develop/stable.
- Virtual environment usage is essential.
- Documentation uses MkDocs with Material theme; test locally with `mkdocs serve`.
- Jupyter notebooks should be cleaned before committing:

```bash
jupyter nbconvert --ClearOutputPreprocessor.enabled=True --inplace <notebook>
```
