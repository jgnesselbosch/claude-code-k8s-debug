# ArgoCD Setup for the Plugin

This plugin includes ArgoCD-specific troubleshooting capabilities. The skill uses the `argocd` CLI tool via the Bash tool to interact with ArgoCD.

## Prerequisites

### 1. Install ArgoCD CLI

**Linux:**
```bash
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/argocd
```

**macOS:**
```bash
brew install argocd
```

**Windows:**
```powershell
choco install argocd-cli
```

**Verify installation:**
```bash
argocd version --client
```

### 2. Login to ArgoCD

The `argocd` CLI needs to be authenticated to your ArgoCD server.

**Option A: Login via CLI**
```bash
# Login to your ArgoCD server
argocd login <argocd-server-url>

# Example:
argocd login argocd.example.com

# You'll be prompted for username/password or can use SSO
```

**Option B: Login with Token**
```bash
# For CI/CD or automation, use a token
argocd login <argocd-server-url> --auth-token <your-token>
```

**Option C: Port-forward and Login Locally**
```bash
# If ArgoCD is running in your cluster
kubectl port-forward svc/argocd-server -n argocd 8080:443

# In another terminal
argocd login localhost:8080 --insecure
```

**Verify login:**
```bash
argocd app list
```

### 3. Set Default Context (Optional)

If you work with multiple ArgoCD instances:

```bash
# Login to multiple servers
argocd login prod.argocd.example.com --name prod
argocd login dev.argocd.example.com --name dev

# Switch contexts
argocd context prod
argocd context dev

# View contexts
argocd context
```

## How the Plugin Uses ArgoCD CLI

The plugin's skill (`.claude/skills/k8s-deployment-debugger.md`) has access to the `Bash` tool, which allows it to run `argocd` CLI commands when needed.

### Commands Used by the Skill

When you ask Claude to debug an ArgoCD-managed application, it may run:

```bash
# Check application status
argocd app get <app-name>

# View sync/health status
argocd app get <app-name> -o json

# Show differences between Git and cluster
argocd app diff <app-name>

# View deployment history
argocd app history <app-name>

# Sync application
argocd app sync <app-name>

# Rollback to previous version
argocd app rollback <app-name> <revision-id>

# List applications
argocd app list
```

### Available Slash Commands

The plugin provides these ArgoCD-specific commands:

- **`/check-argocd-app <app-name>`** - Check sync status and health
- **`/sync-argocd-app <app-name>`** - Manually trigger sync
- **`/argocd-app-diff <app-name>`** - Show Git vs cluster differences
- **`/argocd-app-history <app-name>`** - View deployment history
- **`/argocd-app-rollback <app-name> <revision>`** - Rollback deployment

## Permissions Required

The ArgoCD user you're logged in as needs these permissions:

```yaml
# Minimum permissions for read-only debugging
applications:
  - get
  - list
  - diff
  - history

# Additional permissions for write operations (sync, rollback)
applications:
  - sync
  - rollback
```

### Read-Only Access

If you only want debugging without making changes:

```bash
# The skill can still help with:
# - Checking app status
# - Viewing diffs
# - Reviewing history
# - Identifying issues

# But won't be able to:
# - Trigger syncs
# - Rollback deployments
```

## Troubleshooting ArgoCD CLI Access

### Issue: "argocd: command not found"

**Solution:**
```bash
# Verify installation
which argocd

# If not found, install using instructions above
# Make sure it's in your PATH
echo $PATH
```

### Issue: "FATA[0000] rpc error: code = Unauthenticated"

**Solution:**
```bash
# Re-login to ArgoCD
argocd login <argocd-server-url>

# Or use token-based auth
argocd login <argocd-server-url> --auth-token <token>
```

### Issue: "context deadline exceeded" or timeouts

**Solution:**
```bash
# Check network connectivity to ArgoCD server
curl -k https://<argocd-server-url>

# If using port-forward, ensure it's still running
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Increase timeout
argocd app get <app-name> --grpc-web --plaintext
```

### Issue: "permission denied" errors

**Solution:**
```bash
# Check your current user's permissions
argocd account get-user-info

# Contact your ArgoCD admin to grant necessary permissions
# Or use a different ArgoCD user with appropriate access
```

## Environment Configuration

### For Team Use

Create a shared configuration guide for your team:

```bash
# .envrc or team setup script
export ARGOCD_SERVER=argocd.example.com
export ARGOCD_AUTH_TOKEN=<team-token>  # For automation
export ARGOCD_OPTS="--grpc-web"        # If using ingress

# Login once
argocd login $ARGOCD_SERVER --auth-token $ARGOCD_AUTH_TOKEN
```

### For Multi-Cluster/Multi-Environment

```bash
# Production
argocd login prod.argocd.example.com --name prod

# Staging
argocd login staging.argocd.example.com --name staging

# Development
argocd login dev.argocd.example.com --name dev

# Switch as needed
argocd context prod
argocd context staging
argocd context dev
```

## Integrating with the Plugin

Once ArgoCD CLI is set up, the plugin will automatically use it when you:

1. **Ask about ArgoCD applications:**
   ```
   My payment-service ArgoCD app is OutOfSync, can you help?
   ```

2. **Run ArgoCD slash commands:**
   ```
   /check-argocd-app my-payment-service
   ```

3. **Debug deployments managed by ArgoCD:**
   ```
   The deployment is failing and it's managed by ArgoCD
   ```

The skill will automatically:
- Detect it's an ArgoCD-managed app
- Check ArgoCD sync/health status first
- Identify if the issue is at the ArgoCD level (sync failed, OutOfSync) or pod level (CrashLoop, etc.)
- Suggest appropriate fixes

## Best Practices

1. **Use Read-Only Access for Debugging**
   - Developers can use read-only ArgoCD access
   - Prevents accidental syncs or rollbacks
   - SRE/Ops can have write access for fixes

2. **Token Rotation**
   - Rotate ArgoCD tokens regularly
   - Use short-lived tokens in CI/CD
   - Store tokens securely (not in shell history)

3. **Context Awareness**
   - Always verify you're on the correct ArgoCD context
   - Use `argocd context` before running commands
   - Alias contexts to prevent mistakes (prod vs dev)

4. **Combine with Kubernetes Access**
   - The plugin works best with both ArgoCD and kubectl access
   - ArgoCD shows "desired state" (Git)
   - Kubernetes shows "actual state" (cluster)
   - Together they provide complete visibility

## Further Reading

- [ArgoCD CLI Documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd/)
- [ArgoCD Authentication](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd_login/)
- [ArgoCD RBAC](https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/)
