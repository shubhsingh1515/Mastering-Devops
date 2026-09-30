# MERN Self-Healing Alert Catalog

These alerts are starting points. Tune thresholds against traffic, availability targets, dependency behavior, normal restart rates and business impact. Every alert should identify the service, instance, environment, version and runbook.

## Alert Design Rules

- Alert on conditions that require action.
- Include duration and request or event volume.
- Distinguish one bad replica from loss of total healthy capacity.
- Prefer user and business impact over isolated resource noise.
- Include the first investigation step and expected mitigation.
- Avoid paging repeatedly for the same dependency outage.
- Review alert noise after every incident.

## 1. Liveness Failure

**Condition:** An API instance fails `/health` three consecutive times or for more than 30 seconds.

**Severity:** Warning for one instance with sufficient spare capacity; critical when healthy capacity falls below target.

**Who responds:** Platform or application on-call.

**Initial investigation:** Check container status, restart count, exit code, startup logs, recent image/version and host resource pressure.

**Possible mitigation:** Replace the failed instance through the approved process. Verify liveness, readiness and representative traffic before declaring recovery.

## 2. Readiness Failure

**Condition:** An API instance returns HTTP 503 from `/ready` continuously for 3 minutes, or ready capacity falls below the service target.

**Severity:** Warning for one isolated instance; critical when traffic cannot be served safely.

**Who responds:** Application or platform on-call.

**Initial investigation:** Determine whether the cause is initialization, MongoDB connectivity, configuration, DNS, network policy or a recent deployment.

**Possible mitigation:** Keep the instance out of traffic, preserve capacity on healthy replicas and repair the underlying cause. Do not automatically restart all replicas.

## 3. Restart Loop

**Condition:** The same container restarts more than 5 times in 10 minutes or repeatedly exits with the same non-zero code.

**Severity:** Critical for a new deployment or when healthy capacity is falling.

**Who responds:** Application owner and platform on-call.

**Initial investigation:** Inspect startup logs, image digest, environment validation, schema compatibility, dependency initialization and deployment history.

**Possible mitigation:** Pause the rollout, apply backoff and use the approved compatible rollback or fix-forward procedure. Preserve evidence before cleanup.

## 4. Healthy Capacity Below Target

**Condition:** The number of ready API instances remains below the minimum required capacity for 2 minutes.

**Severity:** Critical because remaining instances may overload or fail under normal traffic.

**Who responds:** Platform and application on-call.

**Initial investigation:** Compare readiness failures, load, latency, error rate, container restarts, MongoDB health and recent changes.

**Possible mitigation:** Stop a rollout, scale healthy capacity when safe, remove bad instances, restore a dependency or use the incident recovery procedure.

## 5. MongoDB Dependency Failure

**Condition:** MongoDB server-selection or connection failures occur across an API instance for 5 minutes, or the connection pool cannot maintain required healthy connections.

**Severity:** Critical for orders, checkout and authentication; warning for an isolated non-critical workflow.

**Who responds:** Application on-call with database/platform support.

**Initial investigation:** Check the safe configuration metadata, DNS, network path, TLS, credentials validity without printing them, connection-pool state and MongoDB health.

**Possible mitigation:** Fail readiness for affected instances, restore network or database health, fail over according to the database procedure or correct configuration. Do not restart every API container by default.

## 6. Graceful Shutdown Timeout

**Condition:** More than 1% of shutdowns exceed the termination grace period or active requests are terminated during replacement.

**Severity:** Warning initially; critical when it causes dropped requests, duplicate work or failed deployments.

**Who responds:** Application owner.

**Initial investigation:** Review SIGTERM logs, active request duration, server drain behavior, queue acknowledgements and database connection closure.

**Possible mitigation:** Stop the rollout if request loss is increasing, fix shutdown handling and retest with active requests before deployment resumes.

## 7. Checkout or Order Success Degradation

**Condition:** Checkout success falls below the agreed baseline, for example below 95% for 10 minutes with sufficient transaction volume.

**Severity:** Critical due to direct business impact.

**Who responds:** Application, product and platform on-call.

**Initial investigation:** Compare readiness, release version, payment-provider latency, MongoDB errors, feature flags, timeout logs and duplicate-order/charge signals.

**Possible mitigation:** Disable the isolated feature, use an approved fallback, stop promotion or roll back only when schema and payment compatibility are confirmed.

## Alert Review Checklist

- [ ] Is the condition actionable?
- [ ] Does it include duration and volume context?
- [ ] Does it identify instance, service and version?
- [ ] Is severity based on user or business impact?
- [ ] Is the responder named?
- [ ] Is the first investigation step documented?
- [ ] Does it link to `RELIABILITY-RUNBOOK.md`?
- [ ] Does it avoid paging repeatedly for the same dependency outage?
- [ ] Has it been tested with a safe simulation?
- [ ] Was it reviewed for noise after use?
