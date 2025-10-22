# /check-resources

View resource consumption (CPU and memory) for pods in your cluster.

Helps identify:
- Memory leaks and high memory usage
- CPU throttling and high CPU usage
- Resource constraint violations
- Capacity planning issues

## Usage

```
/check-resources [namespace] [--pod POD_NAME]
```

## Examples

```
/check-resources
/check-resources production
/check-resources default --pod my-app-abc123
```

## Output

Shows for each pod:
- **CPU** - Current CPU usage
- **Memory** - Current memory usage

This data comes from the Kubernetes Metrics Server.

## Interpreting results

- Compare actual usage against resource limits (defined in pod spec)
- Pods using more than their limits will be throttled (CPU) or killed (memory)
- High memory usage may indicate a leak
- Sustained high CPU may indicate inefficient code or infinite loops

## Tips

- Run this when experiencing slowness or crashes
- Check if any pods are near their memory limits
- Compare resource limits to actual usage to right-size requests
- Use with pod logs to correlate high resource usage with specific events

## Prerequisites

- Kubernetes Metrics Server must be installed in your cluster
- For most distributions, this is installed by default

## Related Commands

- `/debug-app` - For guided troubleshooting
- `/check-deployment` - For deployment status
- `/view-pod-logs` - To investigate related errors
