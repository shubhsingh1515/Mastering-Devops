# DevOps Mentorship Program - Day 48

## Phase 3: CI/CD

### CI/CD Observability: Logs, Metrics, Health and Release Signals

**Level:** Intermediate -> Professional 
**Focus:** Deployment observability, release monitoring, MERN production incidents and interview troubleshooting

Yesterday, Day 47, you learned how to make releases safer with feature flags, gradual rollout, metrics and rollback. Today we answer the next production question:

> How do we know whether a deployment is actually healthy?

A pipeline can report `Deploy successful` while users experience checkout failures, slow API responses, 5xx errors or database timeouts. CI/CD tells us whether the deployment process completed. Observability tells us whether the running system is behaving correctly.

```text
Git
  |
  v
CI/CD deployment
  |
  v
Running production system
  |
  v
Logs + metrics + traces + business evidence
  |
  v
Continue, pause or roll back
```

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain logs, metrics and traces at a high level.
- Distinguish infrastructure health from application and business health.
- Define useful deployment metrics for a MERN application.
- Explain error rate, latency, percentiles and saturation.
- Design a release-monitoring strategy.
- Use liveness and readiness checks correctly.
- Explain structured logs and correlation IDs.
- Decide when to stop or roll back a release.
- Troubleshoot a production deployment using evidence.
- Answer observability questions in DevOps interviews.

---

## 2. Why Deployment Success Is Not Application Health

Imagine CI reports:

```text
Build       OK
Tests       OK
Docker scan OK
Deploy      OK
```

Five minutes later, customers report that checkout is extremely slow. The deployment command succeeded, but the application may still be degraded.

```text
CI/CD
  |
  v
Deploy
  |
  v
Observe the running release
  |
  v
Compare with the baseline
  |
  v
Make a release decision
```

A production release should be evaluated against evidence from the running system, not only against the exit code from a deployment tool.

---

## 3. The Three Pillars of Observability

### Logs: What happened?

Logs are detailed event records. A useful order failure might contain:

```text
2026-09-25T22:04:15Z level=error service=api \
  request_id=7fa92c route=/api/orders \
  error=payment_timeout duration_ms=5000
```

Good logs answer what happened, when it happened, which service was involved and which request was affected. They must not expose passwords, payment credentials or unnecessary personal data.

### Metrics: How much and how often?

Metrics are numeric measurements over time:

```text
5xx rate       = 4.2%
p95 latency    = 820 ms
request rate   = 1,200 requests/minute
CPU            = 72%
```

Metrics are excellent for dashboards, alerts, release gates and before/after comparisons.

### Traces: Where did the time go?

A trace follows one request across services:

```text
Browser      5 ms
  |
Nginx       10 ms
  |
Node API    40 ms
  |
MongoDB    700 ms
```

Traces help identify whether latency comes from the API, a database query, a payment provider or another dependency.

---

## 4. The Four Golden Release Signals

For a MERN API, start with four signals:

| Signal | Question | Example |
|---|---|---|
| Request rate | How much traffic is arriving? | 1,200 requests/minute |
| Error rate | Are requests failing? | 5xx rate = 0.4% |
| Latency | Are requests becoming slow? | p95 = 420 ms |
| Saturation | Are resources or dependencies near capacity? | DB pool = 85% used |

Also monitor CPU, memory, restarts, network, MongoDB connections, query latency and external services. No single metric is enough. CPU can be normal while a bad query makes checkout unusable.

---

## 5. Error Rate

If an API receives 10,000 requests and 200 fail:

```text
error rate = 200 / 10,000 = 2%
```

Compare the release to the baseline:

```text
Before deployment: 0.3%
After deployment:  2.0%
```

That is a strong release signal, but investigate scope before acting. Identify the affected endpoints, users, status codes, error types and business impact. A short-lived 404 increase is different from sustained payment-related 5xx errors.

---

## 6. Latency and Percentiles

Average latency can hide the slow requests that matter most. If most requests take 100 ms but some take 4 seconds, the average may look acceptable while real users wait.

- **p50:** the median request latency.
- **p95:** approximately 95% of requests complete at or below this value.
- **p99:** approximately 99% of requests complete at or below this value.

Example:

```text
p50 = 120 ms
p95 = 350 ms
p99 = 1.8 s
```

The p99 shows that a small but important group of users is experiencing very slow requests.

### MERN example

```text
POST /api/orders
Before deployment: p95 = 420 ms
After deployment:  p95 = 2.8 s
CPU and memory:    normal
```

Possible causes include a slow MongoDB query, a missing index, an N+1 query, new application logic or a slow payment provider. Normal infrastructure metrics do not prove that the application is healthy.

---

## 7. Liveness, Readiness and Health

**Liveness** asks whether the process is alive. **Readiness** asks whether this instance can safely receive traffic.

```text
Process running
      |
      v
Liveness passes
      |
      v
Required dependencies available
      |
      v
Readiness passes
      |
      v
Traffic is allowed
```

A basic endpoint that always returns this response is weak:

```json
{"status":"ok"}
```

If the API process is running but MongoDB is unavailable, the endpoint may lie. A readiness check should verify critical dependencies without becoming so expensive that it creates additional load. Keep liveness simple and use readiness for dependency-aware traffic decisions.

---

## 8. Production Logs and Correlation IDs

A production log should make incident investigation possible:

```json
{
  "timestamp": "2026-09-25T22:04:15.000Z",
  "level": "error",
  "service": "orders-api",
  "release_version": "2.4.0",
  "request_id": "7fa92c",
  "method": "POST",
  "route": "/api/orders",
  "status_code": 504,
  "duration_ms": 5000,
  "error_code": "PAYMENT_TIMEOUT"
}
```

Generate or accept a request ID at the edge and propagate it through Nginx, the Node/Express API, database-related logs and external service calls. When a customer reports a failed order, searching for one ID can reconstruct the request path.

Do not log access tokens, passwords, payment card data or complete sensitive request bodies. Use stable error codes and carefully selected diagnostic fields instead.

---

## 9. Deployment Markers and Baselines

Mark each deployment on your dashboards:

```text
22:00  deploy v2.4.0
       error rate rises
       p95 latency rises
```

A deployment marker lets you correlate a release with changes in error rate, latency, traffic and business outcomes. Always compare the candidate version with a pre-deployment baseline and, during a canary, compare v2 directly with the known-good v1.

Useful release signals include:

1. Availability: Is the service responding?
2. Error rate: Are requests failing?
3. Latency: Are requests becoming slow?
4. Business health: Are users completing important actions?

Examples of business health include login success, checkout success, payment completion and order creation.

---

## 10. Release Health Gates

Define release criteria before the incident:

```text
Deploy
  |
  v
Wait 5 minutes
  |
  v
Check 5xx, p95, readiness and business KPI
  |
  +--> Healthy: continue rollout
  |
  +--> Unhealthy: stop exposure, disable feature or roll back
```

Example thresholds:

```text
5xx rate          < 1%
p95 latency       < 800 ms
readiness         = passing
checkout success  > 98%
```

Thresholds require context: a minimum request count, an observation window and a comparison baseline. Add cooldowns or consecutive-failure requirements to avoid reacting to one noisy data point.

---

## 11. Incident Troubleshooting Sequence

Suppose a release changes these values:

```text
5xx rate:    0.4% -> 3.8%
p95 latency: 300 ms -> 1.7 s
CPU:         45%  -> 48%
Memory:      normal
```

Use evidence in this order:

1. Confirm the deployment timestamp.
2. Identify affected endpoints and release versions.
3. Compare before and after metrics.
4. Inspect structured application logs.
5. Check database latency, errors, indexes and connection pools.
6. Check external API latency and failures.
7. Review traces to locate time spent.
8. Compare the code and configuration changes.
9. Reproduce safely in staging.
10. Pause, disable or roll back when defined thresholds and user impact justify it.

Do not begin by restarting containers. Restarting can hide evidence and does not fix a bad query, incompatible schema or failing external dependency.

---

## 12. Practical Incident Assessment

| Signal | Before | After |
|---|---:|---:|
| 5xx error rate | 0.3% | 4.7% |
| p95 latency | 280 ms | 1.4 s |
| CPU | 45% | 47% |
| Memory | 52% | 54% |
| Checkout success | 98.8% | 82.1% |

The correct assessment is that multiple user-facing signals degraded after the deployment. Stop further rollout, reduce exposure or restore the previous version, preserve logs and metrics, and investigate. Normal CPU and memory do not override error, latency and business evidence.

---

## 13. Summary

Infrastructure health tells you whether the system is running. Business health tells you whether the system is working.

```text
What changed?
      |
What degraded?
      |
Who is affected?
      |
What evidence supports the cause?
      |
What is the safest recovery?
```

A production-grade release loop is:

```text
Deploy -> Observe -> Compare -> Decide -> Continue or recover
```
