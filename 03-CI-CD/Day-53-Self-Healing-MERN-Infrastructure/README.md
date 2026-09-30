# DevOps Mentorship Program - Day 53

## Phase 3: Production Operations

### Self-Healing MERN Infrastructure: Health Checks, Restart Policies and Auto-Recovery

**Duration:** 25-30 minutes  
**Focus:** Liveness, readiness, dependency health, restart policies, load balancing, graceful shutdown and failure isolation

Yesterday, Day 52, you learned how to detect, triage, mitigate, investigate, recover and learn from production incidents. Today we move one level deeper:

> How can the platform detect common failures and recover automatically before an engineer has to intervene?

Self-healing does not eliminate incident response. It reduces the blast radius of predictable failures and preserves human attention for failures that need judgment.

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain the difference between liveness and readiness.
- Explain why health checks are different from process status.
- Design `/health` and `/ready` for an Express API.
- Choose an appropriate Docker restart policy.
- Explain why restart policies do not replace application monitoring.
- Design a self-healing multi-instance MERN service.
- Explain load-balancer health-aware routing.
- Implement graceful shutdown for Node.js.
- Recognize restart loops and dependency failure amplification.
- Explain safe behavior during rolling deployments.
- Troubleshoot an unhealthy container in an interview or incident.

---

## 2. Container Running Does Not Mean Application Healthy

Docker may report:

```text
CONTAINER       STATUS
mern-api        Up 2 hours
```

Inside the process, however:

```text
Node process       OK
MongoDB access     FAILED
API requests       FAILED
```

The important distinction is:

```text
Container running != Application healthy
```

A process can be alive while returning HTTP 500 responses, waiting on a broken dependency, serving stale data or failing every business operation. Platform automation needs application-level signals, not only process-level status.

---

## 3. Liveness and Readiness

### Liveness

Liveness answers:

> Is this process alive enough to continue running?

A liveness check should be minimal. It should normally verify that the HTTP process can respond and that the event loop is not permanently stuck. It should not depend on every external service, because a temporary database outage should not necessarily restart every API instance.

When liveness fails repeatedly, the orchestrator may restart or replace the instance:

```text
API process
    |
    v
Liveness fails repeatedly
    |
    v
Restart or replace instance
```

### Readiness

Readiness answers:

> Should this instance receive traffic right now?

Readiness can include critical dependency and initialization checks. When readiness fails, the load balancer should stop sending new requests to that instance. The process may remain alive and recover when the dependency returns.

```text
Readiness fails
    |
    v
Remove instance from traffic
    |
    v
Dependency recovers
    |
    v
Readiness passes
    |
    v
Return instance to traffic
```

A readiness failure does not automatically mean the application should restart.

---

## 4. Designing MERN Health Endpoints

A practical Express API exposes two endpoints:

```text
GET /health
GET /ready
```

### `/health`: process liveness

Example response:

```json
{
  "status": "ok"
}
```

This endpoint should be fast, deterministic and free from sensitive details. It should not perform an expensive MongoDB query on every probe.

### `/ready`: traffic eligibility

Example response:

```json
{
  "status": "ready",
  "dependencies": {
    "mongodb": "ok"
  }
}
```

Return HTTP `200` only when the application has initialized and its critical dependencies are usable. Return HTTP `503` when it should not receive traffic. Do not expose connection strings, credentials, internal hostnames or detailed error stacks.

A conceptual Express implementation is:

```js
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

app.get('/ready', async (req, res) => {
  const mongoReady = mongoose.connection.readyState === 1;

  if (!mongoReady) {
    return res.status(503).json({
      status: 'not_ready',
      dependencies: { mongodb: 'unavailable' }
    });
  }

  return res.status(200).json({
    status: 'ready',
    dependencies: { mongodb: 'ok' }
  });
});
```

Adapt the check to the driver and application. A readiness check must represent the work the service actually promises to perform.

---

## 5. Dependency Classification

Not every dependency should make the whole API unready.

Suppose the API uses MongoDB, a payment provider, Redis and email:

| Dependency | Typical classification | Reason |
|---|---|---|
| MongoDB | Critical for most API routes | Orders, users and sessions may not work without it. |
| Payment provider | Critical for checkout only | Product browsing may still work. |
| Redis | Application-specific | A cache outage may reduce performance but not correctness. |
| Email service | Often non-critical | Orders may be accepted while email delivery is retried asynchronously. |

A single global readiness result may be too coarse for a large application. Use route-level dependency handling or graceful degradation where appropriate. Do not make readiness so complicated that an optional feature takes the entire service out of rotation.

The decision should be explicit:

```text
Critical dependency fails
    -> readiness fails or affected route is protected
Optional dependency fails
    -> degrade, queue, retry or alert
```

---

## 6. Docker Restart Policies

Docker restart policies act when the container process exits:

- `no`: do not restart automatically.
- `on-failure`: restart when the process exits with a non-zero status.
- `always`: restart whenever the container stops.
- `unless-stopped`: restart after failures or daemon restarts unless an operator explicitly stopped it.

Example Compose configuration:

```yaml
services:
  api:
    image: example/mern-api:1.0.0
    restart: unless-stopped
```

For a process that repeatedly crashes:

```text
Start -> Crash -> Restart -> Crash -> Restart
```

This is a restart loop, not recovery. Use logs, restart counters, backoff, alerting and deployment history to distinguish successful recovery from repeated failure.

Restart policies are insufficient when the process remains alive but returns errors. They also cannot repair invalid configuration, a broken deployment, database corruption or an external outage.

---

## 7. Self-Healing Architecture

A high-availability MERN API can use multiple replicas behind a load balancer:

```text
                         Load Balancer
                         /     |     \
                        /      |      \
                    API-1    API-2    API-3
                      |        |        |
                      +--------+--------+
                               |
                            MongoDB
```

Each replica exposes `/health` and `/ready`:

```text
API-1 /ready -> 200   receives traffic
API-2 /ready -> 503   removed from new traffic
API-3 /ready -> 200   receives traffic
```

If API-2 crashes, the restart policy or orchestrator replaces it while API-1 and API-3 continue serving. If API-2 remains alive but loses MongoDB connectivity, readiness can remove it from traffic without restarting every replica.

Self-healing should reduce blast radius:

| Failure | Preferred first response |
|---|---|
| Process crash | Restart or replace the instance. |
| Dependency outage | Isolate from traffic and monitor dependency recovery. |
| Bad deployment | Stop rollout, disable feature or use a safe rollback. |
| Configuration error | Correct configuration and redeploy deliberately. |
| Data corruption | Preserve evidence and use controlled recovery. |

---

## 8. Readiness During Rolling Deployments

A new version should not receive production traffic merely because its process started:

```text
Start v2
   |
   v
/health = 200
/ready  = 503
   |
   v
No production traffic yet
```

Only after initialization and critical dependency checks pass should the load balancer route requests to v2:

```text
/ready = 200
   |
   v
Introduce traffic gradually
```

A safe rolling sequence is:

1. Start one new instance.
2. Wait for liveness and readiness to pass.
3. Send a controlled amount of traffic.
4. Observe errors, latency, logs and business metrics.
5. Continue replacing old instances only when signals remain healthy.
6. Stop the rollout if the new version becomes unhealthy.

This protects users from receiving traffic before the replacement is actually ready.

---

## 9. Graceful Shutdown

When an instance is being replaced, it should stop accepting new work, finish active requests, close resources and then exit:

```text
Remove from traffic
       |
       v
Stop accepting new requests
       |
       v
Finish active requests
       |
       v
Close server and database connections
       |
       v
Exit
```

Conceptual Node.js implementation:

```js
const server = app.listen(PORT);

async function shutdown(signal) {
  console.log(`${signal} received; shutting down`);

  server.close(async () => {
    await mongoose.connection.close();
    process.exit(0);
  });

  setTimeout(() => process.exit(1), 10000).unref();
}

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));
```

Production code should also stop background consumers, reject new queue work, close Redis connections and protect against duplicate shutdown execution. The termination grace period must be longer than the normal time needed to finish active requests.

---

## 10. Failure Drill

Scenario:

```text
API-1 -> healthy
API-2 -> /health 200, /ready 503
API-3 -> healthy
```

Correct response:

1. API-2 should receive no new traffic.
2. API-2 does not necessarily need a restart.
3. The load balancer should retain API-1 and API-3 in rotation.
4. Investigate MongoDB connectivity, configuration, DNS, network policy and recent changes.
5. Monitor API-2 and the dependency.
6. Return API-2 to traffic only after `/ready` is continuously healthy.

Do not confuse CPU and memory usage with application readiness. An instance can use 45% CPU and 50% memory while every database-backed request fails.

---

## 11. Core Review Questions

### Why is restarting not always correct?

If MongoDB is unavailable, restarting all APIs does not restore MongoDB. It may amplify the outage, destroy evidence and create a restart storm. Isolate the unready instances and let them recover when the dependency becomes usable.

### Why are health checks different from process status?

Process status only proves that a process exists. Health checks test application behavior and traffic eligibility.

### What does self-healing mean?

Self-healing means automatically recovering from specific known failure conditions, such as replacing a crashed instance or removing an unready instance from traffic. It does not mean every incident is solved without humans.

---

## 12. Summary

The production reliability model is:

```text
Process failure
    -> restart may help

Application not ready
    -> remove from traffic

Dependency failure
    -> isolate, degrade where safe and monitor

Bad deployment
    -> stop rollout, disable feature or recover safely

Data problem
    -> controlled investigation and recovery
```

The most important lesson is:

> Self-healing should reduce blast radius, not blindly restart everything.
