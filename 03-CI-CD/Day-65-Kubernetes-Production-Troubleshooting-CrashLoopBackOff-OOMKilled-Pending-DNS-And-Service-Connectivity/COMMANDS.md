# Day 65 — Kubernetes Production Troubleshooting Commands

Use these commands in a development or staging environment first. Replace
placeholders with real values and avoid printing Secret data.

## Observe the cluster

```bash
kubectl get pods -n production -o wide
kubectl get deployments -n production
kubectl get replicasets -n production
kubectl get events -n production --sort-by=.metadata.creationTimestamp
kubectl get nodes
kubectl get resourcequota -n production
```

Events may be retained for a limited period, so collect them during the
incident.

## Investigate a crashing or restarting pod

```bash
kubectl describe pod <pod-name> -n production
kubectl logs <pod-name> -n production
kubectl logs <pod-name> -n production --previous
kubectl logs <pod-name> -n production --all-containers=true
kubectl get pod <pod-name> -n production \
  -o jsonpath='{.status.containerStatuses[*].lastState.terminated.reason}'
```

Check the termination reason, exit code, restart count, probe failures, events,
and application startup output. `--previous` is usually the most useful option
when the container has already restarted.

## Investigate memory and scheduling

```bash
kubectl top pods -n production
kubectl top pod <pod-name> -n production --containers
kubectl describe nodes
kubectl describe pod <pending-pod> -n production
kubectl get resourcequota -n production
kubectl get persistentvolumeclaims -n production
```

For `OOMKilled`, compare observed memory with the container limit. For
`Pending`, inspect scheduler messages, requests, allocatable node capacity,
taints, selectors, quotas, and storage events.

## Investigate image pulls

```bash
kubectl describe pod <pod-name> -n production
kubectl get deployment mern-api -n production \
  -o jsonpath='{.spec.template.spec.containers[*].image}'
kubectl get pod <pod-name> -n production \
  -o jsonpath='{.spec.imagePullSecrets[*].name}'
kubectl get secret registry-credentials -n production
```

The final command confirms only that the Secret exists; do not decode or print
its contents. The Secret and pod must be in the same namespace.

## Investigate Services, selectors, and endpoints

```bash
kubectl get services -n production
kubectl describe service mongodb -n production
kubectl get pods -n production --show-labels
kubectl get endpointslices -n production \
  -l kubernetes.io/service-name=mongodb
kubectl get endpoints mongodb -n production
```

Compare the Service selector with pod labels, then confirm that selected pods
are Ready and that the Service and target ports are correct.

## Investigate DNS and connectivity

```bash
kubectl get pods -n kube-system
kubectl exec -it <api-pod> -n production -- \
  nslookup mongodb.production.svc.cluster.local
kubectl exec -it <api-pod> -n production -- \
  sh -c 'nc -vz mongodb 27017'
kubectl get networkpolicy -n production
```

The diagnostic utilities may not exist in a minimal image. Use an approved
temporary diagnostic pod when necessary. DNS success does not prove that TCP
traffic, authentication, TLS, or database queries will succeed.

## Verify recovery

```bash
kubectl get pods -n production
kubectl rollout status deployment/mern-api -n production
kubectl get endpointslices -n production \
  -l kubernetes.io/service-name=mongodb
kubectl get hpa -n production
```

Also verify application health endpoints, error rate, latency, database
metrics, and a representative business workflow such as checkout.

## Safe operating rules

1. Collect evidence before deleting or restarting a pod.
2. Use `kubectl logs --previous` before the evidence disappears.
3. Do not expose Secret values in terminals, tickets, or chat.
4. Do not scale API replicas while a database bottleneck is unexamined.
5. Apply one targeted mitigation at a time when possible.
6. Confirm user-facing recovery, not only `Running` pod status.

