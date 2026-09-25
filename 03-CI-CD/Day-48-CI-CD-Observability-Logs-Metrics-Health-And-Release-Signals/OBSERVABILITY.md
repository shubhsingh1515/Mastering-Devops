# MERN Release Observability Plan

## Purpose

This document defines the minimum observability contract for a production MERN release. It is intended for dashboards, alerts, deployment gates and incident response.

## Health Endpoints

### Liveness

`GET /health` answers whether the Node/Express process is alive and able to respond. It should be cheap and should not depend on a slow external service.

Example response:

```json
{
  "status": "ok",
  "service": "orders-api",
  "release_version": "2.4.0"
}
```

### Readiness

`GET /ready` answers whether the instance can receive production traffic. It should verify required dependencies such as MongoDB connectivity and return a failure status when the instance cannot serve normal requests safely.

Example response:

```json
{
  "status": "ready",
  "checks": {
    "mongodb": "ok"
  }
}
```

Do not use a readiness check as an expensive synthetic transaction. Keep dependency checks bounded by a timeout.

## Structured Application Logs

Every request and important background operation should include:

- timestamp in UTC
- log level
- service name
- release version
- environment
- request or correlation ID
- HTTP method and route
- status code
- duration in milliseconds
- stable error code when applicable

Example:

```json
{
  "timestamp": "2026-09-25T22:04:15.000Z",
  "level": "error",
  "service": "orders-api",
  "environment": "production",
  "release_version": "2.4.0",
  "request_id": "7fa92c",
  "method": "POST",
  "route": "/api/orders",
  "status_code": 504,
  "duration_ms": 5000,
  "error_code": "PAYMENT_TIMEOUT"
}
```

Never log passwords, access tokens, payment card information or complete sensitive request bodies. Use redaction and stable identifiers.

## Request and Correlation IDs

1. Accept a trusted request ID from the edge only when it meets format and length rules.
2. Otherwise generate a new random request ID.
3. Return it in the response header for support investigation.
4. Include it in every log generated during the request.
5. Propagate it to downstream services where supported.

A support report containing `request_id=7fa92c` should be searchable across Nginx, the API and dependency logs.

## Metrics

| Metric | Type | Example release signal |
|---|---|---|
| Request rate | Traffic | Sudden drop or unexpected spike |
| 4xx rate | Technical | Unexpected client or routing failures |
| 5xx rate | Technical | Above 1% for 3 minutes |
| p95 latency | Technical | Above 800 ms for 5 minutes |
| p99 latency | Technical | Sustained tail-latency increase |
| Availability | Technical | Below the agreed SLO |
| CPU and memory | Saturation | Sustained resource pressure |
| Container restarts | Saturation | Any unexpected restart loop |
| MongoDB latency | Dependency | Query latency above baseline |
| MongoDB errors | Dependency | Connection or operation failures |
| Checkout success | Business | Below 98% or baseline by agreement |
| Payment completion | Business | Meaningful increase in failures |
| Order creation | Business | Drop after deployment |
| Cart abandonment | Business | Significant increase after release |

Measure technical metrics by service, route, release version and canary cohort where possible. Measure business metrics with privacy-preserving aggregation.

## Dashboard Layout

```text
MERN Production - v2.4.0

Availability             99.95%
5xx error rate             0.4%
p95 API latency            420 ms
p99 API latency            1.1 s
Request rate              1,200 req/min
MongoDB latency             35 ms
MongoDB errors               0.1%
Checkout success            98.7%
Container restarts             0

Deployment marker: v2.4.0 at 22:00 UTC
```

Place deployment markers on every dashboard that is used for rollout decisions.

## Release Gates

Before increasing exposure, require:

```text
5xx rate < 1% for 5 consecutive minutes
p95 latency below 800 ms
readiness checks passing
no restart loop
MongoDB errors at or near baseline
checkout success above 98%
minimum request sample reached
```

These are examples. Tune them using normal traffic and service-level objectives. A gate should define the time window, sample size, baseline and action.

If the gate fails:

1. Stop increasing exposure.
2. Keep the known-good version serving most traffic.
3. Disable an isolated feature if that is sufficient.
4. Roll back or isolate the candidate when the release is broadly unhealthy.
5. Notify the on-call engineer.
6. Preserve the evidence and open an incident investigation.

## Incident Evidence

Preserve:

- deployment ID, image digest and release version
- exact deployment and rollback timestamps
- dashboard screenshots or exported time series
- structured logs and request IDs
- traces for representative failed requests
- database and external dependency metrics
- feature flag changes and audit records
- commands executed and their results

## Review Checklist

- [ ] Liveness endpoint exists and is cheap.
- [ ] Readiness includes required dependency checks.
- [ ] Logs are structured and searchable.
- [ ] Request IDs are propagated.
- [ ] Sensitive data is redacted.
- [ ] Request rate, 4xx, 5xx and latency are available.
- [ ] CPU, memory and restarts are available.
- [ ] MongoDB health and latency are available.
- [ ] Business KPIs are available.
- [ ] Deployment markers are visible.
- [ ] Rollout thresholds are documented.
- [ ] Stop and rollback actions are tested.
