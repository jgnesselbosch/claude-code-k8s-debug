# Getting Started with ArgoCD Kubernetes Debugger

Welcome! This guide will help you start using the ArgoCD Kubernetes Debugger plugin in your daily development workflow.

## What This Plugin Does For You

As a developer working with Kubernetes, you've probably encountered frustrating situations:
- Your pod won't start and you're not sure why
- The deployment is stuck and you need to check logs across multiple pods
- Something worked locally but fails in the cluster
- You're getting cryptic error messages and need help interpreting them

This plugin acts as your **AI debugging assistant** that:
- Asks you the right questions to narrow down the problem
- Automatically checks your cluster state (pods, logs, events, resources)
- Explains what's wrong in plain language
- Suggests concrete fixes

## Installation

### Step 1: Install the Plugin

```bash
# If you have the plugin repository locally
cd /path/to/claude-code-k8s-debug
claude plugins install .

# Or install directly from git (if published)
claude plugins install git+https://github.com/jgnesselbosch/claude-code-k8s-debug.git
```

### Step 2: Verify kubectl Access

The plugin needs `kubectl` configured to access your cluster:

```bash
# Check your current context
kubectl config current-context

# List available contexts
kubectl config get-contexts

# Switch to your cluster (if needed)
kubectl config use-context <your-cluster-context>

# Verify access
kubectl get pods --all-namespaces
```

### Step 3: Install kubernetes-mcp-server (Automatic)

The plugin uses `@containerized/kubernetes-mcp-server` which will be automatically installed via `npx` when you first use it. No manual installation needed!

## Daily Usage Scenarios

### Scenario 1: Your Pod Won't Start

You deploy your application and it's not coming up. Here's how to get help:

**In Claude Code, just ask naturally:**

```
My payment-service pod in the production namespace won't start. Can you help?
```

**What happens next:**
1. The Kubernetes Debugger skill activates automatically
2. Claude asks clarifying questions (if needed):
   - "What namespace?" → `production`
   - "What's the deployment or pod name?" → `payment-service`
   - "What symptoms are you seeing?" → `Pod keeps restarting`

3. Claude investigates your cluster:
   - Checks pod status
   - Reviews recent logs
   - Examines cluster events
   - Checks resource limits

4. Claude explains the issue:
   ```
   I found the problem! Your payment-service pod is in CrashLoopBackOff.

   Looking at the logs, it's failing because:
   - Missing environment variable: DATABASE_URL
   - The pod expects this to be set but it's not in your deployment

   Check your ConfigMap or Secret for the database configuration.
   ```

5. You fix it and verify:
   ```
   Thanks! I added the DATABASE_URL to the ConfigMap. Can you check if it's running now?
   ```

### Scenario 2: Quick Status Checks

Sometimes you just need a fast check without the full interactive session.

**Use slash commands:**

```bash
# Check if your deployment is healthy
/check-deployment production payment-service

# View recent logs from a specific pod
/view-pod-logs production payment-service-7d9f8b-xk2lp

# See what's happening in your namespace
/check-events production

# Check resource usage (CPU/Memory)
/check-resources production
```

**Example output:**
```
/check-deployment production payment-service

Deployment: payment-service (namespace: production)
├── Desired replicas: 3
├── Current replicas: 3
├── Available replicas: 3
└── Status: ✓ Healthy

All pods are running and ready!
```

### Scenario 3: Investigating Performance Issues

Your app is slow or getting OOMKilled:

```
My API is running out of memory and getting killed. What's wrong?
```

Claude will:
- Check your pod's memory limits and requests
- Show current memory usage via metrics
- Review OOMKilled events
- Compare your limits vs. actual usage
- Suggest appropriate memory settings

### Scenario 4: ImagePullBackOff Errors

You updated your deployment with a new image version but it won't pull:

```
I updated to v2.1.0 but the pod shows ImagePullBackOff
```

Claude investigates:
- Checks the exact image name and tag
- Reviews ImagePull events for error messages
- Identifies common issues:
  - Typo in image name
  - Tag doesn't exist
  - Missing image pull secrets
  - Private registry authentication

### Scenario 5: Network/Connectivity Issues

Your app can't reach the database or external APIs:

```
My frontend can't connect to the backend service
```

Claude helps by:
- Checking if backend pods are running and ready
- Verifying the Service is properly configured
- Testing DNS resolution
- Checking readiness/liveness probes
- Reviewing network policies (if applicable)

## Integration with Your Workflow

### During Development

**Before deploying:**
```bash
# Make changes to your code
git commit -m "Add new feature"

# Build and push Docker image
docker build -t myapp:v1.2.3 .
docker push myapp:v1.2.3

# Update Kubernetes manifests
# Deploy via ArgoCD, kubectl, or Helm

# Check if it's working
/check-deployment dev my-app
```

If something goes wrong, immediately ask:
```
The deployment failed, can you help debug?
```

### During Incident Response

When you get paged or notified of an issue:

1. **Open Claude Code**
2. **Ask for help:**
   ```
   Production app is down. Namespace: prod-east, app: user-service
   ```
3. **Follow the guided troubleshooting**
4. **Apply the suggested fix**
5. **Verify the resolution:**
   ```
   I restarted the pods. Are they healthy now?
   ```

### As Part of Code Review

When reviewing deployment manifests:

```
Can you check if this deployment configuration looks correct?
```

Paste your YAML and Claude can help identify:
- Missing resource limits
- Incorrect probe configurations
- Security issues
- Best practice violations

## Tips for Best Results

### 1. Be Specific About Context
**Good:**
```
My payment-api in the production namespace is failing health checks
```

**Less helpful:**
```
Something is broken
```

### 2. Provide Symptoms
Tell Claude what you're observing:
- "Pod keeps restarting"
- "Getting 503 errors"
- "Deployment is stuck at 0/3 replicas"
- "High memory usage"

### 3. Ask for Details When Needed
Claude provides summaries by default, but you can always ask:
```
Can you show me the full pod logs?
Can you explain what a liveness probe does?
What does CrashLoopBackOff mean exactly?
```

### 4. Use Commands for Speed
For quick checks, slash commands are faster:
- `/check-deployment` → Fast status check
- `/view-pod-logs` → Quick log view
- `/check-events` → Recent cluster events

For investigation, use natural language:
- "Help me debug why..." → Full guided session

### 5. Iterate on the Solution
After applying a fix:
```
I updated the ConfigMap and restarted the pod. Can you verify it's working?
```

## Common Questions

**Q: Do I need special permissions?**
A: Yes, your kubectl user needs read access to pods, deployments, logs, and events in your namespaces. Your cluster admin can grant these permissions.

**Q: Can it make changes to my cluster?**
A: No, this plugin is read-only. It helps you *diagnose* problems but won't modify your cluster. You apply the fixes yourself.

**Q: Does it work with multiple clusters?**
A: Yes! Use `kubectl config use-context` to switch clusters, then ask Claude to debug.

**Q: What if I need to debug something not covered?**
A: Just ask! Claude can use the kubernetes-mcp-server to access most Kubernetes resources. If you need custom diagnostics, you can extend the plugin.

**Q: Can I use this in CI/CD?**
A: This plugin is designed for interactive debugging. For automated checks, use standard Kubernetes tooling.

## Next Steps

Now that you're set up:

1. **Try the interactive debugger:**
   ```
   Help me check if my deployments are healthy
   ```

2. **Explore the slash commands:**
   ```
   /check-deployment <namespace> <deployment-name>
   ```

3. **Use it when you hit real issues** in your day-to-day work

4. **Share feedback** on what works and what could be better

## Need Help?

- **Plugin issues:** https://github.com/jgnesselbosch/claude-code-k8s-debug/issues
- **Claude Code help:** `/help` in Claude Code
- **Kubernetes concepts:** Just ask Claude to explain!

Happy debugging! 🚀
