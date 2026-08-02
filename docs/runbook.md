# Runbook

## Pod Failure

```bash
kubectl get pods -n taskflow
kubectl describe pod <pod-name> -n taskflow
kubectl logs <pod-name> -n taskflow
```

Kubernetes will automatically recreate a crashed pod (self-healing) as
long as the Deployment's `replicas` count isn't satisfied. If pods keep
crash-looping, check `kubectl logs` for the actual NGINX error.

## Deployment Failure

```bash
kubectl rollout status deployment/taskflow-app -n taskflow
kubectl describe deployment taskflow-app -n taskflow
```

Common causes: bad image tag, image pull auth failure, readiness probe
never passing.

## Rollback Procedure

```bash
kubectl rollout history deployment/taskflow-app -n taskflow
kubectl rollout undo deployment/taskflow-app -n taskflow
# or roll back to a specific revision:
kubectl rollout undo deployment/taskflow-app -n taskflow --to-revision=<N>
kubectl get pods -n taskflow
```

## Scaling Procedure

Manual:
```bash
kubectl scale deployment taskflow-app --replicas=5 -n taskflow
```

Automatic (once `hpa.yaml` is applied):
```bash
kubectl get hpa -n taskflow
```
HPA will scale between 2 and 10 replicas based on 70% average CPU
utilization.

## On-call Checklist

1. Is the app reachable? `minikube service taskflow-service -n taskflow --url`
2. Are pods Ready? `kubectl get pods -n taskflow`
3. Any recent deploy? `kubectl rollout history deployment/taskflow-app -n taskflow`
4. Check Grafana dashboards for CPU/memory spikes or restart counts.
5. If uncertain, roll back first, investigate after.
