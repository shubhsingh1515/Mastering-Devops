# 🚀 Day 64 — Kubernetes Production Operations

**Date:** Wednesday, October 14, 2026  
**Phase 4:** DevSecOps & Production Engineering  
**Level:** Intermediate → Advanced  
**Focus:** Health probes, resource management, autoscaling, rolling updates, and zero-downtime MERN deployments.

## Today's mission

Day 63 secured the Kubernetes runtime with RBAC, service accounts, NetworkPolicies,
and pod security controls. Today, those secure workloads become reliable under
real production conditions.

By the end of this lesson, you should be able to:

- Separate startup, liveness, and readiness responsibilities.
- Choose realistic CPU and memory requests and limits.
- Configure an HPA with the metrics it needs.
- Roll out a new version without intentionally reducing available capacity.
- Drain a Node.js process gracefully before termination.
- Investigate OOMKills, restart loops, slow startup, and rollout failures.
- Validate availability using SLO and business metrics instead of deployment status alone.

## Learning schedule

| Time | Topic |
|---|---|
| 0–5 min | Kubernetes health probes |
| 5–10 min | CPU and memory requests and limits |
| 10–15 min | Horizontal Pod Autoscaling |
| 15–20 min | Rolling updates and zero downtime |
| 20–24 min | Troubleshooting and interviews |
| 24–30 min | Review, quiz, and assessment |

## 1. Health probes

| Probe | Question answered | Failure consequence |
|---|---|---|
| Startup | Has initialization completed? | Container can be restarted after the startup threshold |
| Liveness | Is the process functioning? | Container is restarted after repeated failures |
| Readiness | Can this pod receive traffic? | Pod is removed from normal Service endpoints |

Use a startup probe when an Express API needs time to load configuration,
connect to MongoDB, or prepare internal resources. While startup has not
succeeded, Kubernetes does not run liveness and readiness probes.

Liveness should usually test the process itself, not every external dependency.
Restarting every API pod because MongoDB is temporarily unavailable can make an
outage worse. Readiness may include dependency state when the application truly
cannot serve requests, but it must not remove every replica unnecessarily.

Example health endpoints:

```js
let shuttingDown = false;
let databaseReady = false;

app.get("/health/live", (_req, res) => {
  res.status(200).json({ status: "alive" });
});

app.get("/health/startup", (_req, res) => {
  if (!databaseReady) {
    return res.status(503).json({ status: "starting" });
  }
  res.status(200).json({ status: "started" });
});

app.get("/health/ready", (_req, res) => {
  if (shuttingDown || !databaseReady) {
    return res.status(503).json({ status: "not-ready" });
  }
  res.status(200).json({ status: "ready" });
});
```

The real application must update `databaseReady` from actual initialization and
connection state. A readiness endpoint that returns success immediately can send
traffic to an application that is still starting.

## 2. Resource requests and limits

```yaml
resources:
  requests:
    cpu: "200m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
```

Requests influence scheduling and CPU-utilization HPA calculations. CPU limits
can cause throttling under sustained demand. Exceeding a memory limit may
terminate a container with an `OOMKilled` reason. Validate values with load
testing and production metrics instead of increasing limits blindly.

## 3. Autoscaling

An HPA adjusts replicas based on configured metrics. CPU-based HPA requires
appropriate CPU requests and usable resource metrics. Scaling is not instant:
metrics must be collected, pods scheduled, images pulled, initialization
completed, and readiness established.

HPA cannot fix a saturated MongoDB database, insufficient cluster capacity,
slow initialization, inefficient queries, or an incorrect target metric.

## 4. Rolling updates and graceful shutdown

The example Deployment uses:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

`maxSurge: 1` permits one additional pod during rollout. `maxUnavailable: 0`
asks the Deployment controller not to intentionally reduce available replicas
below the desired count during the normal rollout. This requires enough cluster
capacity and correct readiness checks; it is not an unconditional zero-downtime
guarantee.

Zero-downtime releases also require:

1. Accurate readiness checks.
2. Sufficient replicas and cluster capacity.
3. Graceful shutdown and a suitable termination grace period.
4. Compatibility between old and new application versions.
5. A safe database migration strategy.
6. Continuous error, latency, and business-metric monitoring.
7. A tested rollback procedure.

## Five final principles

1. Startup probes protect slow initialization.
2. Readiness controls whether a pod receives Service traffic.
3. Liveness detects failures that warrant restarting a container.
4. Requests and limits influence scheduling, throttling, and memory failure behavior.
5. Reliable releases must be verified with user-facing and business metrics.

**Next syllabus lesson — Day 65:** Kubernetes production troubleshooting:
CrashLoopBackOff, OOMKilled, Pending Pods, ImagePullBackOff, DNS failures, and
Service connectivity.
