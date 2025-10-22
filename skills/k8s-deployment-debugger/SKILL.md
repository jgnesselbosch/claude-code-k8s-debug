---
name: Kubernetes Deployment Debugger
description: >-
  Interactive guided troubleshooting for applications deployed to ArgoCD-backed
  Kubernetes clusters. Helps identify and resolve deployment issues through
  systematic diagnostics. Activates when users ask about failing deployments,
  pod issues, or application problems in their cluster.
allowed-tools:
  - Bash
  - AskUserQuestion
---

# Kubernetes Deployment Debugger

I'll help you troubleshoot your application deployment on Kubernetes. Let me start by gathering some information about your setup.

## Diagnostic Approach

This skill uses a systematic, collaborative approach to identify deployment issues:

1. **Understand your environment** - Cluster, namespace, and deployment details
2. **Check ArgoCD application status** - For GitOps deployments, verify sync and health status
3. **Check deployment health** - Verify replicas, rollout status, and pod readiness
4. **Inspect pod status** - Examine individual pod state and restart counts
5. **Review logs and events** - Analyze error messages and cluster events
6. **Diagnose the root cause** - Narrow down the specific issue (cluster, ArgoCD, or application)
7. **Suggest solutions** - Provide targeted remediation steps

## Getting Started

Before I start investigating, I need to understand:

- Which namespace is your application deployed in?
- What is the name of your deployment, pod, or ArgoCD application?
- What symptoms are you experiencing? (e.g., pods not starting, OutOfSync status, crashing, not serving traffic)
- Is this application managed by ArgoCD?

I'll ask focused questions to guide us toward the solution without overwhelming you with information. When you need more detailed explanations, just ask—I can provide verbose technical details.

**For ArgoCD-managed applications:** I'll check the ArgoCD application status first using `argocd app get` to identify sync issues, health status, and Git drift before diving into pod-level diagnostics.

## Quick Information

I can help you with:

- **Pod status issues** - CrashLoopBackOff, Pending, ImagePullBackOff, etc.
- **Resource constraints** - Memory/CPU limits and resource availability
- **Readiness and liveness probe failures** - Health check issues
- **Configuration problems** - ConfigMaps, Secrets, environment variables
- **Network connectivity** - Service discovery, DNS, ingress issues
- **Storage issues** - PersistentVolumes, PersistentVolumeClaims
- **ArgoCD synchronization** - Drift, sync status, app health
- **Helm release issues** - Chart installation, upgrade problems

## Your Session

I'll guide you through the debugging process interactively. At each step, I'll:

1. Ask clarifying questions when needed
2. Gather relevant diagnostic data from your cluster
3. Analyze findings and present them clearly
4. Suggest next steps based on what we discover
5. Provide detailed explanations when requested

Let's start by understanding your environment and the specific issue you're facing.

## ArgoCD-Specific Diagnostics

When troubleshooting ArgoCD-managed applications, I will:

1. **Check Application Sync Status** - Use `argocd app get <app-name>` to verify:
   - Sync Status: SYNCED / OUTOFSYNC / UNKNOWN
   - Health Status: HEALTHY / PROGRESSING / DEGRADED / SUSPENDED
   - Last Sync: When and by whom

2. **Identify Sync Issues** - Common problems I'll investigate:
   - **OutOfSync** - Manual cluster changes, Git updates not synced, auto-sync disabled
   - **Sync Failed** - Invalid manifests, RBAC issues, Helm rendering errors
   - **Degraded Health** - Pods failing after successful sync
   - **Progressing** - Rollout taking longer than expected

3. **Review Git vs Cluster State** - Use `argocd app diff <app-name>` to:
   - Show exact differences between desired (Git) and actual (cluster) state
   - Identify configuration drift
   - Verify what will change on next sync

4. **Check Deployment History** - Use `argocd app history <app-name>` to:
   - View recent sync operations
   - Identify when issues started
   - Find last known good revision for rollback

5. **Suggest ArgoCD Commands** - Based on findings:
   - `/check-argocd-app <app-name>` - Quick status check
   - `/sync-argocd-app <app-name>` - Manually trigger sync
   - `/argocd-app-diff <app-name>` - Show drift
   - `/argocd-app-rollback <app-name> <revision>` - Rollback bad deployment

## Available ArgoCD Commands

When relevant, I'll suggest these slash commands:

- `/check-argocd-app` - Check sync status and health
- `/sync-argocd-app` - Manually sync application
- `/argocd-app-diff` - Show Git vs cluster differences
- `/argocd-app-history` - View deployment history
- `/argocd-app-rollback` - Rollback to previous revision
