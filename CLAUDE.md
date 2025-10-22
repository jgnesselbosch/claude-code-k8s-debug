# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the **ArgoCD Kubernetes Debugger** plugin for Claude Code. It's an intelligent debugging assistant for applications deployed to GitOps-enabled Kubernetes clusters backed by ArgoCD. The plugin helps developers and DevOps engineers troubleshoot failing deployments through interactive guided diagnostics.

## Plugin Architecture

### Directory Structure

```
.claude-plugin/              # Plugin metadata
├── plugin.json              # Plugin manifest
.claude/
├── skills/                  # Claude Code Skills
│   └── k8s-deployment-debugger.md  # Interactive debugging skill
└── commands/                # Slash commands
    ├── debug-app.md         # Main debugging command
    ├── check-deployment.md  # Quick deployment status
    ├── view-pod-logs.md     # Pod log viewing
    ├── check-events.md      # Cluster events viewer
    └── check-resources.md   # Resource usage checker
.mcp.json                    # MCP server configuration
```

### Core Components

1. **Interactive Skill** (`.claude/skills/k8s-deployment-debugger.md`)
   - Claude-invoked skill that guides debugging through systematic diagnostics
   - Asks focused questions to narrow down issues
   - Uses kubernetes kubectl CLI for cluster introspection
   - Provides verbose explanations when requested
   - Covers: pod status, logs, events, resources, probes, configuration, storage, ArgoCD sync

2. **Slash Commands** (`.claude/commands/`)

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

1. Create a new command file in `.claude/commands/` with clear examples
2. Document the kubectl commands and expected usage patterns
3. Test with your cluster to ensure proper output formatting

### Enhancing the Skill

The skill in `.claude/skills/k8s-deployment-debugger.md` should:
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
 - Do not just make changes imperatively by using the kubectl command line tool.
 - Instead, make sure that every change is reflected in the Git repository that ArgoCD is syncing from. This ensures that the changes are not overwritten during the next sync operation by ArgoCD.
 - If you need to make temporary changes for debugging purposes, document them clearly and revert them back in the Git repository once the debugging session is complete.
 - Always follow best practices for GitOps workflows to maintain cluster state consistency.
 - You are allowed to act not in accordance with these requirements only if the user explicitly instructs you to do so. Ask the user proactively if you are unsure.
 - When suggesting changes to the user, remind them to update their Git repository accordingly.
 - Write any changes you made to cluster objects in a dedictated section at the end of your response called "Changes made to the cluster" and also write them into a log file called `k8s-debug-changes.log` in the current working directory. This file should not be pushed to any Git repository though