# MERN High-Availability Design

This document is the Day 54 architecture deliverable. Adapt hostnames, thresholds, dashboards, deployment commands and escalation contacts to the project. Never store credentials, complete connection strings or customer data here.

## Design Goal

The API should continue serving users when one API instance crashes, becomes unready or is replaced during a deployment. High availability is based on redundancy, health-aware routing, stateless request handling, externalized shared state, observability and tested recovery.

```text
Users
  |
  v
Load Balancer
  |
  +--> API-1
  +--> API-2
  +--> API-3
          |
          v
       MongoDB
```

## 1. API Instance Architecture

Run multiple compatible Node/Express API instances behind a load balancer. Each instance should use the same image contract, environment configuration, API behavior and compatible database schema expectations.

Each instance exposes:

```text
GET /health  -> process liveness
GET /ready   -> traffic readiness
```

The API layer should be horizontally scalable. An instance must not be the only place where a session, job state, cache entry or other required request state exists.

Recommended log identity fields:

```json
{
  "service": "mern-api",
  "environment": "production",
  "instance": "api-2",
  "version": "11.0.7",
  "requestId": "83ad91",
  "route": "/orders",
  "status": 500,
  "durationMs": 240
}
```

## 2. Load-Balancer Behavior

Use a normal distribution strategy such as round robin or least connections for healthy replicas. Use weighted routing for canary releases or unequal capacity.

The load balancer must evaluate readiness:

```text
API-1 /ready -> 200 -> receives new traffic
API-2 /ready -> 503 -> excluded from new traffic
API-3 /ready -> 200 -> receives new traffic
```

When an instance becomes unready:

1. Stop routing new requests to it.
2. Drain existing requests when supported.
3. Continue probing the instance.
4. Preserve enough healthy capacity for current load.
5. Reintroduce it only after a readiness stability window.

A running container is not automatically a healthy backend. Routing must be based on whether the instance can safely serve the required requests.

## 3. Health Checks

### Liveness: `/health`

Purpose: determine whether the process can respond and should continue running.

```text
Healthy:   HTTP 200
Body:      { "status": "ok" }
Unhealthy: timeout, connection failure or repeated non-200 response
```

Keep liveness lightweight. It should not run an expensive MongoDB query on every probe. A temporary dependency outage should not automatically restart every API instance.

### Readiness: `/ready`

Purpose: determine whether the instance should receive new traffic.

```text
Ready:      HTTP 200
Not ready:  HTTP 503
Ready body: { "status": "ready", "dependencies": { "mongodb": "ok" } }
Not-ready:  { "status": "not_ready", "dependencies": { "mongodb": "unavailable" } }
```

Return `503` when initialization or a critical dependency prevents safe service. Do not expose credentials, internal hostnames, connection strings or stack traces.

Dependency classification must follow the workflow:

| Dependency | Typical policy |
|---|---|
| MongoDB | Critical for users, orders and authentication; fail readiness for affected work when unavailable. |
| Payment provider | Critical for checkout; not necessarily critical for product browsing. |
| Redis | Optional for cache-only use; critical when it stores required sessions or locks. |
| Email | Often asynchronous; queue and retry rather than taking the whole API out of service. |

## 4. Failure Handling

### API instance crash

```text
API-2 exits
    -> health/readiness disappears
    -> load balancer stops sending traffic
    -> API-1 and API-3 continue serving
    -> orchestrator replaces API-2
    -> replacement passes health and readiness
    -> traffic is restored
```

### One instance loses MongoDB access

If MongoDB is critical for the instance's served routes, fail readiness for that instance. Investigate DNS, network policy, TLS, configuration, credentials without printing them and database health. Do not restart every API replica because restarting clients does not restore a shared database.

### Unhealthy or outdated version

Remove the instance from traffic, pause the rollout, preserve logs and compare version, image digest, configuration and schema compatibility. Use a controlled replacement, rollback or fix-forward procedure.

### Insufficient healthy capacity

Declare or escalate the incident when ready capacity falls below the service target. Protect the remaining replicas from overload, stop risky deployment activity and scale only after checking MongoDB and other downstream limits.

## 5. Session and State Strategy

The preferred design is stateless request handling:

```text
Request 1 -> API-1
Request 2 -> API-3
Request 3 -> API-2
```

All healthy instances can handle the next request. Durable application data belongs in MongoDB. Shared sessions, rate limits, locks, temporary workflows and caches belong in a suitable shared system when required.

Avoid relying on in-memory sessions. A sticky session can temporarily bind a client to one backend, but it reduces failover flexibility and loses state when that backend dies. It is acceptable only when its tradeoffs are deliberate and the session loss behavior is understood.

JWT authentication reduces some server-side session state, but it does not solve rate limiting, cache consistency, WebSockets, job state or revocation by itself.

## 6. Scaling Strategy

Scale horizontally by adding instances behind the load balancer:

```text
API-1 +--+
API-2 +--+-> Load Balancer -> Users
API-3 +--+
API-4 +--+
```

To add API-4 safely:

1. Start API-4 with the approved image and configuration.
2. Confirm `/health` returns `200`.
3. Wait for initialization and dependency checks.
4. Confirm `/ready` returns `200` continuously.
5. Run representative smoke checks.
6. Enable API-4 in the load balancer.
7. Monitor latency, errors, capacity and MongoDB connections.

Do not assume that API capacity can grow without limit. More instances can create more database connections and query pressure.

## 7. Deployment Strategy

Use rolling or canary deployment with health gates.

### Rolling deployment

```text
v1 -> v1 -> v1
v2 -> v1 -> v1   (v2 ready before traffic)
v2 -> v2 -> v1
v2 -> v2 -> v2
```

Keep enough old capacity available while each new instance starts, becomes ready and passes smoke checks. Use graceful shutdown for removed instances.

### Canary deployment

```text
v1 -> 95%
v2 -> 5%
```

Increase traffic gradually only after monitoring:

- HTTP 5xx rate.
- Request latency.
- Readiness failures.
- MongoDB and external dependency errors.
- Checkout, login and order success.
- Duplicate operations and queue failures.

If the canary degrades, set its traffic to `0%`, pause promotion and investigate. An API rollback may be unsafe after a non-backward-compatible database migration, so verify schema compatibility first.

## 8. Database Bottlenecks

MongoDB is a shared dependency and may become the limiting tier:

```text
More API replicas
    -> more connections and concurrent queries
    -> MongoDB saturation
    -> higher latency and timeouts
    -> worse user experience
```

Monitor:

- Connection pool usage and limits.
- Query latency and slow queries.
- CPU and memory.
- Disk and I/O.
- Index usage.
- Replication health and lag.
- API timeout and retry rates.

Before increasing API replicas, confirm MongoDB has capacity. If connection pressure is high, tune pool sizes, improve queries and indexes, scale or fail over the database layer according to its own runbook.

## 9. Monitoring and Observability

Dashboards should show fleet and per-instance views:

```text
request rate
5xx and 4xx rate
p50/p95/p99 latency
ready instance count
health-check failures
restart count
CPU and memory
MongoDB connections and query latency
checkout/login/order success
```

Every request and deployment event should be correlated with service, environment, instance, version and request ID. This distinguishes a fleet-wide issue from a single bad backend.

Useful alerts include:

- One instance fails readiness for a sustained period.
- Healthy capacity falls below target.
- 5xx errors cluster on one instance or version.
- Versions drift outside an approved rollout window.
- MongoDB connection pressure or latency rises.
- Canary metrics exceed baseline.
- Graceful shutdown exceeds its deadline.
- Business success rate falls.

## 10. Recovery Procedure

### Detection

1. Confirm user impact, error rate and affected routes.
2. Check fleet health and ready capacity.
3. Group errors by instance and version.

### Isolation

4. Confirm `/health` and `/ready` for every replica.
5. Remove unhealthy or outdated instances from new traffic.
6. Confirm healthy replicas can handle current load.
7. Pause a related rollout or canary.

### Investigation

8. Inspect logs, image digest, configuration metadata and restart history.
9. Check DNS, network policy, TLS and MongoDB connectivity.
10. Check MongoDB connections, latency, replication and resource pressure.
11. Avoid restarting every API replica for a shared dependency failure.

### Recovery

12. Repair or replace the affected instance through the approved process.
13. Wait for liveness and readiness to pass continuously.
14. Reintroduce the instance gradually.
15. Verify technical metrics and representative user workflows.
16. Verify business metrics such as checkout success and duplicate-order rate.
17. Record the incident evidence, cause, mitigation and follow-up action.

## Recovery Evidence Checklist

```text
[ ] Healthy replica count meets the service target
[ ] /health returns 200 for all intended replicas
[ ] /ready returns 200 continuously for the stability window
[ ] Load balancer excludes unready instances
[ ] 5xx rate is back within baseline
[ ] Latency is within the agreed target
[ ] MongoDB connections and query latency are healthy
[ ] Restart counts are stable
[ ] Representative user workflow succeeds
[ ] No duplicate orders or charges occurred
[ ] Deployment and recovery timestamps are recorded
```
