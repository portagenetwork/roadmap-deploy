# Contributing to roadmap-deploy

This document outlines the process for making changes to the DMP Assistant deployment configurations.

## Overview

This repository follows GitOps principles where all deployment changes are made through Git commits and automatically
applied by ArgoCD. No direct `kubectl` commands should be used for configuration changes.

## Making Changes

### 1. Environment-Specific Configuration

To modify environment-specific settings:

```bash
# Edit the appropriate environment values file
vim environments/{environment}/values.yaml
```

Changes are automatically applied when committed to the `main` branch.

### 2. Base Configuration

To modify shared configuration across all environments:

```bash
# Edit base values
vim environments/base/values.yaml
```

**Note**: Base configuration changes affect all environments. Test thoroughly in development first.

### 3. Secrets Management

For sensitive configuration:

```bash
# Create a new sealed secret
kubectl create secret generic secret-name \
  --from-literal=key=value \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > environments/{environment}/sealed-secrets/secret-name.yaml
```

### 4. ArgoCD Application Changes

To modify ArgoCD application definitions:

```bash
# Edit ArgoCD application (centralized in argocd/ directory)
vim argocd/dmp-roadmap-{environment}.yaml
```

## Change Process

### For Internal Contributors

1. **Create Feature Branch**

   ```bash
   git checkout -b feature/description
   ```

2. **Make Changes**

   - Edit configuration files in `environments/`
   - Update ArgoCD applications in `argocd/` if needed
   - Update sealed secrets if needed
   - Test changes locally where possible

3. **Commit Changes**

   ```bash
   git add .
   git commit -m "feat: descriptive commit message"
   ```

4. **Create Pull Request**

   - Push branch to GitHub
   - Create pull request with description
   - Request review from team members

5. **Deploy to Development**
   - Merge to main branch
   - ArgoCD automatically deploys to development
   - Verify deployment success

### For External Contributors

1. **Fork Repository**

   - Fork this repository to your GitHub account

2. **Clone Your Fork**

   ```bash
   git clone https://github.com/YOUR-USERNAME/roadmap-deploy.git
   cd roadmap-deploy
   ```

3. **Create Feature Branch**

   ```bash
   git checkout -b feature/description
   ```

4. **Make Changes**

   - Follow the same configuration steps as internal contributors
   - Edit files in `environments/` and `argocd/` as needed

5. **Commit and Push to Fork**

   ```bash
   git add .
   git commit -m "feat: descriptive commit message"
   git push origin feature/description
   ```

6. **Create Pull Request**
   - Open a pull request from your fork to the main repository
   - Provide detailed description of changes and reasoning
   - Wait for review and approval from maintainers

### Environment Promotion

Follow this sequence for promoting changes:

**Note**: Environment promotion should also follow the PR workflow for protected main branch.

1. **Development → Staging**

   ```bash
   # Create promotion branch
   git checkout -b promote/dev-to-staging
   git pull origin main

   # Option A: Copy dev configuration and adjust for staging
   cp environments/dev/values.yaml environments/staging/values.yaml
   # Edit staging-specific values (domain, image tags, etc.)
   vim environments/staging/values.yaml

   # Option B: Manually add configurations to align with staging environment specifics
   vim environments/staging/values.yaml

   git add .
   git commit -m "promote: dev to staging"
   git push origin promote/dev-to-staging
   ```

2. **Staging → Production**

   ```bash
   # Create promotion branch
   git checkout -b promote/staging-to-production
   git pull origin main

   # Option A: Move common configuration to base, then update prod
   vim environments/base/values.yaml  # Add common configs
   vim environments/dev/values.yaml   # Remove now-common configs
   vim environments/staging/values.yaml  # Remove now-common configs
   vim environments/prod/values.yaml  # Add production-specific configs

   # Option B: Copy staging configuration and adjust for production
   cp environments/staging/values.yaml environments/prod/values.yaml
   vim environments/prod/values.yaml  # Edit production-specific values

   git add .
   git commit -m "promote: staging to production"
   git push origin promote/staging-to-production
   ```

   Then create pull requests for both promotion scenarios and get approval before merging.

## Best Practices

### Configuration Management

- **Environment Parity**: Keep environments as similar as possible
- **Minimal Overrides**: Only override values that must differ between environments
- **Documentation**: Comment complex configuration changes
- **Validation**: Use `helm template` locally to validate changes

### Secrets Management

- **Encrypted Storage**: Always use sealed secrets for sensitive data
- **Rotation**: Regularly rotate secrets following security policies
- **Minimal Scope**: Only include necessary secrets in each environment
- **Backup**: Maintain secure backups of unsealed secrets

### Change Management

- **Small Changes**: Make incremental changes for easier troubleshooting
- **Testing**: Test in development before promoting to higher environments
- **Rollback Plan**: Ensure changes can be easily rolled back
- **Communication**: Notify team of significant changes

## Troubleshooting

### ArgoCD Sync Issues

1. **Check Application Status**

   - Open ArgoCD UI
   - Navigate to Applications page
   - Look for your application in the list

2. **View Sync Details**

   - Click on the application name
   - Review the Summary tab for sync status
   - Check the Events tab for error messages

3. **Force Sync**
   - Click the "Sync" button in the application view
   - Configure sync options (Force, Prune, etc.)
   - Click "Synchronize"

See [ArgoCD Guide](docs/argocd-guide.md) for detailed UI instructions.

### Helm Template Validation

```bash
# Validate Helm templates locally (if you have chart access)
helm template dmp-roadmap ../roadmap/charts/dmp-roadmap \
  -f environments/base/values.yaml \
  -f environments/{environment}/values.yaml
```

### Sealed Secrets Issues

1. **Check SealedSecret Status**

   - Navigate to your application in ArgoCD UI
   - Look for SealedSecret resources in the tree view
   - Check if corresponding Secret resources exist
   - Review resource events for decryption errors

2. **Verify Secret Creation**
   - In ArgoCD, check that SealedSecret resources show as "Healthy"
   - Verify corresponding Secret resources exist in the tree view
   - Check application Events tab for secret-related errors

See [Sealed Secrets Guide](docs/sealed-secrets-guide.md) for detailed instructions.

## Emergency Procedures

### Rollback Deployment

Since the main branch is protected, rollbacks require a pull request:

```bash
# Create rollback branch
git checkout -b rollback/revert-problematic-change

# Rollback to previous commit
git revert HEAD

# Push branch and create PR
git push origin rollback/revert-problematic-change
```

Then:

1. Create pull request for the rollback
2. Get approval and merge (expedited process for critical issues)
3. Monitor the rollback in ArgoCD UI:
   - Navigate to the application
   - Wait for automatic sync or trigger manual sync
   - Verify application returns to healthy state

### Manual Intervention

If ArgoCD sync fails and manual intervention is needed:

1. Contact the platform team
2. Document any manual changes made outside GitOps
3. Update this repository to match manual changes
4. Investigate and fix the root cause
5. Resume normal GitOps workflow

## Support

For deployment issues:

- Check ArgoCD dashboard
- Review application logs
- Consult `/docs/` directory for operational guides
- Contact the platform team for cluster-level issues
