# ArgoCD UI Guide

## Overview

ArgoCD provides a web-based UI for managing GitOps deployments. This guide covers how to use the ArgoCD UI to monitor
and manage the DMP Assistant deployments across different environments.

## Accessing ArgoCD UI

Access the ArgoCD UI through your cluster's ArgoCD endpoint (`https://kestrel.arbutus.cloud/argocd`).

## Application Overview

### Viewing Applications

1. **Application List**: The main dashboard shows all applications
   - Look for applications named: `dmp-roadmap-dev`, `dmp-roadmap-staging`, `dmp-roadmap-prod`
   - Each application shows:
     - **Health Status**: Healthy, Progressing, Degraded, or Unknown
     - **Sync Status**: Synced, OutOfSync, or Unknown
     - **Last Sync**: When the last synchronization occurred

2. **Application Details**: Click on an application to view detailed information
   - **Summary**: High-level application information
   - **Parameters**: Helm values and configuration
   - **Manifests**: Generated Kubernetes manifests
   - **Events**: Recent application events
   - **Logs**: Application and sync logs

### Application Health Status

- **🟢 Healthy**: All resources are running correctly
- **🟡 Progressing**: Deployment is in progress
- **🔴 Degraded**: Some resources are failing
- **⚫ Unknown**: Health status cannot be determined

### Sync Status

- **🟢 Synced**: Git repository matches deployed resources
- **🟡 OutOfSync**: Git repository has changes not yet deployed
- **⚫ Unknown**: Sync status cannot be determined

## Managing Deployments

### Manual Sync

When you make changes to the Git repository, ArgoCD will automatically detect them. To manually trigger a sync:

1. **Navigate to Application**: Click on the application name
2. **Click Sync Button**: Located in the top toolbar
3. **Configure Sync Options**:
   - **Prune**: Remove resources not defined in Git
   - **Dry Run**: Preview changes without applying
   - **Force**: Ignore validation errors
4. **Click Synchronize**: Start the sync process

### Sync Options Explained

- **Prune**: Removes Kubernetes resources that exist in the cluster but are not defined in Git
- **Dry Run**: Shows what would be changed without actually applying changes
- **Force**: Bypasses validation and applies changes even if there are conflicts
- **Replace**: Uses `kubectl replace` instead of `kubectl apply`

### Refresh Application

If the application status seems stale:

1. **Navigate to Application**: Click on the application name
2. **Click Refresh Button**: Located in the top toolbar (circular arrow icon)
3. **Hard Refresh**: Hold Shift while clicking for a complete refresh

## Monitoring and Troubleshooting

### Viewing Resource Status

1. **Application Tree View**: Shows all Kubernetes resources
   - **Green**: Resource is healthy
   - **Yellow**: Resource is progressing or has warnings
   - **Red**: Resource has errors
   - **Gray**: Resource status unknown

2. **Resource Details**: Click on any resource to view:
   - **Summary**: Basic resource information
   - **Manifest**: Full Kubernetes manifest
   - **Events**: Recent events for this resource
   - **Logs**: Container logs (for pods)

### Common Issues and Solutions

#### Application Stuck in "Progressing" State

**What to check**:

1. Click on the application to view details
2. Look at the **Events** tab for error messages
3. Check individual resources in the tree view for red/yellow status
4. Click on failing resources to see detailed error messages

**Solutions**:

1. **Refresh** the application to update status
2. **Sync** with **Force** option if there are conflicts
3. **Prune** resources if there are orphaned resources

#### "OutOfSync" Status

**What it means**: Git repository has changes that haven't been deployed

**Solutions**:

1. **Sync** the application to apply Git changes
2. Review the **Diff** tab to see what will change
3. Use **Dry Run** to preview changes before applying

#### Resource Failures

**Investigation steps**:

1. Click on the failing resource in the tree view
2. Check the **Events** tab for error messages
3. For pods, check the **Logs** tab
4. Look for resource conflicts or configuration errors

### Viewing Logs

1. **Navigate to Pod**: Find the pod in the application tree
2. **Click on Pod**: Open pod details
3. **Logs Tab**: View container logs
4. **Log Options**:
   - **Follow**: Stream logs in real-time
   - **Previous**: View logs from previous container instance
   - **Tail Lines**: Limit number of log lines shown

### Viewing Events

1. **Application Events**: Available in the application details **Events** tab
2. **Resource Events**: Available in individual resource details
3. **Event Information**:
   - **Type**: Normal, Warning, or Error
   - **Reason**: Event category
   - **Message**: Detailed event description
   - **Timestamp**: When the event occurred

## Configuration Management

### Viewing Configuration

1. **Parameters Tab**: Shows Helm values being used
   - **Base Values**: From `environments/base/values.yaml`
   - **Environment Values**: From `environments/{env}/values.yaml`
   - **Merged Values**: Final computed values

2. **Manifests Tab**: Shows generated Kubernetes manifests
   - **Search**: Filter manifests by resource type or name
   - **Diff**: Compare with live cluster state

### Understanding Value Hierarchy

ArgoCD applies values in this order:

1. **Base values** from `environments/base/values.yaml`
2. **Environment-specific values** from `environments/{env}/values.yaml`
3. **Final merged values** used for deployment

### Tracking Changes

1. **History Tab**: Shows deployment history
   - **Revision**: Git commit hash
   - **Deployed At**: Timestamp of deployment
   - **Deployed By**: User who triggered deployment
   - **Status**: Success or failure

2. **Diff View**: Compare any two revisions
   - **Source**: Git repository state
   - **Live**: Current cluster state
   - **Desired**: What will be applied

## Best Practices

### Monitoring

1. **Regular Checks**: Monitor application health daily
2. **Watch for OutOfSync**: Review and sync promptly
3. **Check Events**: Review events for warnings or errors
4. **Monitor Resources**: Ensure all resources are healthy

### Troubleshooting

1. **Start with Events**: Check application and resource events first
2. **Check Logs**: Review pod logs for application errors
3. **Use Dry Run**: Preview changes before applying
4. **Incremental Sync**: Sync one change at a time for easier troubleshooting

### Safety

1. **Review Diffs**: Always check what will change before syncing
2. **Use Staging**: Test changes in staging before production
3. **Backup State**: Note current working state before major changes
4. **Monitor After Changes**: Watch applications after syncing

## Environment-Specific Considerations

### Development Environment

- **Auto-sync**: May be enabled for rapid development
- **Frequent Changes**: Expect more frequent deployments
- **Debugging**: More detailed logging may be enabled

### Staging Environment

- **Manual Sync**: Usually requires manual approval
- **Testing**: Used for validation before production
- **Monitoring**: Close monitoring during testing phases

### Production Environment

- **Strict Controls**: Manual sync only
- **Change Windows**: Deployments during scheduled maintenance
- **Monitoring**: Continuous monitoring and alerting
- **Rollback Plan**: Always have a rollback strategy

## Getting Help

### Information to Gather

When seeking help with ArgoCD issues:

1. **Application name** and environment
2. **Screenshots** of error messages
3. **Sync status** and health status
4. **Recent changes** made to Git repository
5. **Error messages** from Events tab

### UI Navigation Tips

1. **Browser Bookmarks**: Bookmark frequently accessed applications
2. **Multiple Tabs**: Open different environments in separate tabs
3. **Keyboard Shortcuts**: Learn ArgoCD keyboard shortcuts
4. **Search**: Use search functionality to find specific resources

### Common UI Elements

- **🔄 Sync**: Synchronize Git changes
- **↻ Refresh**: Update application status
- **🔍 Search**: Find resources or applications
- **📊 Diff**: Compare states
- **📜 Logs**: View container logs
- **⚙️ Settings**: Application configuration
- **📊 Events**: View application events
