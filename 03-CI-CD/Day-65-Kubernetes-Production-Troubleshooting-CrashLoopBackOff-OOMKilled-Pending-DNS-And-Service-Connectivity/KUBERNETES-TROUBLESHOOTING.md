# Kubernetes Production Troubleshooting Runbook

## Purpose

Use this runbook to diagnose Kubernetes failures affecting a production MERN
application. The goal is to preserve evidence, identify the failure layer,
apply the smallest safe mitigation, and verify recovery.

## Incident checklist

1. Establish user and business impact.
2. Identify affected namespace, workload, pods, and recent changes.
3. Collect pod descriptions, events, current logs, and previous logs.
4. Isolate application, image, scheduling, networking, dependency, or cluster causes.
5. Apply a targeted fix or rollback with incident-owner approval.
6. Verify pod readiness, Service endpoints, metrics, and business workflows.
7. Record root cause, evidence, remediation, and preventive controls.

## Failure response matrix

| Failure | Symptoms | First evidence |
|---|---|---|
| CrashLoopBackOff | Repeated restarts and backoff delay | `describe`, current logs, `--previous` logs |
| OOMKilled | Termination reason is OOMKilled, memory near limit | Termination state, limits, `kubectl top` |
| Pending | Desired replicas exceed ready replicas | Pod Events, requests, node capacity |
| ImagePullBackOff | Pod never starts and image events fail | Pod Events, image reference, pull Secret |
| DNS failure | Service hostname cannot be resolved | Service existence, CoreDNS, lookup from pod |
| No endpoints | Service exists but has no ready backends | Selector, labels, EndpointSlices, readiness |
| Readiness failure | Pod runs but is not Ready | Probe events, endpoint behavior, app logs |
| MongoDB failure | API cannot query database | DNS, endpoints, ports, policy, DB logs |

## CrashLoopBackOff

### Symptoms

- Pod status shows `CrashLoopBackOff`.
- Restart count increases.
- The application may never become Ready.

### Diagnostics

```bash
kubectl describe pod <pod-name> -n production
kubectl logs <pod-name> -n production --previous
kubectl get deployment mern-api -n production
```

### Likely causes

Missing configuration, startup exceptions, invalid commands, failed database
initialization, or an overly aggressive liveness probe. OOMKilled can be the
termination cause behind the backoff state.

### Remediation and verification

Correct the configuration, command, application failure, or probe timing based
on evidence. Then confirm stable logs, a Ready pod, a stable restart count, and
successful application requests. Do not delete the pod before collecting logs.

## OOMKilled

### Symptoms

The container's last termination reason is `OOMKilled`; memory usage approaches
the configured limit and restarts may follow.

### Diagnostics

```bash
kubectl describe pod <pod-name> -n production
kubectl top pod <pod-name> -n production --containers
kubectl get deployment mern-api -n production -o yaml
```

### Likely causes

Memory leaks, unbounded caches, large in-memory operations, workload growth, or
an incorrectly sized limit.

### Remediation and verification

Profile and reduce memory use, bound caches, correct the workload, or adjust a
validated limit. Confirm that memory remains below the limit, restarts stop, and
latency and error metrics recover. Raising a limit without investigating only
delays recurrence.

## Pending pods

### Symptoms

The pod remains Pending and the Deployment has fewer ready replicas than
desired.

### Diagnostics

```bash
kubectl describe pod <pod-name> -n production
kubectl get nodes
kubectl describe nodes
kubectl get resourcequota -n production
kubectl get persistentvolumeclaims -n production
```

### Likely causes and remediation

Check scheduler events for insufficient CPU or memory, node selectors, taints
and tolerations, quotas, storage, and cluster autoscaler behavior. Add
capacity or correct an invalid constraint when appropriate. Do not lower
requests below the workload's actual requirement just to make the pod schedule.

Verify that the pod is scheduled, starts, becomes Ready, and that the desired
number of usable replicas is available.

## ImagePullBackOff

### Symptoms

Events show `ErrImagePull` followed by `ImagePullBackOff`; the container has not
started.

### Diagnostics

```bash
kubectl describe pod <pod-name> -n production
kubectl get deployment mern-api -n production \
  -o jsonpath='{.spec.template.spec.containers[*].image}'
kubectl get secret registry-credentials -n production
```

### Likely causes and remediation

Correct an invalid tag, repository, digest, registry permission, pull Secret,
network route, certificate, or rate-limit issue. Keep credentials out of Git
and command output. Verify the approved image digest and signature policy
before retrying the rollout.

## DNS resolution failures

### Symptoms

The API cannot resolve `mongodb` or its fully qualified Service name.

### Diagnostics

```bash
kubectl get services -n production
kubectl get pods -n kube-system
kubectl exec -it <api-pod> -n production -- \
  nslookup mongodb.production.svc.cluster.local
kubectl get networkpolicy -n production
```

### Likely causes and remediation

Check the Service name and namespace, CoreDNS health, pod DNS configuration,
and policies that may block DNS egress. If lookup succeeds, stop treating the
incident as DNS and investigate TCP connectivity, ports, endpoints, policy,
authentication, and MongoDB health separately.

## Services with no ready endpoints

### Symptoms

The Service exists, but EndpointSlices are empty or contain no ready backend.

### Diagnostics

```bash
kubectl describe service mongodb -n production
kubectl get pods -n production --show-labels
kubectl get endpointslices -n production \
  -l kubernetes.io/service-name=mongodb
```

### Likely causes and remediation

Compare selectors with pod labels, then check pod readiness and Service
`targetPort`. Correct the selector or labels only after confirming the intended
ownership. Fix the readiness failure or target process when the labels are
correct. Verify that EndpointSlices contain the expected ready addresses.

## Failed readiness probes

### Symptoms

The pod is Running but not Ready and is excluded from Service endpoints.

### Diagnostics

```bash
kubectl describe pod <pod-name> -n production
kubectl logs <pod-name> -n production
kubectl exec -it <pod-name> -n production -- \
  sh -c 'wget -qO- http://127.0.0.1:5000/health/ready'
```

### Likely causes and remediation

Check the path, port, timing, application initialization, dependency state,
and shutdown state. Make readiness represent the application's ability to
serve traffic; do not make it return success before initialization completes.
Verify that the pod becomes Ready without masking real dependency failures.

## MongoDB connection failures

### Diagnostics

```bash
kubectl get service mongodb -n production
kubectl get endpointslices -n production \
  -l kubernetes.io/service-name=mongodb
kubectl exec -it <api-pod> -n production -- \
  nslookup mongodb
kubectl get networkpolicy -n production
```

Check the connection string's host, namespace, port, endpoint readiness,
NetworkPolicy, MongoDB authentication/TLS, connection-pool usage, and database
health. Do not assume that a running MongoDB pod means the database is
reachable or accepting application traffic.

## Recovery verification

Recovery is complete only when:

- Required replicas are Ready and restart counts are stable.
- Services have expected ready EndpointSlices.
- DNS and TCP checks succeed from the application environment.
- API error rate and p95/p99 latency meet their targets.
- MongoDB load and query latency are acceptable.
- A critical workflow such as checkout succeeds.
- The incident note records evidence, root cause, fix, and prevention.

