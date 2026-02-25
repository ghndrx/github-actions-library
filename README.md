# GitHub Actions Library

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?logo=github-actions&logoColor=white)](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

A curated collection of production-ready, reusable GitHub Actions workflows and composite actions for modern DevOps pipelines.

## 🎯 Overview

This repository provides battle-tested, security-hardened GitHub Actions components designed for:

- **CI/CD Pipelines** - Build, test, and deploy with confidence
- **Security Scanning** - Shift-left security with automated vulnerability detection
- **Infrastructure as Code** - Terraform workflows with proper state management
- **Container Workflows** - Multi-arch Docker builds with attestations
- **Python Development** - Fast CI with UV package manager
- **Go Development** - Linting, security scanning, and testing for Go projects

## 📁 Repository Structure

```
├── .github/workflows/           # Callable workflows in standard location
│   ├── python-ci.yml           # Python CI with UV (lint, type-check, test)
│   ├── go-ci.yml               # Go CI with golangci-lint, govulncheck, testing
│   └── docker-ci.yml           # Docker CI/CD with OIDC and attestations
├── reusable-workflows/          # Additional callable workflows
│   ├── ci-docker.yml           # Docker build/push with security scanning
│   ├── ci-terraform.yml        # Terraform plan/apply with drift detection
│   └── security-audit.yml      # Comprehensive security scanning
├── composite-actions/           # Reusable action components
│   ├── docker-build/           # Multi-arch Docker build action
│   ├── security-scan/          # Aggregated security scanning
│   └── terraform-deploy/       # Terraform with remote state
├── actions/                     # Additional composite actions
│   └── setup-python-uv/        # Fast Python setup with UV
├── workflow-templates/          # Starter templates for new projects
└── docs/                        # Detailed documentation
```

## 🚀 Quick Start

### Python CI with UV

```yaml
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

### Go CI

```yaml
name: CI
on: [push, pull_request]

jobs:
  ci:
    uses: ghndrx/github-actions-library/.github/workflows/go-ci.yml@main
    with:
      go-versions: '["1.22", "1.23"]'
      coverage-threshold: 70
      race-detection: true
    permissions:
      contents: read
      security-events: write
```

### Docker CI/CD

```yaml
name: Docker
on:
  push:
    branches: [main]

jobs:
  docker:
    uses: ghndrx/github-actions-library/.github/workflows/docker-ci.yml@main
    with:
      image-name: my-app
      push: true
    secrets: inherit
```

### Terraform Infrastructure

```yaml
name: Infrastructure
on:
  pull_request:
    paths: ['infrastructure/**']

jobs:
  terraform:
    uses: ghndrx/github-actions-library/reusable-workflows/ci-terraform.yml@main
    with:
      working-directory: infrastructure
      environment: production
      apply: false
    secrets:
      aws-role-arn: ${{ secrets.AWS_ROLE_ARN }}
```

### Security Audit

```yaml
name: Security
on:
  schedule:
    - cron: '0 6 * * *'

jobs:
  security:
    uses: ghndrx/github-actions-library/reusable-workflows/security-audit.yml@main
    with:
      scan-type: full
      fail-on-severity: high
```

## 🔒 Security Features

All workflows implement security best practices:

- **Pinned Actions** - SHA-pinned dependencies to prevent supply chain attacks
- **Minimal Permissions** - Least-privilege GITHUB_TOKEN permissions
- **Secret Scanning** - Pre-commit hooks and CI checks for leaked secrets
- **SBOM Generation** - Software Bill of Materials for container images
- **Signed Artifacts** - Attestations for provenance and SBOM

## 📚 Workflows Reference

### Reusable Workflows

| Workflow | Description | Key Features |
|----------|-------------|--------------|
| `python-ci.yml` | Python CI pipeline | UV, Ruff, mypy, pytest, coverage |
| `go-ci.yml` | Go CI pipeline | golangci-lint, govulncheck, gosec, coverage |
| `docker-ci.yml` | Docker build & push | Multi-arch, OIDC, attestations |
| `ci-docker.yml` | Enhanced Docker CI | Trivy scanning, SBOM, caching |
| `ci-terraform.yml` | Terraform pipeline | Plan, apply, cost estimation |
| `security-audit.yml` | Security scanning | SAST, secrets, deps, IaC |

### Composite Actions

| Action | Description |
|--------|-------------|
| `setup-python-uv` | Fast Python + UV setup |
| `docker-build` | Streamlined Docker builds |
| `security-scan` | Unified security scanning |
| `terraform-deploy` | Terraform operations |

## 📖 Documentation

- [Reusable Workflows Guide](docs/reusable-workflows.md)
- [Composite Actions Reference](docs/composite-actions.md)
- [Security Best Practices](docs/security.md)

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) for details.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Built with ❤️ for the DevOps community**
