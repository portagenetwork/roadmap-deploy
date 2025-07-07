# Sealed Secrets Guide

## Overview

Sealed Secrets provide a way to encrypt secrets that can be stored in Git repositories. The SealedSecret controller running
in the cluster decrypts these secrets and creates regular Kubernetes secrets.

## How Sealed Secrets Work

1. **Encrypt**: Use `kubeseal` to encrypt regular Kubernetes secrets
2. **Store**: Commit encrypted SealedSecret manifests to Git
3. **Deploy**: ArgoCD deploys SealedSecrets to the cluster
4. **Decrypt**: SealedSecret controller decrypts and creates regular secrets
5. **Use**: Applications consume the decrypted secrets

## Prerequisites

### Install kubeseal CLI

For installation instructions and the latest releases, visit the official Sealed Secrets repository:
**[https://github.com/bitnami-labs/sealed-secrets](https://github.com/bitnami-labs/sealed-secrets)**

## Setup Methods

### Method 1: Using Public Certificate (Recommended for Getting Started)

This method uses a public certificate that you can save locally for offline use.

#### Step 1: Fetch and Save Public Certificate

```bash
# Fetch the public certificate from your cluster
kubeseal --fetch-cert --controller-name=sealed-secrets-controller --controller-namespace=sealed-secrets > public-cert.pem

# Or if you don't have cluster access yet, use this placeholder:
# (Replace with your actual cluster's public certificate)
cat > public-cert.pem <<EOF
-----BEGIN CERTIFICATE-----
MIIErjCCApagAwIBAgIRAJWJaVqgIE7WnuSqhMiODPEwDQYJKoZIhvcNAQELBQAw
... (your cluster's public certificate) ...
-----END CERTIFICATE-----
EOF
```

#### Step 2: Create Sealed Secrets Using Local Certificate

```bash
# Create a regular secret manifest
kubectl create secret generic my-secret \
  --from-literal=username=myuser \
  --from-literal=password=mypassword \
  --namespace=dmp-roadmap-dev \
  --dry-run=client -o yaml > secret.yaml

# Seal the secret using local certificate
kubeseal --cert public-cert.pem --format yaml < secret.yaml > sealed-secret.yaml

# Clean up temporary file
rm secret.yaml
```

### Method 2: Using Cluster Access (When Available)

When you have direct cluster access, kubeseal can fetch the certificate automatically.

```bash
# Create sealed secret directly from cluster
kubectl create secret generic my-secret \
  --from-literal=username=myuser \
  --from-literal=password=mypassword \
  --namespace=dmp-roadmap-dev \
  --dry-run=client -o yaml | \
  kubeseal --controller-name=sealed-secrets-controller \
           --controller-namespace=sealed-secrets \
           --format yaml > sealed-secret.yaml
```

## Creating Sealed Secrets

### General Pattern

All sealed secrets follow this basic pattern:

```bash
# 1. Create regular Kubernetes secret (dry-run)
kubectl create secret [TYPE] [NAME] \
  [SECRET_DATA] \
  --namespace=dmp-roadmap-{environment} \
  --dry-run=client -o yaml > temp-secret.yaml

# 2. Seal the secret
kubeseal --cert public-cert.pem --format yaml < temp-secret.yaml > environments/{environment}/sealed-secrets/[NAME].yaml

# 3. Clean up temporary file
rm temp-secret.yaml
```

### Secret Types and Data Sources

**From literal values**:

```bash
--from-literal=key1=value1 --from-literal=key2=value2
```

**From environment variables**:

```bash
--from-literal=username=$DB_USERNAME --from-literal=password=$DB_PASSWORD
```

**From files**:

```bash
--from-file=config.yaml --from-file=credentials.json
```

**TLS certificates**:

```bash
kubectl create secret tls [NAME] --cert=certificate.crt --key=private.key
```

## DMP Roadmap Secret Types

The DMP Assistant application requires several types of secrets across environments. Each secret should be created for the
appropriate namespace (`dmp-roadmap-{environment}`).

### Database Secrets (`dmp-roadmap-db`)

Contains PostgreSQL connection details:

- `POSTGRES_HOST` - Database server hostname
- `POSTGRES_PORT` - Database port (typically 5432)
- `POSTGRES_USER` - Database username
- `POSTGRES_PASSWORD` - Database password
- `POSTGRES_DB` - Database name

### Rails Application Secrets (`dmp-roadmap-devise-secrets`)

Contains Rails and authentication secrets:

- `SECRET_KEY_BASE` - Rails secret key base
- `DEVISE_SECRET_KEY` - Devise authentication secret

### Integration Secrets (`dmp-roadmap-integrations`)

Contains third-party service credentials:

- `ORCID_CLIENT_ID` / `ORCID_CLIENT_SECRET` - ORCID authentication
- `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` - Google OAuth
- Additional integration credentials as needed

### Storage Secrets (CephFS)

Contains storage access credentials:

- `cephfs-locales-sealed-secret.yaml` - Locales storage access
- `cephfs-uploads-sealed-secret.yaml` - File uploads storage access

## Environment Considerations

Each environment (dev, staging, prod) should have:

- **Unique credentials** - Never share secrets between environments
- **Environment-specific values** - Hostnames, databases, and service endpoints
- **Appropriate security levels** - Production secrets should be more complex
- **Proper namespacing** - Always use `dmp-roadmap-{environment}` namespace

## Best Practices

### Secret Management

1. **Never commit unencrypted secrets** to Git
2. **Use environment-specific namespaces** for proper isolation
3. **Rotate secrets regularly** by creating new sealed secrets
4. **Use descriptive names** for secrets and keys
5. **Document secret contents** without revealing values

### File Organization

```text
environments/
├── dev/
│   └── sealed-secrets/
│       ├── dmp-roadmap-db.yaml
│       ├── dmp-roadmap-devise-secrets.yaml
│       └── dmp-roadmap-integrations.yaml
├── staging/
│   └── sealed-secrets/
│       ├── dmp-roadmap-db.yaml
│       ├── dmp-roadmap-devise-secrets.yaml
│       └── dmp-roadmap-integrations.yaml
└── prod/
    └── sealed-secrets/
        ├── dmp-roadmap-db.yaml
        ├── dmp-roadmap-devise-secrets.yaml
        └── dmp-roadmap-integrations.yaml
```

### Security Practices

1. **Store public certificate securely** but it's safe to share
2. **Never share private certificates** (these stay in the cluster)
3. **Use strong passwords** for all secrets
4. **Limit access** to sealed secrets repository
5. **Audit secret usage** regularly

## Troubleshooting

### Common Issues

#### "cannot fetch certificate" Error

**Solution using public certificate method**:

```bash
# Make sure you have the public certificate saved locally
kubeseal --cert public-cert.pem --format yaml < secret.yaml > sealed-secret.yaml
```

#### Wrong Namespace Error

**Problem**: Secret encrypted for wrong namespace
**Solution**: Always specify the correct namespace when creating secrets:

```bash
kubectl create secret generic my-secret \
  --from-literal=key=value \
  --namespace=dmp-roadmap-{environment} \  # <- Important!
  --dry-run=client -o yaml | \
  kubeseal --cert public-cert.pem --format yaml > sealed-secret.yaml
```

#### SealedSecret Not Creating Regular Secret

**Check in ArgoCD UI**:

1. Navigate to the application
2. Look for SealedSecret resource in the tree
3. Check if corresponding Secret resource exists
4. Review Events tab for decryption errors

**Common causes**:

- Wrong namespace in SealedSecret
- SealedSecret controller not running
- Certificate mismatch

### Verification

#### Check SealedSecret Status

In ArgoCD UI:

1. Navigate to your application
2. Find the SealedSecret resource
3. Check its status and events
4. Verify corresponding Secret exists

#### Verify Secret Contents

```bash
# List secrets in namespace
kubectl get secrets -n dmp-roadmap-{environment}

# Check if secret exists (don't view contents in production)
kubectl get secret dmp-roadmap-db -n dmp-roadmap-{environment}
```

## Updating Secrets

### Process for Secret Updates

1. **Create new sealed secret** with updated values using the general pattern above
2. **Commit to Git** repository following the normal PR workflow
3. **ArgoCD sync** will apply the changes automatically
4. **Pods restart** automatically to use new secret values

**Important**: Always update the entire secret with all key-value pairs, as sealed secrets replace the entire secret object.

## Backup and Recovery

### Backup Sealed Secrets

```bash
# Backup all sealed secrets
mkdir -p sealed-secrets-backup
cp -r environments/*/sealed-secrets/ sealed-secrets-backup/
tar -czf sealed-secrets-backup-$(date +%Y%m%d).tar.gz sealed-secrets-backup/
```

### Recovery Process

1. **Restore sealed secret files** from backup
2. **Commit to Git** repository
3. **ArgoCD sync** to apply secrets
4. **Verify applications** can access secrets

## Security Considerations

### What's Safe to Share

- **Public certificates**: Safe to share and store in repositories
- **SealedSecret manifests**: Encrypted and safe to store in Git
- **Documentation**: This guide and processes

### What to Keep Private

- **Private certificates**: Never leave the cluster
- **Unencrypted secrets**: Never commit to Git
- **Secret values**: Never share actual secret values

### Access Control

- **Repository access**: Limit who can modify sealed secrets
- **Certificate access**: Control who can fetch certificates
- **Namespace isolation**: Use separate namespaces for environments
- **Audit logging**: Monitor secret access and changes

## Getting Help

### Information to Gather

When troubleshooting sealed secrets:

1. **Error messages** from kubeseal command
2. **SealedSecret manifest** (safe to share)
3. **Namespace** and **environment** information
4. **ArgoCD application status** screenshot
5. **Recent changes** made to secrets

### Common Commands for Troubleshooting

```bash
# Check kubeseal version
kubeseal --version

# Validate certificate
kubeseal --cert public-cert.pem --validate

# Test encryption (dry run)
echo "test" | kubeseal --cert public-cert.pem --raw \
  --from-file=/dev/stdin \
  --namespace=dmp-roadmap-{environment} \
  --name=test-secret
```

For detailed troubleshooting and advanced usage, refer to the official documentation:
**[Sealed Secrets Documentation](https://github.com/bitnami-labs/sealed-secrets)**
