# /argocd-app-rollback

Rollback an ArgoCD application to a previous deployment revision.

This command reverts your application to a previously deployed state, useful when a new deployment causes issues and you need to quickly restore service.

## Usage

```
/argocd-app-rollback <app-name> [<revision-id>] [--prune]
```

## Parameters

- `<app-name>` - Name of the ArgoCD application (required)
- `<revision-id>` - Specific history revision to rollback to (optional, defaults to previous)
- `--prune` - Remove resources that don't exist in the target revision

## Examples

```bash
# Rollback to the immediately previous deployment
/argocd-app-rollback my-payment-service

# Rollback to a specific revision from history
/argocd-app-rollback my-payment-service 14

# Rollback and prune resources added in newer versions
/argocd-app-rollback my-payment-service 14 --prune
```

## What it does

1. **Identifies target revision** - The Git commit/tag to rollback to
2. **Syncs to that revision** - Updates cluster to match that historical state
3. **Updates tracking** - ArgoCD tracks this as a new sync operation
4. **Applies changes** - Kubernetes resources are updated

## Workflow Example

```bash
# 1. Current deployment is broken
/check-argocd-app production-api
# Output: Health: Degraded (v3.0.0)

# 2. Check history to find last good version
/argocd-app-history production-api
# ID 18: v3.0.0 (now)     - Succeeded, Health: Degraded ❌
# ID 17: v2.9.1 (5m ago)  - Succeeded, Health: Healthy ✅
# ID 16: v2.9.0 (1h ago)  - Succeeded, Health: Healthy ✅

# 3. Rollback to the last healthy version
/argocd-app-rollback production-api 17

# 4. Verify rollback succeeded
/check-argocd-app production-api
# Output: Health: Healthy (v2.9.1) ✅

# 5. Check pods are running
/check-deployment production production-api
```

## When to use

### Production Incident Response
Bad deployment causing outages:
```bash
# Fast rollback to restore service
/argocd-app-rollback production-api
/check-argocd-app production-api
```

### Failed Deployment Recovery
New version won't deploy:
```bash
# Deployment failed, rollback to stable version
/argocd-app-history staging-app
/argocd-app-rollback staging-app 12
```

### Testing a Known Good State
Reproduce an issue or verify a previous version:
```bash
# Rollback to test if issue exists in older version
/argocd-app-rollback dev-app 8
# Run tests
# Roll forward again when done
```

### Reverting Bad Configuration
Configuration change caused problems:
```bash
# Rollback to before the config change
/argocd-app-rollback my-app 20
```

## Important Considerations

### Rollback vs. Git Revert

**Rollback (this command):**
- Temporarily deploys an older version
- ArgoCD will try to sync forward again if auto-sync is enabled
- Good for immediate incident response

**Git Revert (recommended for permanent fix):**
```bash
# Revert the problematic commit in Git
git revert <bad-commit>
git push

# Then sync ArgoCD
/sync-argocd-app my-app
```

### Auto-Sync Behavior

⚠️ **If auto-sync is enabled**, ArgoCD will detect that the cluster doesn't match the latest Git state and sync forward again!

**Solutions:**
1. **Disable auto-sync temporarily:**
   ```bash
   argocd app set my-app --sync-policy none
   /argocd-app-rollback my-app 14
   # Fix the issue in Git
   # Re-enable auto-sync
   argocd app set my-app --sync-policy automated
   ```

2. **Revert in Git immediately:**
   ```bash
   /argocd-app-rollback my-app 14
   # Quickly revert the bad commit in Git
   git revert HEAD
   git push
   ```

### Database Migrations

⚠️ **Be careful with rollbacks if your app runs database migrations!**

Rolling back the application doesn't rollback database schema changes:
- Older app version might be incompatible with newer schema
- Consider the full migration strategy before rolling back

## Options Explained

### `--prune`

**Without `--prune`:**
```bash
/argocd-app-rollback my-app 10
# Resources added in revisions 11-15 remain in cluster
```

**With `--prune`:**
```bash
/argocd-app-rollback my-app 10 --prune
# Resources added after revision 10 are DELETED
```

⚠️ **Use with caution** - this deletes resources!

**When to use:**
- New resources were added that are causing issues
- You want a clean rollback to exactly match the old state
- New ConfigMaps/Secrets are conflicting

## Post-Rollback Actions

### 1. Verify Health
```bash
/check-argocd-app my-app
/check-deployment <namespace> <deployment>
```

### 2. Check Logs
```bash
/view-pod-logs <namespace> <pod-name>
```

### 3. Monitor Events
```bash
/check-events <namespace>
```

### 4. Fix the Root Cause in Git
```bash
# Review what went wrong
git diff <good-revision> <bad-revision>

# Fix the issue
# Commit and push

# Sync forward with the fix
/sync-argocd-app my-app
```

### 5. Re-enable Auto-Sync (if disabled)
```bash
argocd app set my-app --sync-policy automated
```

## Common Issues

### "Rollback successful but app still unhealthy"
- The previous version had the same issue
- Try rolling back further: `/argocd-app-rollback my-app <earlier-id>`
- Check if infrastructure or dependencies changed

### "Rollback keeps getting overwritten"
- Auto-sync is enabled
- Disable auto-sync or revert in Git

### "Can't rollback - revision not found"
- Revision might be too old and pruned
- Check available revisions: `/argocd-app-history my-app --limit 50`

### "Rollback succeeded but different pods failing"
- Cluster state changed (nodes, resources, dependencies)
- Previous version might not be compatible with current cluster state

## Safety Checklist

Before rolling back production:

- [ ] Identified the target revision from history
- [ ] Verified that revision was previously healthy
- [ ] Considered database migration compatibility
- [ ] Notified team members
- [ ] Prepared to revert the rollback if needed
- [ ] Plan to fix root cause in Git

## Related Commands

- `/argocd-app-history` - Find the revision to rollback to
- `/check-argocd-app` - Verify rollback succeeded
- `/argocd-app-diff` - See what will change during rollback
- `/sync-argocd-app` - Sync forward after fixing Git
