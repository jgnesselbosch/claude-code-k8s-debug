# ArgoCD Kubernetes Debugger

An intelligent Claude Code plugin for debugging applications deployed to GitOps-enabled Kubernetes clusters backed by ArgoCD.

## Overview

The ArgoCD Kubernetes Debugger is a Claude Code plugin that helps developers and DevOps engineers troubleshoot failing deployments. It provides:

- **Interactive Guided Diagnostics** - A focused skill that asks the right questions to systematically identify issues
- **Quick Status Checks** - Slash commands for rapid deployment and pod inspection
- **Kubernetes Integration** - Seamless access to cluster state via kubectl CLI
- **Team-Friendly Approach** - Progressive disclosure of technical details based on user needs

## Quick Start

### Installation

```bash
# Clone or navigate to this repository
cd claude-code-k8s-debug

# Install the plugin in Claude Code
claude plugins install .
```

### First Use

In Claude Code, simply ask:

```
Help me debug why my app isn't running
```

The Kubernetes Deployment Debugger skill will activate and guide you through troubleshooting.

Or use a slash command:

```
/debug-app
/check-deployment production my-app
/view-pod-logs production my-pod-xyz123
```

## Features

### Interactive Debugging Skill

The core feature is an intelligent, model-invoked skill that:

- Asks focused questions about your issue (namespace, deployment name, symptoms)
- Systematically gathers diagnostic data from your cluster
- Analyzes pod status, logs, events, and resource usage
- Identifies root causes (CrashLoops, ImagePull failures, resource constraints, probe failures, etc.)
- Suggests remediation steps
- Provides detailed explanations when requested

**Use it when**: You're experiencing deployment issues and need guided help

### Slash Commands

Quick diagnostic commands for specific tasks:

**Kubernetes Commands:**
- **`/debug-app`** - Launch interactive debugging (same as triggering the skill)
- **`/check-deployment <namespace> <name>`** - Quick status check for a deployment
- **`/view-pod-logs <namespace> <pod> [--tail N] [--previous]`** - View pod logs with filtering
- **`/check-events [namespace]`** - View cluster events and warnings
- **`/check-resources [namespace]`** - Check pod resource usage (CPU/memory)

**ArgoCD Commands:**
- **`/check-argocd-app <app-name>`** - Check ArgoCD app sync status and health
- **`/sync-argocd-app <app-name>`** - Manually trigger an ArgoCD sync
- **`/argocd-app-diff <app-name>`** - Show Git vs cluster differences
- **`/argocd-app-history <app-name>`** - View deployment history and past syncs
- **`/argocd-app-rollback <app-name> [revision]`** - Rollback to previous deployment

**Use them when**: You want a quick check or specific diagnostic without full guided debugging

## Prerequisites

- **kubectl configured** - Access to your Kubernetes cluster with appropriate permissions
- **Kubernetes 1.20+** - Cluster version
- **ArgoCD CLI** (optional) - For ArgoCD-specific troubleshooting commands
- **Metrics Server** (optional) - For resource usage metrics in `/check-resources`

### ArgoCD CLI Setup

For ArgoCD-specific features, install and configure the ArgoCD CLI:

```bash
# Install ArgoCD CLI (Linux)
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/argocd

# Install ArgoCD CLI (macOS)
brew install argocd

# Login to your ArgoCD instance
argocd login <argocd-server-url>
```

See [ARGOCD_SETUP.md](./ARGOCD_SETUP.md) for detailed setup instructions.

### Required Permissions

The plugin needs these permissions:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: claude-debugger
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "namespaces", "events", "services"]
  verbs: ["list", "get", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets", "daemonsets"]
  verbs: ["list", "get"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["list", "get"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["pods"]
  verbs: ["get", "list"]
```

## Supported Scenarios

The plugin helps troubleshoot:

- **Pod Status Issues**
  - CrashLoopBackOff
  - ImagePullBackOff / ImagePullError
  - Pending (stuck scheduling)
  - Terminating (stuck deletion)

- **Resource Constraints**
  - Out of Memory (OOMKilled)
  - CPU throttling
  - Insufficient resources for scheduling
  - Request/limit misconfigurations

- **Health & Readiness**
  - Liveness probe failures
  - Readiness probe failures
  - Startup probe failures

- **Configuration Issues**
  - ConfigMap/Secret mounting
  - Environment variable problems
  - Volume mount issues

- **Network Connectivity**
  - Service discovery problems
  - DNS resolution issues
  - Ingress/LoadBalancer issues

- **Storage Issues**
  - PersistentVolume mounting
  - PersistentVolumeClaim binding
  - Storage provisioning

- **ArgoCD Integration**
  - Sync status and drift
  - Application health
  - Git source issues

## Architecture

### Components

```
.claude-plugin/
├── plugin.json                    # Plugin metadata and capabilities

.claude/
├── skills/
│   └── k8s-deployment-debugger.md # Interactive debugging skill
│       └── Allowed tools: Bash, AskUserQuestion
└── commands/
    ├── debug-app.md               # /debug-app command
    ├── check-deployment.md        # /check-deployment command
    ├── view-pod-logs.md           # /view-pod-logs command
    ├── check-events.md            # /check-events command
    └── check-resources.md         # /check-resources command
```

### How It Works

1. **User Request** → "Help me debug my failing pod"
2. **Skill Activation** → The debugger skill activates automatically
3. **Interactive Questioning** → Asks for namespace, deployment name, symptoms
4. **Data Collection** → Uses kubectl commands to query cluster state
5. **Analysis** → Examines logs, events, resource usage, pod status
6. **Diagnosis** → Identifies root cause
7. **Guidance** → Suggests fixes and explains solutions
8. **Follow-up** → Asks if they need more details or help with next steps

## kubectl Integration

The plugin uses **kubectl CLI** for all Kubernetes operations:

- **Simple** - No additional dependencies beyond kubectl
- **Direct** - Uses standard kubectl commands you already know
- **Flexible** - Respects your KUBECONFIG and context settings
- **Powerful** - Full access to cluster resources

**Common kubectl commands used**:
- `kubectl get pods/deployments/events`
- `kubectl describe pod/deployment`
- `kubectl logs <pod>`
- `kubectl top pods` (requires Metrics Server)

The plugin works with your existing kubectl configuration and cluster contexts.

## Usage Examples

### Example 1: Debug a Failing Deployment

```
User: My production API service won't start. Can you help?


## Cost Tracking

This plugin includes built-in cost tracking to help you monitor your Claude usage.

### Check Your Costs
```bash
# In Claude Code, simply type:
/costs
```

This will show you:
- Token usage for today
- Estimated cost in USD
- Tips to optimize usage

### Cost-Saving Tips

1. **Use `/clear` regularly** - Clears conversation context between debugging tasks
2. **Be specific** - "Debug nginx deployment in production namespace" is better than "help me debug"
3. **Use the diagnostic scripts** - They don't consume tokens:
```bash
   python scripts/check_deployment.py my-deploy production
```
4. **Close large log outputs** - After reviewing logs, clear them from context

### Understanding Costs

- **Pro Plan ($17/month)**: ~500K tokens/day typical usage
- **Max Plan ($100/month)**: ~2M tokens/day typical usage  
- **API Pay-as-you-go**: ~$3 per 1M input tokens

Your `/costs` command shows real usage so you can stay within your limits.