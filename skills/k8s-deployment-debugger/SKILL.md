---
name: Kubernetes Deployment Debugger
description: >-
  Interactive guided troubleshooting for applications deployed to ArgoCD-backed
  Kubernetes clusters. Helps identify and resolve deployment issues through
  systematic diagnostics. Activates for ANY cluster-related issues including pod
  failures, deployment problems, image pull errors, resource issues, networking
  problems, storage issues, and ArgoCD sync problems. Always invoke this skill
  when users mention any Kubernetes deployment or application trouble.
allowed-tools:
  - Bash
  - AskUserQuestion
---

# Kubernetes Deployment Debugger

**I'm here to fix your Kubernetes issues.** Let me diagnose what's wrong and get your application running.

## ⚡ What I Can Fix Right Now

I can help with these common problems:
- 🔴 **Pods won't start** (CrashLoopBackOff, ImagePullBackOff, Pending)
- 🔴 **Apps crashing or restarting** constantly
- 🔴 **Image pull failures** - Container registry issues
- 🔴 **Out of resources** - Memory/CPU limits exceeded
- 🔴 **Health check failures** - Readiness/liveness probes
- 🔴 **Configuration errors** - ConfigMaps, Secrets, env vars
- 🔴 **Network issues** - Services, DNS, connectivity
- 🔴 **Storage problems** - PersistentVolumes, claims
- 🔴 **ArgoCD sync problems** - OutOfSync, drift, failed syncs
- 🔴 **Helm deployment issues** - Chart rendering, upgrades

**Ready? Tell me what's broken and I'll fix it.**

## How I Work

My approach is **fast and systematic**:

1. **Ask what's wrong** - Get the specifics from you
2. **Gather cluster data** - Run diagnostics automatically
3. **Identify root cause** - Analyze the findings
4. **Fix it** - Suggest or apply targeted solutions
5. **Verify** - Confirm everything is working

You don't need to know kubectl - I handle the technical details. You just describe the problem.

## Quick Start: Tell Me Your Issue

**What's happening?** Describe your problem in your own words. For example:

- *"My nginx deployment won't start"*
- *"Pods keep crashing with CrashLoopBackOff"*
- *"Image pull is failing, can't download container"*
- *"My app is deployed but pods are in Pending state"*
- *"ArgoCD says OutOfSync and I don't know why"*
- *"Resources are maxed out, pods won't schedule"*
- *"My deployment is healthy but app isn't responding"*

**Or just tell me:**
- Namespace name (if not default)
- Deployment or pod name
- What error message you're seeing

## 🎯 I'm Triggered By These Keywords

If you mention any of these, I should automatically be invoked:

**Deployment Issues:** deployment, pod, service, replica, rollout, image pull, CrashLoop, ImagePullBackOff, Pending, Terminating, Error, Failed, not running, won't start, stuck, broken

**ArgoCD Issues:** OutOfSync, sync failed, drift, ArgoCD, ApplicationSet, health status, degraded

**Resource Issues:** out of memory, OOM, CPU limit, resources, scheduling, pending

**Application Issues:** app crashing, app not responding, container error, logs, event, debug, troubleshoot

**Generic Cluster Issues:** "what's wrong", "help me debug", "broken deployment", "cluster issue"

---

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
