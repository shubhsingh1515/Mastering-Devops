# 🚀 Day 65 — Kubernetes Production Troubleshooting

**Phase 4:** DevSecOps & Production Engineering  
**Level:** Intermediate → Advanced  
**Focus:** CrashLoopBackOff, OOMKilled, Pending pods, ImagePullBackOff, DNS, Service connectivity, and evidence-driven incident response.

## Today's mission

Day 64 covered health probes, resource requests and limits, autoscaling, rolling
updates, and graceful shutdown. Today focuses on what to do when those controls
expose or encounter a production failure.

By the end of this lesson, you should be able to:

- Follow a repeatable troubleshooting sequence instead of restarting everything.
- Distinguish a restart-backoff state from an OOM termination reason.
- Diagnose Pending pods using scheduler events and capacity evidence.
- Investigate image-pull failures without exposing credentials.
- Separate DNS failures from Service, port, policy, and database failures.
- Verify recovery with ready endpoints, application metrics, and business workflows.
- Document root cause and preventive controls after an incident.

## Learning schedule

| Time | Topic |
|---|---|
| 0–4 min | Troubleshooting framework |
| 4–8 min | CrashLoopBackOff and OOMKilled |
| 8–12 min | Pending pods and scheduling |
| 12–16 min | ImagePullBackOff |
| 16–20 min | DNS and Service connectivity |
| 20–24 min | Production incident and interviews |
| 24–30 min | Quiz and practical assessment |

## 1. The troubleshooting framework

Use the same sequence for most Kubernetes incidents:

```text
OBSERVE
   ↓
IDENTIFY THE FAILURE LAYER
   ↓
COLLECT EVIDENCE
   ↓
APPLY THE SMALLEST SAFE FIX
   ↓
VERIFY RECOVERY
   ↓
DOCUMENT AND PREVENT RECURRENCE
```

Start with workload status, deployments, and recent events:

```bash
kubectl get pods -n production
kubectl get deployments -n production
kubectl get events -n production --sort-by=.metadata.creationTimestamp
```

The failure may be in application configuration, the image registry, pod
scheduling, resource limits, probes, Services, DNS, NetworkPolicies, MongoDB,
or cluster capacity. Do not assume that a Kubernetes symptom identifies the
root cause.

## 2. CrashLoopBackOff and OOMKilled

`CrashLoopBackOff` is a pod state that means a container has repeatedly
terminated and Kubernetes is delaying another restart. Common causes include:

- Startup exceptions or missing environment variables.
- Database initialization failures.
- An incorrect command or entrypoint.
- Failed liveness probes.
- An OOMKilled termination.

Use the previous-container logs because the current instance may have restarted:

```bash
kubectl describe pod <pod-name> -n production
kubectl logs <pod-name> -n production
kubectl logs <pod-name> -n production --previous
kubectl describe deployment mern-api -n production
```

`OOMKilled` is a termination reason, not the same thing as CrashLoopBackOff. It
means the container exceeded its memory limit or was terminated by an
out-of-memory condition. Inspect the termination state, restart count, memory
limit, and metrics:

```bash
kubectl top pod <pod-name> -n production --containers
```

Do not increase the limit blindly. Check for memory leaks, unbounded caching,
large in-memory work, workload growth, and an incorrectly sized limit.

## 3. Pending pods and scheduling failures

A Pending pod has not completed scheduling and startup. Describe it and read the
Events section:

```bash
kubectl describe pod <pod-name> -n production
kubectl get nodes
kubectl describe nodes
kubectl get resourcequota -n production
```

Typical causes include insufficient CPU or memory, node-selector mismatches,
taints without tolerations, storage provisioning failures, and namespace
resource quotas. HPA can increase the desired replica count, but it cannot
guarantee that the cluster has capacity to schedule ready replicas.

Do not reduce resource requests simply to force scheduling if the workload
actually needs those resources. That can convert a scheduling failure into
runtime throttling or OOM failures.

## 4. ImagePullBackOff

`ImagePullBackOff` means Kubernetes failed to retrieve the image and is backing
off before retrying. Check pod events and the image reference:

```bash
kubectl describe pod <pod-name> -n production
kubectl get deployment mern-api -n production \
  -o jsonpath='{.spec.template.spec.containers[*].image}'
```

Investigate an incorrect image name, missing tag or digest, registry
authentication, rate limits, network access, and TLS problems. For private
registries, verify that the `imagePullSecret` exists in the same namespace and
that its reference is correct. Never print or commit real registry credentials.

Prefer approved immutable digests where the release policy supports them:

```yaml
image: registry.example.com/mern-api@sha256:APPROVED_DIGEST
```

Image retrieval, signature verification, and vulnerability scanning are
separate controls. A successful image pull does not prove that the image is
trusted or free of known vulnerabilities.

## 5. DNS and Service connectivity

For a connection such as `mongodb://mongodb:27017/shop`, `mongodb` normally
refers to a Kubernetes Service in the caller's namespace. A fully qualified
name can be used when appropriate:

```text
mongodb.production.svc.cluster.local
```

The cluster DNS suffix may differ from `cluster.local`.

Check the Service and DNS components:

```bash
kubectl get services -n production
kubectl get endpointslices -n production \
  -l kubernetes.io/service-name=mongodb
kubectl get pods -n kube-system
kubectl exec -it <api-pod> -n production -- \
  nslookup mongodb.production.svc.cluster.local
```

The `nslookup` command is available only if the image contains it. Use an
approved diagnostic pod when the production image is intentionally minimal.

DNS resolution and TCP connectivity are different checks. After DNS resolves,
verify Service selectors, ready endpoints, ports, NetworkPolicies, the
application's listening port, MongoDB authentication, and database health.

## 6. Services with no ready endpoints

A Service can exist while routing to no pods. For example, a Service selecting
`app: mongodb` will not select a pod labeled `app: mongo-db`.

```yaml
spec:
  selector:
    app: mongodb
  ports:
    - port: 27017
      targetPort: 27017
```

Check labels, selectors, readiness, EndpointSlices, and port mapping:

```bash
kubectl get pods -n production --show-labels
kubectl describe service mongodb -n production
kubectl get endpointslices -n production \
  -l kubernetes.io/service-name=mongodb
```

`port` is the Service port; `targetPort` is the port on the selected pod. A
matching selector is not enough if the target process is not listening or the
pods are not Ready.

## 7. Production incident example

Suppose checkout errors rise to 12%, API p95 latency reaches four seconds, only
two of five API replicas are ready, two pods are CrashLoopBackOff, one is
Pending, and MongoDB has no ready endpoints.

1. Establish user impact and coordinate the incident.
2. Inspect events and previous logs for the crashing pods.
3. Check termination state and memory metrics for OOMKilled.
4. Inspect scheduler events, requests, quotas, and node capacity for Pending.
5. Compare the MongoDB Service selector with pod labels and readiness.
6. Investigate database load, query latency, and connection pools.
7. Apply targeted fixes or roll back the change that caused the regression.
8. Verify ready replicas, endpoints, error rate, latency, and checkout.

Do not automatically scale the API while MongoDB is saturated. More API replicas
can increase database pressure without addressing the bottleneck.

## Final takeaways

| Symptom | Meaning |
|---|---|
| `CrashLoopBackOff` | Repeated container termination with restart backoff |
| `OOMKilled` | Container termination caused by an out-of-memory condition |
| `Pending` | Pod has not completed scheduling and startup |
| `ImagePullBackOff` | Kubernetes is backing off after an image-pull failure |
| DNS failure | The requested hostname cannot be resolved successfully |
| Empty endpoints | The Service has no usable ready backends |
| HTTP 503 | The request was not served; investigate the actual failure layer |

The Day 65 files provide the operational detail:

- [COMMANDS.md](./COMMANDS.md) — safe diagnostic and recovery commands.
- [KUBERNETES-TROUBLESHOOTING.md](./KUBERNETES-TROUBLESHOOTING.md) — incident runbook.
- [ASSIGNMENT.md](./ASSIGNMENT.md) — practical exercise and assessment.
- [INTERVIEW-QUESTIONS.md](./INTERVIEW-QUESTIONS.md) — interview preparation.
- [QUIZ.md](./QUIZ.md) — knowledge check.

