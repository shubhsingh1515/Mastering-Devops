# Day 65 — Kubernetes Troubleshooting Interview Questions

## 1. How do you troubleshoot CrashLoopBackOff?

Inspect the pod description, events, termination state, current logs, and
previous-container logs. Check for startup exceptions, missing configuration,
failed probes, incorrect commands, and OOMKilled. Apply the smallest safe fix
and verify that the container remains Ready with a stable restart count.

## 2. What is the difference between CrashLoopBackOff and OOMKilled?

CrashLoopBackOff is a restart-backoff state caused by repeated container
failures. OOMKilled is a termination reason indicating an out-of-memory
condition. OOMKilled can cause CrashLoopBackOff, but the backoff can also result
from application exceptions, configuration errors, or failed probes.

## 3. What causes ImagePullBackOff?

Common causes include an incorrect image reference, missing tag or digest,
registry authentication failures, rate limits, and network or TLS issues.
Inspect pod events, validate the image reference, and check registry access
without exposing credentials.

## 4. A pod is Pending. What do you check?

Read scheduler events first. Then check resource requests, node allocatable
capacity, selectors, taints and tolerations, quotas, storage provisioning, and
cluster autoscaler behavior. HPA can request replicas but cannot guarantee that
the cluster can schedule them.

## 5. How do you troubleshoot a Service connection failure?

Verify that the Service exists, its selector matches intended pod labels, and
EndpointSlices contain Ready backends. Validate `port` and `targetPort`, test
DNS and TCP connectivity from the application environment, and inspect
NetworkPolicies and destination logs.

## 6. What is the difference between DNS failure and TCP failure?

DNS failure means the hostname cannot be resolved to an address. TCP failure
occurs after or without resolution when traffic cannot reach the target port.
They require different evidence and fixes.

## 7. Why inspect previous container logs?

The current container may have restarted and no longer contain the error that
caused the previous instance to terminate. `kubectl logs --previous` preserves
access to that preceding instance's output.

## 8. Why should you avoid blindly scaling an API when MongoDB is saturated?

More API replicas may create more database connections and queries. That can
increase database pressure without fixing the bottleneck. Investigate query
latency, connection pools, and database capacity first.

## 9. Why can a Service exist with no endpoints?

Its selector may match no pod labels, selected pods may not be Ready, or the
Service may target the wrong port. Inspect selectors, labels, readiness, and
EndpointSlices.

## 10. Why is `Running` not enough to prove recovery?

Running only shows that the container process exists. The pod may not be Ready,
the Service may have no endpoints, dependencies may be unavailable, or users
may still see high errors and latency. Verify application and business metrics.

## Final interview challenge

Your Kubernetes-hosted MERN application has increasing 503 errors. Some pods
are CrashLoopBackOff, others are Pending, and MongoDB has no ready endpoints.
Walk through your investigation.

### Model answer

I would first establish checkout impact and coordinate the incident. I would
inspect pod status and events, use previous logs for CrashLoopBackOff, and check
termination state and memory metrics for OOMKilled. For Pending pods, I would
inspect scheduling events, requests, quotas, and node capacity. I would then
compare MongoDB Service selectors with pod labels, readiness, ports, and
EndpointSlices, followed by DNS, network-policy, and database checks. I would
apply targeted fixes rather than restart everything or blindly scale the API.
Finally, I would verify ready replicas, database endpoints, error rate, latency,
and checkout success, then document root cause and prevention.

