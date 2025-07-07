# DMP Assistant Deploymnent Configuration

[![MIT License](https://img.shields.io/github/license/portagenetwork/roadmap-deploy)](https://github.com/portagenetwork/roadmap-deploy/blob/main/LICENSE)
![Dev App](https://img.shields.io/badge/dynamic/yaml?url=https://raw.githubusercontent.com/portagenetwork/roadmap-deploy/refs/heads/main/environments/dev/values.yaml&query=%24.app.image.tag&label=Dev%20App)
![Dev Assets](https://img.shields.io/badge/dynamic/yaml?url=https://raw.githubusercontent.com/portagenetwork/roadmap-deploy/refs/heads/main/environments/dev/values.yaml&query=$.assets.image.tag&label=Dev%20Assets)
![Staging App](https://img.shields.io/badge/dynamic/yaml?url=https://raw.githubusercontent.com/portagenetwork/roadmap-deploy/refs/heads/main/environments/staging/values.yaml&query=%24.app.image.tag&label=Staging%20App)
![Staging Assets](https://img.shields.io/badge/dynamic/yaml?url=https://raw.githubusercontent.com/portagenetwork/roadmap-deploy/refs/heads/main/environments/staging/values.yaml&query=$.assets.image.tag&label=Staging%20Assets)
![Prod App](https://img.shields.io/badge/dynamic/yaml?url=https://raw.githubusercontent.com/portagenetwork/roadmap-deploy/refs/heads/main/environments/prod/values.yaml&query=%24.app.image.tag&label=Prod%20App)
![Prod Assets](https://img.shields.io/badge/dynamic/yaml?url=https://raw.githubusercontent.com/portagenetwork/roadmap-deploy/refs/heads/main/environments/prod/values.yaml&query=$.assets.image.tag&label=Prod%20Assets)

GitOps deployment repository for the DMP Assistant application. This repository contains environment-specific Kubernetes
deployment configurations managed through ArgoCD.

## Overview

This repository separates deployment configuration from the main application source code (`roadmap` repo), following GitOps
best practices for:

- Environment-specific configuration management
- Automated deployments through ArgoCD
- Secure secrets management with sealed secrets
- Infrastructure as Code principles

## Repository Structure

```text
argocd/
├── dmp-roadmap-dev.yaml      # Development ArgoCD application definition
├── dmp-roadmap-staging.yaml  # Staging ArgoCD application definition
└── dmp-roadmap-prod.yaml     # Production ArgoCD application definition

environments/
├── base/
│   └── values.yaml           # Base Helm values shared across environments
├── dev/
│   ├── values.yaml          # Development environment overrides
│   └── sealed-secrets/      # Encrypted secrets for dev
├── staging/
│   ├── values.yaml          # Staging environment overrides
│   └── sealed-secrets/      # Encrypted secrets for staging
└── prod/
    ├── values.yaml          # Production environment overrides
    └── sealed-secrets/      # Encrypted secrets for production
```

## DMP Roadmap Configuration

### Application Architecture

The DMP Assistant (roadmap) application consists of:

- **Rails Application**: Main web application
- **PostgreSQL Database**: Data persistence
- **Background Jobs**: Sidekiq for async processing
- **File Storage**: CephFS for uploads and locales

### Environment Configuration

#### Base Configuration (`environments/base/values.yaml`)

Contains shared settings across all environments:

- **Domain Configuration**: Base domain and TLS settings
- **Application Settings**: Common Rails configuration
- **Resource Limits**: Default CPU and memory limits
- **Storage Configuration**: CephFS connection details
- **Cronjobs**: Scheduled tasks for translations and statistics

#### Environment-Specific Overrides

Each environment overrides base settings:

**Development** (`environments/dev/values.yaml`):

- **Domain**: `dev.dmp-pgd.ca`
- **Image Tags**: Release candidate versions (ex: `4.3.0-rc.1`)
- **Resources**: Lower limits for development
- **Certificates**: Let's Encrypt staging server

**Staging** (`environments/staging/values.yaml`):

- **Domain**: `staging.dmp-pgd.ca`
- **Image Tags**: Release candidate versions (ex: `4.3.0-rc.1`)
- **Resources**: Production-like limits
- **Certificates**: Let's Encrypt staging server

**Production** (`environments/prod/values.yaml`):

- **Domain**: `dmp-pgd.ca`
- **Image Tags**: Stable release versions (ex: `4.3.0`)
- **Resources**: Full production limits
- **Certificates**: Let's Encrypt production server

### Configuration Hierarchy

ArgoCD applies configuration in this order:

1. **Base values** from `environments/base/values.yaml`
2. **Environment-specific values** from `environments/{env}/values.yaml`
3. **Secrets** from sealed secrets in `environments/{env}/sealed-secrets/`

## Deployment Patterns

### GitOps Workflow

1. **Code Changes**: Developers push changes to `roadmap` repository
2. **Image Build**: CI/CD builds and tags container images
3. **Config Update**: Update image tags in this repository
4. **ArgoCD Sync**: ArgoCD detects changes and deploys automatically
5. **Verification**: Monitor deployment through ArgoCD UI

### Configuration Management

#### Updating Application Configuration

```bash
# Edit environment-specific settings
vim environments/dev/values.yaml

# Common settings to modify:
# - app.image.tag: Update application version
```

#### Managing Secrets

All sensitive configuration is stored as sealed secrets:

- **Database credentials**: Connection strings and passwords
- **Rails secrets**: SECRET_KEY_BASE and devise keys
- **Integration secrets**: ORCID, Google OAuth credentials
- **Storage secrets**: CephFS access credentials

See [Sealed Secrets Guide](docs/sealed-secrets-guide.md) for detailed instructions.

### Environment Promotion

Changes typically flow: `dev` → `staging` → `prod`

1. Test and validate in development environment
2. Promote to staging with environment-specific adjustments
3. After staging validation, promote to production
4. Monitor deployments at each stage

See [Contributing Guide](CONTRIBUTING.md) for detailed promotion procedures.

## ArgoCD Integration

Each environment has its own ArgoCD application (`dmp-roadmap-{environment}`) that:

- Monitors this repository for configuration changes
- References the Helm chart from the main `roadmap` repository
- Automatically syncs changes and deploys to the target cluster

Use the ArgoCD UI to monitor deployments, trigger manual syncs, and troubleshoot issues. See
[ArgoCD Guide](docs/argocd-guide.md) for detailed instructions.

## Getting Started

See the [Contributing Guide](CONTRIBUTING.md) for detailed instructions on making changes to deployment configurations.

## Documentation

- **[ArgoCD Guide](docs/argocd-guide.md)**: Complete guide to using ArgoCD UI
- **[Sealed Secrets Guide](docs/sealed-secrets-guide.md)**: Detailed sealed secrets management
- **[Contributing Guide](CONTRIBUTING.md)**: How to make changes to deployments

## Support

For deployment issues:

1. Check ArgoCD application status in the UI
2. Review application logs and events
3. Contact platform team for cluster-level issues
