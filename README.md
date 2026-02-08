# GitHub Actions Library

![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue)

Reusable GitHub Actions workflows and composite actions for CI/CD pipelines.

## Workflows

| Workflow | Description |
|----------|-------------|
| [`python-ci.yml`](.github/workflows/python-ci.yml) | Python CI with UV (lint, type-check, test, security) |
| [`docker-ci.yml`](.github/workflows/docker-ci.yml) | Docker CI/CD with OIDC, attestations, and security scanning |

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

## Docker CI Workflow Features

The `docker-ci.yml` reusable workflow provides production-ready container builds:

- **OIDC authentication** - Keyless auth to GHCR (no secrets needed)
- **Multi-platform builds** - linux/amd64 + linux/arm64 by default
- **SBOM generation** - Software Bill of Materials attestation
- **Build provenance** - Cryptographic proof of build origin
- **Trivy scanning** - Vulnerability detection with SARIF upload
- **Smart caching** - GitHub Actions cache for layer reuse
- **Semantic tagging** - Auto-tags from git refs and versions

### Quick Start

```yaml
# .github/workflows/docker.yml
name: Docker

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:

jobs:
  build:
    uses: ghndrx/github-actions-library/.github/workflows/docker-ci.yml@main
    with:
      image-name: my-app
      push: ${{ github.event_name != 'pull_request' }}
    permissions:
      contents: read
      packages: write
      id-token: write
      attestations: write
```

### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `image-name` | string | *required* | Image name (without registry) |
| `context` | string | `.` | Docker build context path |
| `dockerfile` | string | `Dockerfile` | Dockerfile path relative to context |
| `push` | boolean | `false` | Push image to GHCR |
| `platforms` | string | `linux/amd64,linux/arm64` | Target platforms |
| `build-args` | string | `''` | Build args (newline-separated) |
| `target` | string | `''` | Multi-stage build target |
| `scan-severity` | string | `CRITICAL,HIGH` | Trivy severity threshold |
| `fail-on-vuln` | boolean | `false` | Fail on vulnerabilities |
| `generate-sbom` | boolean | `true` | Generate SBOM attestation |
| `generate-provenance` | boolean | `true` | Generate provenance attestation |

### Outputs

| Output | Description |
|--------|-------------|
| `image-digest` | Image digest (sha256:...) |
| `image-tags` | Generated tags (JSON array) |
| `sbom-attestation-id` | SBOM attestation bundle ID |

### Security Features

All actions are **pinned to SHA** for supply chain security:

```yaml
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

Images pushed to GHCR include:
- **SBOM attestation** - Full dependency manifest
- **Build provenance** - Verifiable build metadata
- **Vulnerability scan results** - Uploaded as SARIF to Security tab

## License

MIT
