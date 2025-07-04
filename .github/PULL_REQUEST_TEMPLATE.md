# Pull Request

## Description

Brief description of the changes in this PR.

## Type of Change

Please delete options that are not relevant.

- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Configuration update (environment-specific configuration changes)
- [ ] Documentation update
- [ ] Refactoring (no functional changes)

## Environments Affected

- [ ] Development
- [ ] Staging
- [ ] Production
- [ ] All environments
- [ ] N/A (documentation/tooling only)

## Changes Made

Detailed description of what was changed:

-
-
-

## Testing

Describe the tests that you ran to verify your changes:

- [ ] Local validation (YAML linting, etc.)
- [ ] Helm template validation
- [ ] ArgoCD application validation
- [ ] Pre-commit hooks passed
- [ ] GitHub Actions validation passed
- [ ] Manual testing in development environment

## Configuration Validation

- [ ] All YAML files are valid
- [ ] Helm templates render correctly
- [ ] ArgoCD applications reference correct files
- [ ] Sealed secrets are properly encrypted
- [ ] No sensitive information in plaintext

## Deployment Plan

If this requires special deployment steps:

1.
2.
3.

## Rollback Plan

How to rollback if issues occur:

1.
2.
3.

## Screenshots (if applicable)

Add screenshots to help explain your changes.

## Security Considerations

- [ ] No secrets are exposed in plaintext
- [ ] All secrets use sealed secrets encryption
- [ ] Changes follow security best practices
- [ ] No unnecessary permissions granted

## Additional Notes

Any additional information that reviewers should know:

---

## For Reviewers

### Review Checklist

- [ ] Configuration changes are appropriate for target environment(s)
- [ ] Security best practices are followed
- [ ] Documentation is updated appropriately
- [ ] Change risk is acceptable
- [ ] Deployment plan is clear and safe
- [ ] Rollback plan is feasible
