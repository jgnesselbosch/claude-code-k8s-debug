# /view-pod-logs

Display logs from a Kubernetes pod with filtering and context options.

Useful for:
- Seeing application error messages
- Tracing execution flow
- Identifying failure points
- Debugging startup issues

## Usage

```
/view-pod-logs <namespace> <pod-name> [--previous] [--tail N] [--container NAME]
```

## Examples

```
/view-pod-logs default my-app-pod-abc123
/view-pod-logs production api-service-xyz789 --tail 50
/view-pod-logs kube-system coredns-xyz --previous
/view-pod-logs production multi-container-app --container sidecar --tail 100
```

## Options

- `--previous` - Show logs from the previous terminated container (useful for CrashLoopBackOff)
- `--tail N` - Show the last N lines (default: 100)
- `--container NAME` - Specify which container's logs to view (for multi-container pods)

## Tips

- Use `--previous` when investigating pod crashes
- Grep for "ERROR" or "Exception" in the output to find issues quickly
- Check timestamps to correlate with events
- Previous logs often reveal the root cause of CrashLoopBackOff

## Related Commands

- `/debug-app` - For guided troubleshooting
- `/check-deployment` - To find pod names
- `/check-events` - To see cluster-level events
