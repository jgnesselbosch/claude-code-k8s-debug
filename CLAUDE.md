# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🎯 Plugin Usage Philosophy

This plugin and its Kubernetes Deployment Debugger skill should be your **default approach** for ANY Kubernetes-related issues. The philosophy is:

> **"When the user has ANY trouble with anything on their Kubernetes cluster, the skill should be invoked."**

This means:
- ✅ **Default to using the skill** - It's designed to handle all cluster debugging scenarios
- ✅ **Proactively offer help** - Don't make the user ask; suggest the skill when you detect cluster issues
- ✅ **Trust the systematic approach** - The skill's guided diagnostics are more reliable than ad-hoc debugging
- ✅ **Log everything** - Every change is documented for future reference and GitOps compliance

---

## ⚠️ CRITICAL: Mandatory Cluster Change Logging

**EVERY cluster change MUST have a corresponding entry in `k8s-debug-changes.log`.**

When you complete any debugging task that modifies the cluster:
1. ✅ Make the cluster change
2. ✅ Create an entry in `k8s-debug-changes.log` using the standardized template
3. ✅ Only then mark the task as complete

**Failure to log changes results in lost debugging history and potential GitOps conflicts.**

See the [k8s-debug-changes.log Entry Format](#k8s-debug-changeslog-entry-format) section below for the required template.

---

## Project Overview

This repository contains the **ArgoCD Kubernetes Debugger** plugin for Claude Code. It's an intelligent debugging assistant for applications deployed to GitOps-enabled Kubernetes clusters backed by ArgoCD. The plugin helps developers and DevOps engineers troubleshoot failing deployments through interactive guided diagnostics.

## When to Use This Plugin (ACTIVATION CRITERIA)

**Use the "Kubernetes Deployment Debugger" skill and plugin commands for ANY of the following scenarios:**

### User Reports Issues With:
- Kubernetes deployments, pods, or services not working
- Applications deployed to the cluster that are failing
- Pods crashing, failing to start, or behaving unexpectedly
- Phrases like: "my app won't start", "deployment is broken", "pods are stuck", "cluster is broken"
- Any error related to running applications in the cluster
- Resource allocation problems (out of memory, CPU limits)
- Network connectivity or DNS issues in the cluster
- Storage or persistent volume issues
- Container image pull failures or startup errors

### Activation Strategy (PROACTIVE - NOT REACTIVE)

**Do NOT wait for an explicit request to use the skill.** Instead:

1. **Detect cluster-related issues** - When user mentions any Kubernetes/deployment/pod problem
2. **Offer to help** - Clearly state: "I can help debug this using the Kubernetes Deployment Debugger"
3. **Invoke the skill** - Launch the skill immediately (either after brief diagnosis or directly if issue is clear)
4. **Guide systematically** - Use the skill to walk through diagnostics step-by-step
5. **Document all changes** - Log any cluster modifications to `k8s-debug-changes.log`

### What NOT to Do

- ❌ Do NOT assume a non-Kubernetes issue without checking
- ❌ Do NOT skip the skill and try to debug manually with kubectl commands
- ❌ Do NOT require explicit user permission to invoke the skill
- ❌ Do NOT offer other diagnostic approaches when a cluster issue is detected

---

## Plugin Architecture

### Directory Structure

```
.claude-plugin/                      # Plugin metadata
├── plugin.json                      # Plugin manifest
├── marketplace.json                 # Plugin marketplace configuration
commands/                            # Slash commands (at root)
├── debug-app.md                     # Main debugging command
├── check-deployment.md              # Quick deployment status
├── view-pod-logs.md                 # Pod log viewing
├── check-events.md                  # Cluster events viewer
├── check-resources.md               # Resource usage checker
└── [ArgoCD commands...]             # ArgoCD-specific commands
skills/                              # Claude Code Skills (at root)
└── k8s-deployment-debugger/
    └── SKILL.md                     # Interactive debugging skill
docs/                                # Documentation
├── ARGOCD_FEATURES.md               # ArgoCD integration details
├── ARGOCD_SETUP.md                  # Setup instructions
└── GETTING_STARTED.md               # Getting started guide
.claude/
└── settings.local.json              # Local settings for development
```

### Core Components

1. **Interactive Skill** (`skills/k8s-deployment-debugger/SKILL.md`)
   - Claude-invoked skill that guides debugging through systematic diagnostics
   - Asks focused questions to narrow down issues
   - Uses kubernetes kubectl CLI for cluster introspection
   - Provides verbose explanations when requested
   - Covers: pod status, logs, events, resources, probes, configuration, storage, ArgoCD sync

2. **Slash Commands** (`commands/`)

   **Kubernetes Commands:**
   - `/debug-app` - Launch interactive debugging session
   - `/check-deployment` - Quick deployment status check
   - `/view-pod-logs` - View pod logs with filtering
   - `/check-events` - View cluster events
   - `/check-resources` - Check resource usage

   **ArgoCD Commands:**
   - `/check-argocd-app` - Check ArgoCD app sync/health status
   - `/sync-argocd-app` - Manually trigger sync
   - `/argocd-app-diff` - Show Git vs cluster differences
   - `/argocd-app-history` - View deployment history
   - `/argocd-app-rollback` - Rollback to previous revision

3. **kubectl CLI Integration**
   - Uses `kubectl` commands via Bash for all Kubernetes operations
   - No additional MCP server dependencies required
   - Direct and simple integration with standard Kubernetes tooling

## Installation and Setup

### Prerequisites

1. **kubectl configured** - With access to your Kubernetes cluster
2. **Kubernetes cluster** - With ArgoCD or another deployment mechanism
3. **ArgoCD CLI** (optional) - For ArgoCD-specific troubleshooting (`argocd` command)
4. **Metrics Server** (optional) - For resource usage checking

### Installation Steps

1. Install the plugin in Claude Code:
   ```bash
   claude plugins install ./path/to/claude-code-k8s-debug
   ```

2. Ensure `kubectl` is configured and can access your cluster:
   ```bash
   kubectl cluster-info
   kubectl get nodes
   ```

3. Optional: Configure a specific kubeconfig if not using default:
   ```bash
   export KUBECONFIG=/path/to/your/kubeconfig
   ```

## Development Commands

### Testing the Skill

When developing, trigger the skill in Claude Code with requests like:
```
Help me debug why my app isn't running
What's wrong with my deployment?
My pods keep crashing, can you help?
```

### Testing Commands

Try the Kubernetes slash commands:
```
/debug-app
/check-deployment production my-app
/view-pod-logs production my-app-xyz123
/check-events production
/check-resources production
```

Try the ArgoCD slash commands (requires `argocd` CLI installed and logged in):
```
/check-argocd-app my-app
/argocd-app-diff my-app
/argocd-app-history my-app
/sync-argocd-app my-app
```

## Key Architecture Decisions

### Skill Design
- **Model-invoked** (not user-invoked) to activate automatically when relevant
- **Focused interaction** - Asks questions rather than overwhelming with information
- **Progressive disclosure** - Detailed explanations available on request
- **Collaborative approach** - Guides users through systematic problem-solving

### kubectl Integration
- Uses **kubectl CLI** via Bash for all Kubernetes operations
- Respects `KUBECONFIG` environment variable for cluster selection
- No additional dependencies beyond standard Kubernetes tooling
- Simple and direct access to cluster resources

### Command Design
- **Modular** - Each command focuses on one diagnostic aspect
- **Parameterized** - Accept namespace and resource names for specificity
- **Documented** - Clear examples and explanations for users

## Common Development Scenarios

### Adding a New Diagnostic Capability

1. Create a new command file in `commands/` with clear examples
2. Document the kubectl commands and expected usage patterns
3. Test with your cluster to ensure proper output formatting

### Enhancing the Skill

The skill in `skills/k8s-deployment-debugger/SKILL.md` should:
- Guide users through problems step-by-step
- Ask clarifying questions using AskUserQuestion
- Use kubectl commands via Bash to gather cluster state
- Present findings clearly and suggest next steps
- Reference slash commands when appropriate

### Testing Cluster Access

```bash
# Verify kubectl is installed and configured
kubectl version --client
kubectl cluster-info
kubectl get nodes

# Test access to your cluster
kubectl get namespaces
kubectl get pods --all-namespaces
```

## Integration Points

- **ArgoCD**: The skill can help diagnose ArgoCD sync issues, app health, and deployment status
- **GitOps**: Provides debugging for git-reconciled workloads
- **kubectl contexts**: Supports multi-cluster operations via kubectl context switching

## Testing Considerations

When testing the plugin:
- Ensure kubectl access works: `kubectl auth can-i list pods --all-namespaces`
- Test with various deployment failure scenarios (CrashLoop, ImagePull, Resource limits, etc.)
- Verify logs are accessible for your pods
- Confirm metrics server is available if testing `/check-resources`
- Test against both healthy and failing deployments

## Requirements while changing objects in the cluster

When making changes to objects in the Kubernetes cluster during debugging sessions, ensure that:

### GitOps Compliance (MANDATORY)
 - **NEVER** make changes imperatively using kubectl without ensuring they're reflected in Git
 - **ALWAYS** ensure that every cluster change is reflected in the Git repository that ArgoCD is syncing from
 - This prevents changes from being overwritten during the next ArgoCD sync operation
 - Always follow best practices for GitOps workflows to maintain cluster state consistency

### Temporary Changes
 - If you need temporary changes for debugging purposes, document them clearly
 - Revert temporary changes in the Git repository once the debugging session is complete
 - Temporary cluster-only changes will be lost on next ArgoCD sync - this is by design

### User Authorization
 - You are allowed to deviate from these requirements **ONLY IF** the user explicitly instructs you to do so
 - Ask the user proactively if you are unsure about their intent
 - When suggesting changes to the user, **always** remind them to update their Git repository accordingly

### Mandatory Logging (CRITICAL - DO NOT SKIP)
 - **EVERY change made to cluster objects MUST be logged**
 - Write changes in a dedicated section at the end of your response called "Changes made to the cluster"
 - **IMMEDIATELY** write an entry to `k8s-debug-changes.log` in the current working directory following the standardized template
 - This file should NOT be pushed to any Git repository
 - **Do not consider a debugging session complete until the log entry is written**

## k8s-debug-changes.log Entry Format

All entries in the `k8s-debug-changes.log` file **MUST** follow this standardized template to maintain consistency and clarity across debugging sessions.

### Pre-Completion Checklist

**Before marking any cluster-change task as complete, verify:**

- [ ] Cluster change has been successfully applied
- [ ] Change has been verified to work as expected
- [ ] Entry created in `k8s-debug-changes.log`
- [ ] Entry follows the template format exactly
- [ ] All required sections are completed (Changes Made, Issue Resolved, etc.)
- [ ] GitOps implications have been documented
- [ ] User has been informed about updating Git repository

**Do NOT consider the task complete without ALL checkboxes passing.**

### Template Structure

```markdown
## Session: [Brief description of what was fixed/changed]

### Changes Made

1. **[Resource Type] - [Resource Name]**
   - Namespace: [namespace name]
   - Resource Type: [e.g., Deployment, StatefulSet, Pod, ConfigMap, etc.]
   - Component: [specific component being changed, if applicable]
   - Change: [Concise description of what changed from → to]
   - Method: [kubectl command or tool used, e.g., `kubectl set image deployment/...`]
   - Reason: [Why this change was necessary - root cause analysis]
   - Status: [Current status after change]
   - Timestamp: [Date and time of change in ISO format]

### Issue Resolved

- **Problem**: [What was broken or not working]
- **Root Cause**: [Technical root cause analysis]
- **Solution**: [How the issue was fixed]
- **Result**: [Actual outcome of the fix]
- **Old State**: [Previous pod/resource name/status if applicable]
- **New State**: [Current pod/resource name/status if applicable]

### Important Notes

[Any critical information about the change, caveats, or warnings]

### Follow-up Actions Required

[Actions user must take, such as updating Git repository, reverting changes, etc.]
```

### Usage Guidelines

- **One session per heading**: Each debugging session gets one `## Session:` header
- **Multiple resources**: If multiple resources were changed in one session, list each under "Changes Made" with separate numbered items
- **Timestamps**: Use ISO 8601 format: `YYYY-MM-DD HH:MM:SS TIMEZONE`
- **GitOps reminder**: Always include a note about updating the Git repository for imperative changes
- **Root cause clarity**: Be specific about *why* the issue occurred, not just *what* was changed
- **Status updates**: Document both the problem state and the fixed state clearly

### Example Entry

```markdown
## Session: nginx-foul Deployment ImagePullBackOff Fix

### Changes Made

1. **Deployment - nginx-foul**
   - Namespace: default
   - Resource Type: Deployment
   - Component: Container image specification
   - Change: Updated image from `nginx:9.9.9` → `nginx:latest`
   - Method: `kubectl set image deployment/nginx-foul nginx=nginx:latest -n default`
   - Reason: The tag `nginx:9.9.9` does not exist in Docker registry; blocking pod startup
   - Status: Successfully applied; pods now running
   - Timestamp: 2025-10-22 16:19:05 UTC

### Issue Resolved

- **Problem**: Pod stuck in ImagePullBackOff state, unable to start
- **Root Cause**: Invalid/non-existent Docker image tag (`nginx:9.9.9`) specified in deployment manifest
- **Solution**: Updated to valid, stable nginx image tag (`nginx:latest`)
- **Result**: Deployment healthy with all replicas running
- **Old State**: Pod `nginx-foul-74d8b5c584-d2ptw` (failed, deleted)
- **New State**: Pod `nginx-foul-85478887d-x9hp4` (running, healthy)

### Important Notes

This was an imperative hotfix using kubectl. The change will be lost on next ArgoCD sync unless the source manifest in Git is updated.

### Follow-up Actions Required

1. Update deployment manifest in Git repository with corrected image tag
2. Commit and push changes to trigger ArgoCD sync
3. Verify new pods are deployed from Git-synced manifests
```