# Reusable Workflows Guide

This guide covers how to use the reusable workflows in this library.

## Overview

Reusable workflows allow you to define a workflow once and call it from other workflows. This promotes DRY (Don't Repeat Yourself) principles and ensures consistency across your organization.

## Available Workflows

### ci-docker.yml

A complete Docker CI/CD pipeline with multi-architecture support.

**Features:**
- Multi-arch builds (amd64/arm64) using QEMU
- Layer caching with GitHub Actions cache
- Vulnerability scanning with Trivy
- SBOM generation
- Semantic versioning

**Usage:**

```yaml
name: Docker CI
on:
  push:
    branches: [main]

jobs:
  build:
    uses: gregorygreenspan/github-actions-library/.github/workflows/ci-docker.yml@main
    with:
      image-name: my-application
      platforms: linux/amd64,linux/arm64
      push: ${{ github.event_name != 'pull_request' }}
    secrets:
      registry-username: ${{ secrets.DOCKER_USERNAME }}
      registry-password: ${{ secrets.DOCKER_PASSWORD }}
```

**Inputs:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `image-name` | string | ✅ | - | Name of the Docker image |
| `registry` | string | | `ghcr.io` | Container registry URL |
| `context` | string | | `.` | Docker build context path |
| `platforms` | string | | `linux/amd64,linux/arm64` | Target platforms |
| `push` | boolean | | `false` | Push image to registry |
| `scan-severity` | string | | `HIGH,CRITICAL` | Minimum severity to fail |

---

### ci-terraform.yml

Production-ready Terraform workflow with plan/apply support.

**Features:**
- Multi-environment support
- Plan output in PR comments
- Cost estimation with Infracost
- Security scanning with Checkov
- AWS OIDC authentication

**Usage:**

```yaml
name: Infrastructure
on:
  pull_request:
    paths: ['infrastructure/**']
  push:
    branches: [main]
    paths: ['infrastructure/**']

jobs:
  terraform:
    uses: gregorygreenspan/github-actions-library/.github/workflows/ci-terraform.yml@main
    with:
      working-directory: infrastructure
      environment: production
      apply: ${{ github.event_name == 'push' }}
    secrets:
      aws-role-arn: ${{ secrets.AWS_ROLE_ARN }}
      infracost-api-key: ${{ secrets.INFRACOST_API_KEY }}
```

---

### security-audit.yml

Comprehensive security scanning for applications.

**Features:**
- SAST with Semgrep
- Secret detection with Gitleaks
- Dependency scanning with Trivy and OSV
- Container scanning
- IaC security with Checkov

**Usage:**

```yaml
name: Security
on:
  schedule:
    - cron: '0 6 * * *'  # Daily at 6 AM
  push:
    branches: [main]

jobs:
  security:
    uses: gregorygreenspan/github-actions-library/.github/workflows/security-audit.yml@main
    with:
      scan-type: full
      fail-on-severity: high
```

## Best Practices

1. **Pin to a specific SHA** - Instead of `@main`, use a specific commit SHA for reproducibility
2. **Use GitHub Environments** - For apply operations, configure environment protection rules
3. **Leverage Secrets** - Pass sensitive values through the `secrets` block
4. **Check Outputs** - Reusable workflows expose outputs for conditional logic

## Migrating Existing Workflows

See our [Migration Guide](migration.md) for step-by-step instructions.
