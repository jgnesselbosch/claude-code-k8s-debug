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
   - Uses kubernetes MCP server for cluster introspection
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

3. **MCP Server Integration** (`.mcp.json`)
   - Configured to use kubernetes-mcp-server via stdio transport
   - Provides programmatic access to Kubernetes API
   - Allows listing/getting/reading pods, deployments, logs, events, metrics, and more

## Installation and Setup

### Prerequisites

1. **Node.js and npm** - For running the kubernetes-mcp-server
2. **kubectl configured** - With access to your Kubernetes cluster
3. **Kubernetes cluster** - With ArgoCD or another deployment mechanism
4. **ArgoCD CLI** (optional) - For ArgoCD-specific troubleshooting (`argocd` command)
5. **Metrics Server** (optional) - For resource usage checking

### Installation Steps

1. Install the plugin in Claude Code:
   ```bash
   claude plugins install ./path/to/claude-code-k8s-debug
   ```

2. The kubernetes-mcp-server will be automatically available when the plugin loads

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

### MCP Server Integration
- Uses **stdio transport** with npx for easy dependency management
- Respects `KUBECONFIG` environment variable for cluster selection
- Access scoped through allowed-tools in skill definition
- Enables dynamic cluster introspection vs. static scripts

### Command Design
- **Modular** - Each command focuses on one diagnostic aspect
- **Parameterized** - Accept namespace and resource names for specificity
- **Documented** - Clear examples and explanations for users

## Common Development Scenarios

### Adding a New Diagnostic Capability

1. Create a new command file in `.claude/commands/` with clear examples
2. Add the allowed tool(s) to the skill's `allowed-tools` list in `.claude/skills/`
3. Update `.mcp.json` if new MCP server interaction types are needed

### Enhancing the Skill

The skill in `.claude/skills/k8s-deployment-debugger.md` should:
- Guide users through problems step-by-step
- Ask clarifying questions using AskUserQuestion
- Use MCP tools to gather cluster state
- Present findings clearly and suggest next steps
- Reference slash commands when appropriate

### Testing Cluster Access

```bash
# Verify kubernetes-mcp-server is installed
npx @containerized/kubernetes-mcp-server --version

# Test kubectl access
kubectl cluster-info
kubectl get nodes
```

## MCP Server Details

The plugin uses the kubernetes-mcp-server with these capabilities:

**Pod Operations**: List, get, delete, logs, exec, run, top (metrics)
**Resource Management**: Create/update/delete any K8s resource (CRUD)
**Cluster Info**: View kubeconfig, namespaces, events, contexts
**Helm Integration**: Install, list, uninstall releases

See https://github.com/containers/kubernetes-mcp-server for full documentation.

## Integration Points

- **ArgoCD**: The skill can help diagnose ArgoCD sync issues, app health, and deployment status
- **GitOps**: Provides debugging for git-reconciled workloads
- **Multi-cluster**: The kubernetes-mcp-server supports multi-cluster operations (future enhancement)

## Testing Considerations

When testing the plugin:
- Ensure kubectl access works: `kubectl auth can-i list pods --all-namespaces`
- Test with various deployment failure scenarios (CrashLoop, ImagePull, Resource limits, etc.)
- Verify logs are accessible for your pods
- Confirm metrics server is available if testing `/check-resources`
- Test against both healthy and failing deployments
