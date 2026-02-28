# FreqTrade Documentation: Data Analysis

> Source: https://www.freqtrade.io/en/stable/data-analysis/

## Overview

FreqTrade supports using Jupyter notebooks for analyzing backtesting results and trading history. Sample notebooks are located in `user_data/notebooks/` after running `create-userdir`.

## Setup

### Docker Jupyter Setup

```bash
docker compose -f docker/docker-compose-jupyter.yml up
# Accessible at https://127.0.0.1:8888/lab
```

### Virtual Environment Kernel Setup

```bash
pip install ipykernel
python -m ipykernel install --user --name=freqtrade
```

## Code Patterns

### Navigate to Project and Load Configuration

```python
# Navigate to project
import os
from pathlib import Path

project_root = "somedir/freqtrade"
os.chdir(project_root)

# Load multiple configs
from freqtrade.configuration import Configuration
config = Configuration.from_files(["config1.json", "config2.json"])
```

## Recommended Workflow

| Task | Recommended Tool |
|------|-----------------|
| Bot operations | CLI |
| Repetitive tasks | Shell scripts |
| Data analysis and visualization | Jupyter Notebooks |

## Practical Usage Notes

- **Copy example notebooks** before modifying to prevent overwriting during updates.
- Some tasks (especially **async operations**) do not work well in notebooks because Jupyter bypasses CLI arguments.
- Activate **conda/venv** before launching Jupyter, or use `nb_conda_kernels` to manage kernel environments.
- The Docker Jupyter setup automatically mounts the necessary directories for accessing FreqTrade data.
- Notebooks provide interactive exploration capabilities that are more flexible than CLI output for iterative analysis.
