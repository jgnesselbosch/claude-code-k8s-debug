# /argocd-app-diff

Show the differences between the desired state (Git) and actual state (cluster) for an ArgoCD application.

This command helps you understand exactly what changes exist between your Git repository and what's currently deployed, making it easier to diagnose drift and sync issues.

## Usage

```
/argocd-app-diff <app-name> [--local <path>]
```

## Parameters

- `<app-name>` - Name of the ArgoCD application (required)
- `--local <path>` - Compare against local Git repo path instead of remote

## Examples

```bash
# Show differences for an application
/argocd-app-diff my-payment-service

# Compare with local changes (before pushing to Git)
/argocd-app-diff my-payment-service --local ./manifests
```

## What it shows

The diff displays:

1. **Added resources** - Resources in Git but not in cluster (will be created on sync)
2. **Removed resources** - Resources in cluster but not in Git (will be deleted if sync with --prune)
3. **Modified resources** - Resources that exist in both but have differences

### Example Output Format

```diff
===== apps/Deployment my-app/payment-service ======
--- live state (cluster)
+++ desired state (git)
@@ -10,7 +10,7 @@
   spec:
     containers:
     - name: payment-service
-      image: payment-service:v1.2.0
+      image: payment-service:v1.3.0
       env:
       - name: DATABASE_URL
-        value: postgres://old-db:5432
+        value: postgres://new-db:5432
```

## When to use

### Before Syncing
Verify what will change before triggering a sync:
```bash
/argocd-app-diff my-app
# Review changes
/sync-argocd-app my-app
```

### Investigating Drift
When app shows "OutOfSync", understand exactly what's different:
```bash
/check-argocd-app my-app
# Shows: OutOfSync
/argocd-app-diff my-app
# See exactly what changed
```

### Debugging Failed Syncs
If sync fails, the diff helps identify problematic resources:
```bash
/argocd-app-diff my-app
# Look for invalid configurations or conflicts
```

### Testing Local Changes
Before committing to Git, preview the impact:
```bash
/argocd-app-diff my-app --local ./k8s-manifests
# Verify your local changes look correct
```

## Common Patterns

### Image Tag Differences
```diff
- image: myapp:v1.0.0
+ image: myapp:v1.1.0
```
**Meaning:** Git has a newer image version

### Configuration Changes
```diff
- replicas: 2
+ replicas: 5
```
**Meaning:** Someone scaled manually or you updated Git

### Resource Additions
```
+ apiVersion: v1
+ kind: ConfigMap
+ metadata:
+   name: new-config
```
**Meaning:** New resource in Git, will be created on sync

### Manual Changes (Drift)
```diff
- value: "manual-override"
+ value: "git-value"
```
**Meaning:** Someone ran `kubectl` to modify the cluster directly

## Interpreting Results

### No differences shown
✅ Application is in sync - cluster matches Git

### Small differences (image tags, replicas)
🔄 Normal deployment changes - sync when ready

### Large differences (many resources)
⚠️ Investigate before syncing:
- Was there a major Git update?
- Did someone make manual cluster changes?
- Is this the correct Git branch/revision?

### Removed resources (--)
⚠️ These will be **deleted** if you sync with `--prune`
- Verify they should actually be removed
- Check if they were deleted from Git intentionally

## Workflow Example

```bash
# 1. Check app status
/check-argocd-app production-api
# Output: Status: OutOfSync, Health: Healthy

# 2. See what's different
/argocd-app-diff production-api
# Output shows: image tag changed from v2.1.0 -> v2.2.0

# 3. Verify this is expected (check Git commits)
# Git shows: "Update to v2.2.0 with bug fixes"

# 4. Sync to apply changes
/sync-argocd-app production-api

# 5. Verify deployment
/check-deployment production production-api
```

## Tips

1. **Always diff before sync** - Especially in production
2. **Watch for unexpected changes** - Could indicate manual modifications
3. **Check removed resources carefully** - Avoid accidental deletions
4. **Use `--local` for testing** - Validate changes before pushing to Git

## Related Commands

- `/check-argocd-app` - Check if app is in sync
- `/sync-argocd-app` - Apply the differences
- `/argocd-app-history` - See previous sync operations
- `/check-deployment` - Check actual pod status after sync
