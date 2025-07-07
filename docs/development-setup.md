# Development Setup

## Prerequisites

- Git
- Python 3.8+
- [pre-commit](https://pre-commit.com/)
- [yq](https://github.com/mikefarah/yq) (for validation scripts)

## Getting Started

### 1. Install Pre-commit

```bash
# Using pip
pip install pre-commit

# Using homebrew (macOS)
brew install pre-commit

# Using conda
conda install -c conda-forge pre-commit
```

### 2. Install Pre-commit Hooks

```bash
# Clone the repository
git clone https://github.com/portagenetwork/roadmap-deploy.git
cd roadmap-deploy

# Install pre-commit hooks
pre-commit install
```

### 3. Run Pre-commit

```bash
# Run against all files (first time)
pre-commit run --all-files

# Pre-commit will now run automatically on git commit
git add .
git commit -m "your commit message"
```

## Available Tools

### Validation

- **YAML Linting**: Validates YAML syntax and formatting
- **Markdown Linting**: Ensures consistent markdown formatting
- **Helm Validation**: Validates Helm templates and values
- **ArgoCD Validation**: Validates ArgoCD application configurations

### Security

- **Secret Detection**: Scans for accidentally committed secrets
- **Security Scanning**: Vulnerability scanning with Trivy

### Code Quality

- **Trailing Whitespace**: Removes trailing whitespace
- **End of File**: Ensures files end with newline
- **Mixed Line Endings**: Standardizes line endings

## Manual Validation

### Validate Helm Templates

```bash
# Run validation script
./scripts/validate-helm.sh

# Or validate specific environment
helm template dmp-roadmap-dev ../roadmap/charts/dmp-roadmap \
  --values environments/base/values.yaml \
  --values environments/dev/values.yaml \
  --dry-run
```

### Validate ArgoCD Applications

```bash
# Run validation script
./scripts/validate-argocd.sh

# Or validate with yq
yq eval '.spec.destination.namespace' argocd/dmp-roadmap-dev.yaml
```

### Check for Secrets

```bash
# Run secret detection
detect-secrets scan --baseline .secrets.baseline

# Update baseline if needed
detect-secrets scan --update .secrets.baseline
```

## IDE Integration

### VS Code

Install the following extensions for better development experience:

- **YAML**: Syntax highlighting and validation
- **Markdownlint**: Markdown linting
- **GitLens**: Enhanced Git capabilities
- **EditorConfig**: Automatic formatting

### IntelliJ/PyCharm

- Enable EditorConfig support
- Install YAML plugin
- Configure Git integration

## Troubleshooting

### Pre-commit Hook Failures

```bash
# Skip hooks temporarily (not recommended)
git commit --no-verify

# Fix issues and retry
pre-commit run --all-files

# Update hooks to latest versions
pre-commit autoupdate
```

### YAML Validation Errors

```bash
# Check YAML syntax
yamllint environments/dev/values.yaml

# Validate specific file
yq eval . environments/dev/values.yaml
```

### Permission Errors

```bash
# Make scripts executable
chmod +x scripts/*.sh
```

## Best Practices

1. **Run pre-commit before pushing**: `pre-commit run --all-files`
2. **Keep commits small**: Easier to validate and review
3. **Use descriptive commit messages**: Follow conventional commits
4. **Test changes locally**: Validate before creating PR
5. **Update documentation**: Keep docs in sync with changes
