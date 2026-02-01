# GitHub Actions Library

![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue)

Reusable GitHub Actions workflows and composite actions for CI/CD pipelines.

## Workflows

| Workflow | Description |
|----------|-------------|
| [`python-ci.yml`](.github/workflows/python-ci.yml) | Python CI with UV (lint, type-check, test, security) |

## Composite Actions

| Action | Description |
|--------|-------------|
| [`setup-python-uv`](actions/setup-python-uv) | Fast Python setup with UV package manager |

## Quick Start

### Python CI

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  ci:
    uses: ghndrx/github-actions-library/.github/workflows/python-ci.yml@main
    with:
      python-versions: '["3.11", "3.12", "3.13"]'
      run-typecheck: true
      coverage-threshold: 80
```

### Setup Python with UV (Composite Action)

```yaml
steps:
  - uses: actions/checkout@v4
  
  - uses: ghndrx/github-actions-library/actions/setup-python-uv@main
    with:
      python-version: '3.12'
      extras: 'dev,test'
  
  - run: uv run pytest
```

## Python CI Workflow Features

The `python-ci.yml` reusable workflow provides:

- **Ruff linting** - Fast Python linter with auto-fix suggestions
- **Pyright type checking** - Strict type validation
- **Matrix testing** - Test across multiple Python versions
- **Coverage enforcement** - Fail if coverage drops below threshold
- **Bandit security scanning** - Detect security vulnerabilities
- **UV caching** - 10-100x faster than pip installs

### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `python-versions` | string | `'["3.12"]'` | JSON array of Python versions |
| `working-directory` | string | `.` | Project directory |
| `run-lint` | boolean | `true` | Run Ruff linter |
| `run-typecheck` | boolean | `true` | Run Pyright |
| `run-tests` | boolean | `true` | Run pytest |
| `run-security` | boolean | `true` | Run Bandit scanner |
| `test-command` | string | `pytest --cov --cov-report=xml` | Custom test command |
| `coverage-threshold` | number | `0` | Min coverage % (0 to disable) |
| `extras` | string | `''` | Extra dependency groups |

## Requirements

Projects using the Python CI workflow should have:

- `pyproject.toml` with UV-compatible configuration
- Dev dependencies: `ruff`, `pyright`, `pytest`, `pytest-cov`, `bandit`

Example `pyproject.toml`:

```toml
[project]
name = "myproject"
requires-python = ">=3.11"

[tool.uv]
dev-dependencies = [
    "ruff>=0.8",
    "pyright>=1.1",
    "pytest>=8.0",
    "pytest-cov>=6.0",
    "bandit>=1.8",
]

[tool.ruff]
line-length = 100
target-version = "py311"

[tool.pyright]
pythonVersion = "3.12"
typeCheckingMode = "standard"
```

## License

MIT
