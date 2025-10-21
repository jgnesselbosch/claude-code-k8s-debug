# /argocd-app-history

View the deployment history of an ArgoCD application, including past sync operations and their results.

This command shows when syncs occurred, which Git revisions were deployed, and whether they succeeded or failed - essential for troubleshooting and rollback decisions.

## Usage

```
/argocd-app-history <app-name> [--limit <n>]
```

## Parameters

- `<app-name>` - Name of the ArgoCD application (required)
- `--limit <n>` - Number of history entries to show (default: 10)

## Examples

```bash
# Show recent deployment history
/argocd-app-history my-payment-service

# Show last 20 deployments
/argocd-app-history my-payment-service --limit 20

# Show all deployment history
/argocd-app-history my-payment-service --limit 100
```

## What it shows

Each history entry includes:

1. **Revision ID** - Unique identifier for the deployment
2. **Git Commit** - SHA, branch, or tag that was deployed
3. **Timestamp** - When the sync occurred
4. **Status** - Whether the sync succeeded or failed
5. **Sync Initiator** - Who/what triggered the sync (user, automated, webhook)
6. **Sync Result** - Summary of changes (resources created/updated/deleted)

### Example Output

```
ID  REVISION                          DATE                  STATUS    INITIATOR
10  abc123f (v2.3.0)                  2025-01-20 14:32:11  Succeeded  john@company.com
9   def456a (v2.2.1)                  2025-01-20 10:15:33  Succeeded  argocd-auto-sync
8   789beef (v2.2.0)                  2025-01-19 16:42:19  Failed     webhook
7   321cafe (v2.1.5)                  2025-01-19 09:11:07  Succeeded  alice@company.com
```

## When to use

### After a Failed Deployment
Identify what changed and when things broke:
```bash
/argocd-app-history my-app
# Look for recent Failed status
# Compare working revision vs failed revision
```

### Planning a Rollback
Find the last known good deployment:
```bash
/argocd-app-history my-app --limit 20
# Identify the last "Succeeded" revision before issues started
# Use that revision ID to roll back
```

### Investigating Frequent Changes
See if app is being synced too often or by unexpected sources:
```bash
/argocd-app-history my-app --limit 50
# Check frequency and initiators
# Identify if auto-sync is too aggressive
```

### Audit Trail
Track who deployed what and when:
```bash
/argocd-app-history production-api
# Review recent deployment activity
# Correlate with incidents or changes
```

## Common Scenarios

### Scenario 1: Rollback After Bad Deploy

```bash
# 1. Check current status - app is degraded
/check-argocd-app my-app
# Status: Synced, Health: Degraded

# 2. Review history to find last good deployment
/argocd-app-history my-app
# ID 15: v2.4.0 (current) - Succeeded but Degraded
# ID 14: v2.3.0 (10 min ago) - Succeeded, was Healthy
# ID 13: v2.2.0 (2 days ago) - Succeeded, was Healthy

# 3. Rollback to last known good version
/argocd-app-rollback my-app 14

# 4. Verify health restored
/check-argocd-app my-app
```

### Scenario 2: Understanding Deployment Frequency

```bash
/argocd-app-history my-app --limit 30

# Analysis:
# - 15 syncs in the last hour
# - All triggered by "argocd-auto-sync"
# - Indicates potential sync loop or flapping issue
```

### Scenario 3: Failed Sync Investigation

```bash
/argocd-app-history my-app

# Output shows:
# ID 8: abc123 (v3.0.0) - Failed - "manifest rendering error"

# Follow up:
# - Check what was in commit abc123
# - Look for Helm/Kustomize syntax errors
# - Review ArgoCD application logs
```

## Interpreting Status

### Succeeded
✅ Sync completed successfully
- Resources were applied to cluster
- No errors during sync operation
- **Note:** "Succeeded" doesn't guarantee healthy - check app health separately

### Failed
❌ Sync operation failed
- Could be invalid manifests
- RBAC permission issues
- Resource conflicts
- Network/API errors

### Running
🔄 Sync currently in progress
- Wait for completion
- Can take minutes for large apps

### Terminated
⚠️ Sync was cancelled or terminated
- Manual cancellation
- Timeout
- ArgoCD restart during sync

## Following Up

After reviewing history, you might need:

```bash
# Compare two revisions
git diff <old-sha> <new-sha>

# Rollback to previous version
/argocd-app-rollback my-app <revision-id>

# Check what changed in a failed sync
/argocd-app-diff my-app

# View detailed logs from a specific sync operation
# (Check ArgoCD UI or application controller logs)
```

## Tips

1. **Correlate with Git commits** - Match revision SHAs to your Git history
2. **Look for patterns** - Frequent failures might indicate CI/CD issues
3. **Check initiators** - Unexpected sources might indicate misconfiguration
4. **Compare timings** - Long gaps might mean auto-sync is disabled
5. **Note transition points** - When did it go from Healthy to Degraded?

## Related Commands

- `/check-argocd-app` - Check current sync/health status
- `/argocd-app-diff` - See what changed between revisions
- `/argocd-app-rollback` - Rollback to a previous revision
- `/sync-argocd-app` - Manually trigger a new sync
