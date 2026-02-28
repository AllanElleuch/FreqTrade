# FreqTrade Documentation: Installation

> Source: https://www.freqtrade.io/en/stable/installation/

## Key Concepts

- Four installation methods: Docker (recommended), Script (`./setup.sh`), Manual (virtualenv + pip), and Conda.
- Requires Python 3.11 or higher plus development headers.
- System clock must be NTP-synchronized for exchange communication.

## Important Commands

```bash
# Script install (Linux/macOS)
./setup.sh -i       # Install from scratch
./setup.sh -u       # Update existing installation
source .venv/bin/activate  # Activate virtual environment (required each new terminal)

# Manual install
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
python3 -m pip install -e .

# First-time setup
freqtrade create-userdir --userdir user_data
freqtrade new-config --config user_data/config.json
freqtrade trade --config user_data/config.json --strategy SampleStrategy
```

## Platform-Specific Notes

- **Windows:** Docker strongly recommended; alternatively use WSL or PowerShell (`./setup.ps1`).
- **ARM64 (Apple M1, Oracle VM):** Docker recommended; native installation unsupported.
- **Raspberry Pi:** Installation possible but slow; Docker preferred.
- MacOS compilation failures may require SDK headers via Command Line Tools.
- Windows C++ dependency errors require Visual C++ Build Tools.

## Practical Notes

- The `develop` branch has the latest features with automated testing; `stable` has monthly release snapshots.
- Always start with `dry_run: True` before live trading.
- Server deployments should use Docker or terminal multiplexers (screen/tmux).
- The "command not found" error indicates the virtual environment is not activated.
