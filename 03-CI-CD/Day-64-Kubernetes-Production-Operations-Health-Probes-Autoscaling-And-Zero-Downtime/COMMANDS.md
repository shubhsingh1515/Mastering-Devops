# Day 64 — Production Operations Commands

## Apply and observe

```bash
kubectl apply -f DEPLOYMENT.yaml
kubectl apply -f HPA.yaml
kubectl get deployment mern-api -n production
kubectl get pods -n production -l app=mern-api -w
kubectl get hpa mern-api -n production
```

## Rollout and rollback

```bash
kubectl rollout status deployment/mern-api -n production
kubectl rollout history deployment/mern-api -n production
kubectl rollout pause deployment/mern-api -n production
kubectl rollout resume deployment/mern-api -n production
kubectl rollout undo deployment/mern-api -n production
```

A rollback restores a Deployment revision. It does not automatically reverse
database schema changes, data mutations, or external side effects.

## Troubleshoot a pod

```bash
kubectl describe pod <pod-name> -n production
kubectl logs <pod-name> -n production
kubectl logs <pod-name> -n production --previous
kubectl get events -n production --sort-by=.lastTimestamp
kubectl top pods -n production
kubectl top pod <pod-name> -n production --containers
```

Check termination reasons, restart count, probe failures, events, resource
configuration, and prior-container logs. `kubectl top` requires usable metrics
components.

## Evidence-driven diagnosis

| Symptom | First checks | Likely direction |
|---|---|---|
| `OOMKilled` | Memory usage, limits, termination state, application logs | Memory leak, workload spike, or undersized limit |
| Repeated startup restarts | Startup and liveness timing, initialization logs | Missing or overly aggressive startup protection |
| High API latency | CPU throttling, p95/p99 latency, MongoDB metrics | Event-loop, query, connection-pool, or database bottleneck |
| HPA does not scale | HPA status, resource requests, metrics API, cluster capacity | Missing metrics, bad requests, or unschedulable replicas |
| Rollout stalls | Ready replicas, readiness failures, events, capacity | New version is not ready or cannot be scheduled |

Do not add API replicas automatically when MongoDB is saturated. Investigate
query latency, connection-pool behavior, and database capacity first.

## Recovery validation

After remediation, verify all of the following:

```text
Ready replicas = desired replicas
Restart count  = stable
API latency    = within target
Error rate     = within SLO
Checkout       = functioning
```

Deployment success only proves that the controller completed its work. It does
not prove that users experienced zero downtime.

## Release checklist

1. Confirm the image uses an approved digest.
2. Confirm health endpoints report real application state.
3. Apply to staging and test old/new version compatibility.
4. Watch ready replicas, restarts, error rate, and p95/p99 latency.
5. Exercise a representative API workflow during rollout.
6. Check database connection and query latency.
7. Pause or roll back if SLO or business metrics degrade.
