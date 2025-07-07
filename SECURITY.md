# Security Policy

## Overview

The security of the DMP Assistant deployment configuration is important to us. This document outlines our security
practices and how to report vulnerabilities.

## Supported Versions

We actively maintain security updates for the following:

| Component                  | Supported Version   |
| -------------------------- | ------------------- |
| ArgoCD Applications        | Current main branch |
| Sealed Secrets             | Current main branch |
| Environment Configurations | Current main branch |
| Documentation              | Current main branch |

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub issues.**

Instead, please report vulnerabilities by emailing: **<support@portagenetwork.ca>**

Include the following information in your report:

- **Type of issue** (e.g., exposed secret, configuration vulnerability, etc.)
- **Full paths** of source file(s) related to the manifestation of the issue
- **Location** of the affected source code (tag/branch/commit or direct URL)
- **Step-by-step instructions** to reproduce the issue
- **Impact** of the issue, including how an attacker might exploit it
- **Any special configuration** required to reproduce the issue

### What to expect

- **Acknowledgment**: We will acknowledge receipt of your vulnerability report within 72 hours
- **Assessment**: We will assess the vulnerability and determine its severity within 5 business days
- **Updates**: We will provide regular updates on our progress
- **Resolution**: We will work to resolve the vulnerability as quickly as possible
- **Disclosure**: We will coordinate with you on appropriate disclosure timing

### Responsible Disclosure

We ask that you:

- Allow us reasonable time to resolve the vulnerability before public disclosure
- Avoid accessing, modifying, or deleting data during your research
- Only interact with accounts you own or with explicit permission from the account holder
- Do not perform actions that could negatively affect our users or services

## Security Best Practices for Contributors

### Secrets Management

- **Never commit plaintext secrets** to this repository
- **Always use sealed secrets** for sensitive configuration
- **Rotate secrets regularly** following security policies
- **Use strong, unique passwords** for all environments
- **Validate sealed secrets** before committing

### Configuration Security

- **Follow principle of least privilege** in configurations
- **Separate environments** completely (no shared secrets/configs)
- **Use environment-specific namespaces** for proper isolation
- **Validate all YAML** before committing
- **Review all changes** through pull request process

### Access Control

- **Enable two-factor authentication** on your GitHub account
- **Use signed commits** where possible
- **Follow branch protection rules**
- **Request appropriate reviews** for changes

### Infrastructure Security

- **Keep ArgoCD applications up to date**
- **Monitor deployment logs** for anomalies
- **Use secure communication** (HTTPS/TLS) for all connections
- **Implement network policies** where appropriate
- **Regular security assessments** of deployed applications

## Security Features

### Automated Security Scanning

This repository includes:

- **Secret detection** via pre-commit hooks and CI/CD
- **Vulnerability scanning** with Trivy in GitHub Actions
- **YAML validation** to prevent misconfigurations
- **Dependency scanning** for security updates

### Deployment Security

- **GitOps principle**: All changes go through Git with audit trail
- **Sealed secrets**: Encrypted secrets safe for Git storage
- **Namespace isolation**: Environment separation
- **RBAC**: Role-based access controls in ArgoCD
- **Audit logging**: All deployment actions are logged

## Incident Response

In case of a security incident:

1. **Immediate containment**: Isolate affected systems
2. **Assessment**: Determine scope and impact
3. **Communication**: Notify stakeholders appropriately
4. **Remediation**: Apply fixes and security updates
5. **Post-incident review**: Learn and improve processes

## Security Resources

- [Sealed Secrets Security](https://github.com/bitnami-labs/sealed-secrets#security)
- [ArgoCD Security](https://argo-cd.readthedocs.io/en/stable/operator-manual/security/)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)

## Contact

For security-related questions or concerns, contact:

- **Email**: <support@portagenetwork.ca>
- **GitHub**: Create a private security advisory

## Updates

This security policy may be updated from time to time. Check this document regularly for the latest information.
