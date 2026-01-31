# GitHub Actions Library

![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue)

Reusable GitHub Actions workflows and composite actions for CI/CD pipelines.

## Workflows

```
.github/workflows/
├── docker-build.yml      # Build, scan, and push Docker images
├── terraform-plan.yml    # Terraform plan with cost estimation
├── k8s-deploy.yml        # Kubernetes deployment with ArgoCD
├── security-scan.yml     # SAST, DAST, dependency scanning
└── release.yml           # Semantic release automation
```

## Composite Actions

```
actions/
├── docker-build/         # Multi-arch Docker build
├── terraform-plan/       # Terraform plan with PR comments
├── k8s-deploy/           # Kubernetes deployment
└── security-scan/        # Trivy, Grype, CodeQL
```

## Usage

```yaml
jobs:
  build:
    uses: ghndrx/github-actions-library/.github/workflows/docker-build.yml@main
    with:
      image-name: myapp
    secrets: inherit
```

## Features

- ✅ Reusable workflows (DRY)
- ✅ Matrix builds
- ✅ Security scanning built-in
- ✅ Caching optimization
- ✅ OIDC authentication (no long-lived secrets)

## License

MIT
