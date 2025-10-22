# /check-argocd-app

Check the sync status, health, and recent events of an ArgoCD application.

This command uses the ArgoCD CLI to inspect application state and identify sync issues, health problems, or Git source discrepancies.

## Usage

```
/check-argocd-app <app-name> [--namespace <namespace>]
```

## Parameters

- `<app-name>` - Name of the ArgoCD application (required)
- `--namespace` - Namespace where the ArgoCD app resource is located (optional, defaults to "argocd")

## Examples

```bash
# Check a specific ArgoCD application
/check-argocd-app my-payment-service

# Check an app in a different namespace
/check-argocd-app my-frontend --namespace argocd-apps
```

## What it checks

1. **Sync Status** - Is the app in sync with Git?
   - `SYNCED` - Deployed resources match Git
   - `OUTOFSYNC` - Drift detected, resources don't match Git
   - `UNKNOWN` - Unable to determine sync status

2. **Health Status** - Is the application healthy?
   - `HEALTHY` - All resources are healthy
   - `PROGRESSING` - Deployment/rollout in progress
   - `DEGRADED` - Some resources are unhealthy
   - `SUSPENDED` - Application is suspended
   - `MISSING` - Expected resources not found

3. **Recent Sync Events** - Last sync attempt details
   - When was the last sync?
   - Was it successful?
   - Any errors during sync?

4. **Git Source Information**
   - Repository URL
   - Target revision (branch/tag/commit)
   - Path within repository

## Common Issues Detected

### OutOfSync Status
- Manual changes made to cluster (kubectl apply)
- Git repository updated but app hasn't synced
- Auto-sync disabled and manual sync needed
- Helm parameter overrides causing drift

### Degraded Health
- Pods in CrashLoopBackOff
- Deployment stuck in rollout
- Resource quotas exceeded
- Liveness/readiness probe failures

### Sync Failures
- Invalid Kubernetes manifests in Git
- Missing secrets or configmaps
- RBAC permission issues
- Helm chart rendering errors

## Tips

- Use this command when your deployment is stuck or out of sync
- If OutOfSync, check recent Git commits for issues
- If Degraded, follow up with `/check-deployment` to investigate pod-level issues
- Compare the "target state" (Git) vs "live state" (cluster)

## Follow-up Commands

After checking ArgoCD app status, you might need:

```bash
# Sync the application manually
/sync-argocd-app my-payment-service

# View detailed application resources
/argocd-app-resources my-payment-service

# Check the underlying pods
/check-deployment production my-payment-service
```

## Related

- `/sync-argocd-app` - Manually trigger a sync
- `/argocd-app-diff` - Show differences between Git and cluster
- `/argocd-app-history` - View deployment history
