# /check-deployment

Quick status check for a specific Kubernetes deployment.

Returns immediate information about:
- Replica status (desired, current, ready, updated)
- Rollout status
- Recent events
- Pod status overview

## Usage

```
/check-deployment <namespace> <deployment-name>
```

## Examples

```
/check-deployment default my-app
/check-deployment production api-service
/check-deployment kube-system coredns
```

## What it shows

- Deployment replica information
- Pod readiness status
- Recent warnings or errors
- Image and resource configuration
- Last update timestamp

## When to use

- Quick health check before detailed debugging
- Verify deployment status in CI/CD pipelines
- Monitor during deployment rollouts
- Confirm fix after applying changes

## Related Commands

- `/debug-app` - For detailed troubleshooting
- `/view-pod-logs` - To see application logs
- `/check-events` - To see cluster events
