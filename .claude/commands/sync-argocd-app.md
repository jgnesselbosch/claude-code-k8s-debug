# /sync-argocd-app

Manually trigger an ArgoCD application sync to reconcile the cluster state with Git.

This command forces ArgoCD to apply the latest configuration from Git to your cluster, useful when auto-sync is disabled or when you need immediate reconciliation.

## Usage

```
/sync-argocd-app <app-name> [--prune] [--force] [--dry-run]
```

## Parameters

- `<app-name>` - Name of the ArgoCD application (required)
- `--prune` - Delete resources that exist in cluster but not in Git
- `--force` - Force sync even if no changes detected
- `--dry-run` - Preview what would be synced without applying

## Examples

```bash
# Basic sync
/sync-argocd-app my-payment-service

# Sync and remove resources not in Git
/sync-argocd-app my-payment-service --prune

# Preview sync without applying
/sync-argocd-app my-payment-service --dry-run

# Force sync even if app appears in-sync
/sync-argocd-app my-payment-service --force
```

## What it does

1. **Fetches latest Git state** - Pulls the target revision from your repository
2. **Renders manifests** - Processes Helm charts, Kustomize, or plain YAML
3. **Compares with cluster** - Identifies differences between desired and actual state
4. **Applies changes** - Updates Kubernetes resources to match Git
5. **Reports results** - Shows what was created, updated, or deleted

## When to use

### Manual Sync Needed
- Auto-sync is disabled on the application
- You made Git changes and want immediate deployment
- Previous sync failed and you fixed the issue

### Drift Correction
- Someone ran `kubectl apply` manually and you want to revert to Git
- Configuration drift detected (OutOfSync status)
- Resources were deleted and need to be recreated

### Troubleshooting
- Deployment is stuck and you want to retry
- Testing a fix you just pushed to Git
- Forcing reconciliation after ArgoCD upgrade

## Options Explained

### `--prune`
**Use when:** Resources exist in cluster but were removed from Git

**Example scenario:**
```bash
# You deleted a ConfigMap from Git but it's still in the cluster
/sync-argocd-app my-app --prune
# This will delete the ConfigMap from the cluster
```

⚠️ **Warning:** This deletes resources. Review with `--dry-run` first.

### `--force`
**Use when:** App shows as Synced but you know changes exist

**Example scenario:**
```bash
# Git SHA changed but ArgoCD didn't detect it
/sync-argocd-app my-app --force
```

### `--dry-run`
**Use when:** You want to see what would change without applying

**Example scenario:**
```bash
# Preview changes before actually syncing
/sync-argocd-app my-app --dry-run --prune
# Review output, then sync for real if it looks correct
```

## Common Issues

### Sync Fails with "Permission Denied"
- ArgoCD service account lacks RBAC permissions
- Check ArgoCD application project restrictions
- Verify namespace permissions

### Sync Stuck at "Progressing"
- Deployment rollout taking longer than expected
- Use `/check-deployment` to inspect pod status
- Check if readiness probes are failing

### "OutOfSync" After Sync
- Auto-sync pruning disabled but resources need deletion
- Re-run with `--prune` flag
- Check for manual cluster changes happening continuously

### Sync Fails with "Manifest Error"
- Invalid YAML/Helm syntax in Git
- Helm value conflicts
- Kustomize rendering errors
- Check ArgoCD application logs

## Safety Tips

1. **Always review with dry-run first for production apps**
   ```bash
   /sync-argocd-app prod-api --dry-run
   ```

2. **Use prune carefully** - it deletes resources
   ```bash
   # Review what would be deleted
   /argocd-app-diff my-app
   # Then prune if appropriate
   /sync-argocd-app my-app --prune
   ```

3. **Check health after sync**
   ```bash
   /sync-argocd-app my-app
   # Wait a moment, then verify
   /check-argocd-app my-app
   ```

## Follow-up Commands

After syncing, verify the deployment:

```bash
# Check ArgoCD application health
/check-argocd-app my-payment-service

# Check pod-level status
/check-deployment production my-payment-service

# View application events
/check-events production
```

## Related

- `/check-argocd-app` - Check app sync and health status
- `/argocd-app-diff` - See what will change before syncing
- `/argocd-app-history` - View past sync operations
