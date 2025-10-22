# ArgoCD Features Summary

This document summarizes the ArgoCD-specific capabilities added to the plugin.

## What Was Added

### 5 New ArgoCD Slash Commands

1. **`/check-argocd-app <app-name>`** (`.claude/commands/check-argocd-app.md`)
   - Check sync status (SYNCED, OUTOFSYNC, UNKNOWN)
   - Check health status (HEALTHY, PROGRESSING, DEGRADED, SUSPENDED)
   - View recent sync events
   - Show Git source information
   - Identify common issues (drift, sync failures, degraded health)

2. **`/sync-argocd-app <app-name>`** (`.claude/commands/sync-argocd-app.md`)
   - Manually trigger an ArgoCD sync
   - Options: `--prune`, `--force`, `--dry-run`
   - Reconcile cluster state with Git
   - Use for manual syncs, drift correction, troubleshooting

3. **`/argocd-app-diff <app-name>`** (`.claude/commands/argocd-app-diff.md`)
   - Show differences between Git (desired) and cluster (actual)
   - Identify drift, additions, removals, modifications
   - Preview changes before syncing
   - Option: `--local <path>` to compare with local changes

4. **`/argocd-app-history <app-name>`** (`.claude/commands/argocd-app-history.md`)
   - View deployment history
   - Show past sync operations with timestamps
   - Identify when issues started
   - Find last known good revision for rollback
   - Option: `--limit <n>` to control number of entries

5. **`/argocd-app-rollback <app-name> [revision]`** (`.claude/commands/argocd-app-rollback.md`)
   - Rollback to previous deployment
   - Quick incident response for bad deployments
   - Option: `--prune` to remove resources added in newer versions
   - Important: Consider auto-sync behavior and database migrations

### Enhanced Skill with ArgoCD Diagnostics

The main debugging skill (`.claude/skills/k8s-deployment-debugger.md`) now:

- **Asks if the app is managed by ArgoCD** during initial questioning
- **Checks ArgoCD status first** for GitOps deployments before pod-level diagnostics
- **Uses `argocd` CLI** via the Bash tool to run commands like:
  - `argocd app get <app-name>`
  - `argocd app diff <app-name>`
  - `argocd app history <app-name>`
- **Identifies ArgoCD-specific issues:**
  - OutOfSync status (drift, Git updates, manual changes)
  - Sync failures (invalid manifests, RBAC, Helm errors)
  - Degraded health (pods failing after successful sync)
  - Progressing state (slow rollouts)
- **Suggests appropriate ArgoCD commands** based on findings

### Setup Documentation

**ARGOCD_SETUP.md** - Comprehensive guide covering:
- Installing ArgoCD CLI on Linux/macOS/Windows
- Logging in to ArgoCD (CLI, token, port-forward)
- Managing multiple ArgoCD contexts
- Required permissions for debugging
- Troubleshooting CLI access issues
- Environment configuration for teams
- Best practices (read-only access, token rotation, context awareness)

## How It Works

### Workflow for ArgoCD-Managed Apps

```
User: "My payment-service ArgoCD app won't deploy"
  ↓
Skill activates
  ↓
Asks: "What namespace? Is it managed by ArgoCD?"
  ↓
User: "Yes, it's an ArgoCD app"
  ↓
Skill runs: argocd app get payment-service
  ↓
Finds: Status=OutOfSync, Health=Degraded
  ↓
Skill runs: argocd app diff payment-service
  ↓
Shows: Image tag changed from v2.0 to v2.1
  ↓
Skill suggests: "/sync-argocd-app payment-service"
  ↓
After sync, checks pod status with kubectl/MCP
  ↓
Identifies: CrashLoopBackOff due to missing env var
  ↓
Suggests: Update ConfigMap in Git and re-sync
```

### Integration Points

The plugin now addresses **both layers** of GitOps deployments:

**Layer 1: ArgoCD (Git → Cluster Sync)**
- Is the app in sync with Git?
- Did the sync succeed?
- What changed between Git and cluster?
- When was the last successful deployment?

**Layer 2: Kubernetes (Cluster Resources)**
- Are the pods running?
- What do the logs show?
- Are there resource constraints?
- Are probes passing?

## Common Scenarios Supported

### 1. OutOfSync Drift
```bash
/check-argocd-app my-app
# Shows: OutOfSync

/argocd-app-diff my-app
# Shows: Manual kubectl changes

/sync-argocd-app my-app --prune
# Corrects drift
```

### 2. Failed Deployment Rollback
```bash
/check-argocd-app my-app
# Shows: Synced but Degraded

/argocd-app-history my-app
# Find last healthy revision

/argocd-app-rollback my-app 14
# Restore service
```

### 3. Pre-Sync Verification
```bash
/argocd-app-diff my-app --dry-run
# Preview changes

# Review output, then:
/sync-argocd-app my-app
```

### 4. Investigating Sync Failures
```bash
/argocd-app-history my-app
# Shows: Last sync failed

/argocd-app-diff my-app
# Identify problematic manifests

# Fix in Git, then:
/sync-argocd-app my-app
```

## Prerequisites

### Required
- **ArgoCD CLI installed** (`argocd` command in PATH)
- **ArgoCD login configured** (`argocd login <server>`)
- **Network access** to ArgoCD server

### Permissions Needed

**Read-Only (Debugging):**
```yaml
applications:
  - get
  - list
  - diff
  - history
```

**Write (Sync/Rollback):**
```yaml
applications:
  - sync
  - rollback
```

## Technical Implementation

### How Commands Execute

The skill has access to the `Bash` tool, which allows it to run:

```bash
# Check status
argocd app get my-app -o json

# Show diff
argocd app diff my-app

# Sync
argocd app sync my-app --prune

# Rollback
argocd app rollback my-app 14
```

### No MCP Server for ArgoCD

Unlike Kubernetes (which uses `kubernetes-mcp-server`), ArgoCD uses the **direct CLI approach** via the Bash tool because:
- ArgoCD CLI is mature and feature-complete
- No official ArgoCD MCP server exists yet
- CLI provides all needed functionality
- Simpler setup than creating custom MCP server

## Benefits

### For Developers
✅ Understand why their deployment failed at the GitOps level
✅ See exactly what changed between Git and cluster
✅ Quickly rollback bad deployments
✅ Verify sync status before investigating pods

### For DevOps/SRE
✅ Faster incident response with guided rollbacks
✅ Identify configuration drift automatically
✅ Audit deployment history for investigations
✅ Combine ArgoCD + Kubernetes diagnostics in one tool

### For Teams
✅ Junior developers get guided troubleshooting
✅ Consistent debugging workflow across team
✅ Reduced time spent on deployment issues
✅ Better understanding of GitOps failure modes

## Next Steps

1. **Install ArgoCD CLI** - See ARGOCD_SETUP.md
2. **Login to ArgoCD** - Configure access to your instance
3. **Try the commands** - Test with a non-production app
4. **Use in debugging** - Ask Claude about ArgoCD-managed deployments

## Examples

### Example 1: Natural Language
```
User: My frontend ArgoCD app shows OutOfSync, what's wrong?

Claude: Let me check your ArgoCD application status...
[Runs: argocd app get frontend]

I see the issue! Your frontend app is OutOfSync because:
- Someone manually scaled the deployment from 3 to 5 replicas in the cluster
- But Git still shows 3 replicas
- ArgoCD detected this drift

Let me show you the exact differences...
[Runs: argocd app diff frontend]

To fix this, you can either:
1. Sync to restore Git's 3 replicas: /sync-argocd-app frontend --prune
2. Or update Git to match the cluster's 5 replicas

Which would you like to do?
```

### Example 2: Slash Command
```
User: /check-argocd-app payment-api

Claude:
ArgoCD Application: payment-api
├── Sync Status: OutOfSync
├── Health Status: Healthy
├── Last Sync: 2025-01-20 14:32:11 by john@company.com
└── Git Source: https://github.com/company/k8s-config.git (main)

Issues detected:
- ConfigMap 'payment-config' was modified in cluster but not in Git
- Image tag updated in Git but not synced to cluster

Suggested actions:
- Run /argocd-app-diff payment-api to see exact differences
- Run /sync-argocd-app payment-api to sync to Git state
```

## Files Added

```
.claude/commands/
├── check-argocd-app.md        # New
├── sync-argocd-app.md         # New
├── argocd-app-diff.md         # New
├── argocd-app-history.md      # New
└── argocd-app-rollback.md     # New

.claude/skills/
└── k8s-deployment-debugger.md # Enhanced with ArgoCD support

ARGOCD_SETUP.md                # New setup guide
ARGOCD_FEATURES.md             # This file
```

## Comparison: Before vs After

### Before
- ❌ Only Kubernetes-level diagnostics
- ❌ No GitOps sync status visibility
- ❌ No drift detection
- ❌ No deployment history/rollback
- ❌ Couldn't identify ArgoCD vs pod issues

### After
- ✅ Full ArgoCD + Kubernetes diagnostics
- ✅ Sync and health status checking
- ✅ Git vs cluster drift detection
- ✅ Deployment history and easy rollback
- ✅ Distinguishes GitOps layer from pod layer
- ✅ Guides users through ArgoCD issues
- ✅ Suggests appropriate fixes at each layer
