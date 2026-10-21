# Day 64 — Practical Assessment

## Exercise: Troubleshoot an unhealthy MERN API

An Express API has three desired replicas, one ready replica, increasing
restarts, memory usage near its limit, API p95 latency of 2.5 seconds, and
MongoDB CPU at 85%.

### Tasks

1. List at least three possible causes without assuming a single cause.
2. Collect evidence with `kubectl describe`, current and previous logs, events,
   and resource metrics.
3. Explain how you would distinguish an OOMKill, startup restart loop, and
   database bottleneck.
4. Define the metrics that prove recovery.

## Zero-downtime release scenario

Three replicas are being upgraded to `v27` with `maxSurge: 1` and
`maxUnavailable: 0`. The new version takes 40 seconds to start, has no startup
probe, and its readiness endpoint returns success immediately. Requests fail
while database initialization is still running.

### Expected solution

1. Make readiness depend on actual application readiness.
2. Add a startup probe with a threshold that covers expected initialization.
3. Keep liveness independent from temporary MongoDB slowness where possible.
4. Confirm old and new versions can coexist during the rollout.
5. Test in staging while continuously exercising a representative workflow.
6. Monitor error rate, p95/p99 latency, ready replicas, restarts, database
   latency, and checkout success rate.

## Review questions

1. What is the difference between RPO and RTO?
2. Why does an image signature not prove that an image is vulnerability-free?
3. Why should a MERN API avoid unnecessary Kubernetes API permissions?
4. What must be verified before enforcing default-deny egress?
5. How is readiness different from a PodDisruptionBudget?

<details>
<summary>Answers</summary>

1. RPO is the acceptable data-loss window; RTO is the target restoration time.
2. A signature proves artifact integrity and signing identity; vulnerability
   scanning addresses known security issues.
3. Excess permissions increase the impact of a compromised workload.
4. Verify DNS, monitoring, health checks, database traffic, and required
   external dependencies.
5. Readiness controls Service traffic; a PDB limits certain voluntary
   disruptions. Neither guarantees availability during every failure.

</details>

## Quiz

1. Which probe controls normal Service traffic?  
   **Answer: Readiness.**
2. Why use a startup probe?  
   **Answer: To let initialization complete before liveness and readiness checks.**
3. What may happen when memory exceeds its limit?  
   **Answer: The container may be terminated with an out-of-memory condition.**
4. What commonly happens at a CPU limit?  
   **Answer: CPU throttling.**
5. What does HPA adjust?  
   **Answer: Workload replicas based on configured metrics.**
6. What does CPU-utilization HPA require?  
   **Answer: CPU requests and usable resource metrics.**
7. What does `maxUnavailable: 0` aim to prevent?  
   **Answer: Intentionally reducing available replicas below the desired count during a normal rollout.**
8. Why implement graceful shutdown?  
   **Answer: To stop accepting new work and finish in-flight work safely.**
9. What is a rolling-update limitation?  
   **Answer: It cannot automatically solve compatibility or dependency failures.**
10. What is a PodDisruptionBudget for?  
    **Answer: Limiting certain voluntary disruptions to help maintain availability.**

## Interview challenge

Design a Kubernetes strategy for a MERN application that handles traffic spikes
and supports zero-downtime releases. Cover probes, resource requests and
limits, HPA metrics, cluster capacity, rolling updates, graceful shutdown,
application/database compatibility, SLO monitoring, rollback, and post-release
validation.
