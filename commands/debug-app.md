# /debug-app

Start an interactive debugging session for a failing application deployment in your Kubernetes cluster.

This command launches the Kubernetes Deployment Debugger skill, which will:
- Guide you through a systematic diagnostic process
- Ask targeted questions about your deployment
- Analyze cluster state using the kubernetes MCP server
- Help identify and resolve issues

## Usage

```
/debug-app
```

## Example

```
/debug-app
```

When you run this command, the debugger will ask you for:
1. The namespace where your application is deployed
2. The name of your deployment or pod
3. The symptoms or issues you're experiencing

Then it will begin investigating and guide you to a solution.

## What it does

- Checks deployment and pod status
- Analyzes logs and events
- Identifies resource constraints
- Checks probe configurations
- Suggests remediation steps
- Provides detailed explanations when needed

## Tips

- Be as specific as possible about what's not working
- Tell the debugger if you want more technical details
- Ask follow-up questions at any point
- The debugger can investigate multiple deployments in a session
