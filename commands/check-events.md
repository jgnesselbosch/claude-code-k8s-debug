# /check-events

View Kubernetes cluster events, optionally filtered by namespace or resource.

Cluster events provide valuable context about what's happening in your cluster:
- Pod creation and termination
- Image pull failures
- Resource constraint warnings
- Health check failures
- Node pressure events

## Usage

```
/check-events [namespace]
```

## Examples

```
/check-events
/check-events default
/check-events production
/check-events kube-system
```

## What events show

Each event includes:
- **Type** - Normal or Warning
- **Reason** - Why the event occurred (e.g., ImagePullBackOff, OOMKilled, Unhealthy)
- **Message** - Description of what happened
- **Object** - Which Kubernetes resource it involves
- **Timestamp** - When it occurred

## Reading the output

- Focus on **Warning** events when troubleshooting
- **ImagePullBackOff** - The container image couldn't be found/pulled
- **OOMKilled** - Pod ran out of memory
- **Unhealthy** - Liveness probe failed
- **FailedScheduling** - Pod couldn't be placed on a node

## Tips

- No namespace specified shows all events from all namespaces (noisy)
- Run this early in debugging to see what warnings exist
- Events are rotated out after a period, so recent ones are most relevant
- Compare timing with your deployment changes

## Related Commands

- `/debug-app` - For interactive guided debugging
- `/check-deployment` - For deployment status
- `/view-pod-logs` - To see application logs
