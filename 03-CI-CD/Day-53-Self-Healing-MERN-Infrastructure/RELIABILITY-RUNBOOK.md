# MERN Self-Healing Reliability Runbook

This runbook is for a customer-facing Dockerized MERN API. Adapt service names, dashboards, escalation contacts, thresholds and deployment commands to the project. Never store secrets, complete connection strings or customer data in this file.

## Runbook Header

```text
Incident ID:
Incident commander:
Technical/platform owner:
Start time UTC:
Detection source:
Affected service or instance:
Current severity:
Incident channel:
Dashboard:
Release version:
Runbook version:
```

## 1. Liveness Endpoint

### Purpose

Liveness answers whether the application process can still respond and continue running. It should be minimal and fast. It should not depend on every external service.

### Contract

```text
GET /health
Healthy:    HTTP 200
Unhealthy:  timeout, connection failure or repeated non-200 response
Body:       { "status": "ok" }
```

Do not include stack traces, credentials, database URLs, internal hostnames or customer details. A liveness endpoint should not run an expensive query on every probe.

### Action

If liveness fails repeatedly:

1. Check whether the failure affects one instance or the whole service.
2. Preserve logs, restart history and recent deployment metadata.
3. Allow the orchestrator or approved process manager to replace the instance when restart is expected to help.
4. Confirm that the replacement passes liveness and readiness.
5. Alert if healthy capacity falls below the availability target or the replacement enters a restart loop.

## 2. Readiness Endpoint

### Purpose

Readiness answers whether this instance should receive new traffic. It may check initialization state and critical dependencies for the workflows served by the instance.

### Contract

```text
GET /ready
Ready:      HTTP 200
Not ready:  HTTP 503
Ready body: { "status": "ready", "dependencies": { "mongodb": "ok" } }
Not-ready:  { "status": "not_ready", "dependencies": { "mongodb": "unavailable" } }
```

The response must remain safe to expose to the load balancer and operators. Do not include the full MongoDB URI, passwords, tokens or detailed internal errors.

### Action

When readiness fails:

1. Remove the instance from new load-balancer traffic.
2. Keep the process running unless there is evidence that restarting will repair the instance.
3. Check critical dependency access, initialization, configuration, DNS, network policy and recent changes.
4. Monitor healthy capacity and protect the remaining replicas from overload.
5. Return the instance only after readiness passes continuously for the agreed stability window.

## 3. Dependency Behavior

Classify dependencies by workflow rather than assuming every service is equally critical:

| Dependency | Example behavior |
|---|---|
| MongoDB | Usually critical for users, orders and authentication. Fail readiness for affected API work when it cannot be used safely. |
| Payment provider | Critical for checkout, but not necessarily product browsing. Use a safe fallback or protect checkout specifically. |
| Redis | A cache failure may permit degraded operation; a session-store failure may be critical if sessions cannot be validated. |
| Email | Often asynchronous and non-critical to accepting an order. Queue and retry rather than taking the whole API out of service. |

Document the dependency contract for each important route. Optional dependencies should not unnecessarily make all traffic unready. Critical dependencies should not be hidden behind a misleading `200 OK`.

### Dependency outage decision

```text
Dependency unavailable
        |
        +--> Critical to this workflow?
        |        |
        |        +--> Yes: fail readiness or protect the affected route
        |        |
        |        +--> No: degrade, queue, retry and alert
        |
        +--> Is restart likely to restore access?
                 |
                 +--> No: do not restart every instance
```

## 4. Restart Policy

A typical Compose policy is:

```yaml
services:
  api:
    restart: unless-stopped
```

Use `on-failure` when only non-zero process exits should restart. Use `unless-stopped` when the service should return after daemon or host recovery unless an operator intentionally stopped it. Choose based on the runtime and operational procedure.

A restart policy helps with process crashes. It does not detect all application failures and does not repair:

- A process that remains alive but returns HTTP 500.
- Invalid environment variables.
- A bad application release.
- A MongoDB outage.
- Database corruption or incompatible data.
- A payment-provider incident.

### Restart-loop controls

For repeated crash and restart behavior:

1. Capture startup logs and exit codes.
2. Check image tag, configuration presence and dependency initialization.
3. Inspect restart count and crash timing.
4. Pause the rollout if a new version is involved.
5. Apply backoff or stop repeated attempts according to the platform procedure.
6. Fix the cause or return to a compatible known-good version.
7. Confirm stable liveness and readiness before restoring traffic.

## 5. Load-Balancer Behavior

The load balancer should use readiness, not only container state:

```text
API-1 /ready -> 200  -> receive traffic
API-2 /ready -> 503  -> no new traffic
API-3 /ready -> 200  -> receive traffic
```

When an instance fails readiness:

- Stop routing new requests to it.
- Allow existing requests to drain when supported.
- Preserve enough healthy capacity for current traffic.
- Continue probing it so it can recover automatically.
- Reintroduce it only after readiness and stability checks pass.

A readiness failure is traffic isolation, not proof that the container must be destroyed. If healthy capacity is too low, scale or activate the approved availability procedure.

## 6. Graceful Shutdown

Use this sequence during deployment or replacement:

```text
Remove instance from traffic
        |
        v
Stop accepting new work
        |
        v
Finish active requests
        |
        v
Stop consumers and close connections
        |
        v
Exit before termination deadline
```

Node.js services should handle `SIGTERM` and `SIGINT`. The shutdown handler should be idempotent, stop new HTTP work, allow active requests to finish, close MongoDB/Redis connections, stop queue consumers and exit. Configure a termination grace period longer than the normal request-drain time.

Conceptual implementation:

```js
let shuttingDown = false;

async function shutdown(signal) {
  if (shuttingDown) return;
  shuttingDown = true;

  server.close(async () => {
    await mongoose.connection.close();
    process.exit(0);
  });

  setTimeout(() => process.exit(1), 10000).unref();
}

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));
```

Test shutdown with active requests and confirm that in-flight work is not abruptly terminated.

## 7. Failure Scenarios

### Scenario A: API process crashes

**Signal:** Container exits; restart count increases.  
**Response:** Preserve logs, inspect exit code and recent changes, allow approved restart or replacement, then verify liveness, readiness and user flow.

### Scenario B: MongoDB is unavailable to one API instance

**Signal:** `/health` returns 200, `/ready` returns 503, MongoDB connection errors appear.  
**Response:** Remove the instance from traffic. Do not restart every API replica. Investigate DNS, network, TLS, credentials, configuration and MongoDB health.

### Scenario C: All replicas fail readiness

**Signal:** Healthy capacity falls below target and customer errors rise.  
**Response:** Declare or escalate the incident, protect remaining capacity, determine whether the shared dependency, configuration or deployment is responsible, and use the approved mitigation procedure.

### Scenario D: New version starts but is not ready

**Signal:** `/health` is 200 while `/ready` remains 503.  
**Response:** Do not send production traffic to it. Inspect startup logs and dependency initialization. Stop the rollout if the stability deadline is exceeded.

### Scenario E: Restart loop after deployment

**Signal:** The same container repeatedly starts and exits.  
**Response:** Pause the rollout, preserve evidence, inspect image/configuration/schema compatibility and apply a controlled rollback or fix-forward procedure.

### Scenario F: Email service is unavailable

**Signal:** Email delivery failures while order writes succeed.  
**Response:** Keep order handling available if the contract permits, queue and retry email, alert the owner and avoid making the whole API unready for an optional dependency.

### Scenario G: Graceful shutdown exceeds its deadline

**Signal:** Active requests or consumers remain when the termination grace period expires.  
**Response:** Inspect long-running work, drain behavior, connection closure and queue acknowledgement. Increase the deadline only after understanding the cause.

## 8. Recovery Procedure

### Before changing production

- Confirm whether the failure is process, application, dependency, deployment or data related.
- Capture timestamps, logs, status codes, restart count and release metadata.
- Identify healthy capacity and current user impact.
- Avoid exposing secrets or personal data in evidence.

### Recovery sequence

1. Confirm `/health` and `/ready` for every replica.
2. Isolate unready instances from new traffic.
3. Stop or pause an unsafe rollout.
4. Select the least risky action: dependency repair, configuration correction, feature disablement, scale-out, replacement or compatible rollback.
5. Allow graceful shutdown for instances being replaced.
6. Wait for the replacement to pass liveness and readiness.
7. Introduce traffic gradually when possible.
8. Monitor error rate, latency, restarts, dependency health and logs.
9. Validate a safe representative user workflow.
10. Check business signals such as checkout success, order completion and duplicate-charge rate.
11. Record the timeline, decision, owner and result.

### Recovery evidence

Do not declare recovery because Docker says `running`. Require:

```text
[ ] /health returns 200
[ ] /ready returns 200 continuously
[ ] Load balancer routes only to ready instances
[ ] Restart count is stable
[ ] 5xx rate is at baseline
[ ] Latency is at baseline
[ ] Critical dependencies are healthy
[ ] Representative user flow succeeds
[ ] No duplicate orders, charges or data corruption
[ ] Monitoring remains stable through the observation window
```

## Incident Timeline

```text
[time] - Failure detected
[time] - Liveness/readiness and healthy capacity confirmed
[time] - Unready instance removed from traffic
[time] - Deployment or dependency correlation completed
[time] - Recovery action applied
[time] - Replacement passed liveness and readiness
[time] - Traffic restored gradually
[time] - Technical and business recovery verified
```

## Corrective Action Register

| Action | Owner | Priority | Due date | Verification |
|---|---|---|---|---|
| Add separate liveness and readiness probes |  | High |  | Dependency outage does not restart all replicas |
| Add restart-loop alert with backoff |  | High |  | Repeated crashes page the owner once with context |
| Test graceful shutdown with active requests |  | High |  | Requests drain before termination |
| Add readiness gate to rolling deployment |  | High |  | New version receives traffic only after readiness |
| Document dependency criticality by route |  | Medium |  | Optional failure does not remove unrelated traffic |
| Add business recovery checks |  | Medium |  | Checkout and order metrics are included in verification |
