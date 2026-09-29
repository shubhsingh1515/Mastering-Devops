# MERN Production Incident Alert Catalog

These alerts are starting points. Tune thresholds against real traffic, service-level objectives, historical behavior and business impact.

## Alert Design Rules

- Alert on a condition that requires action.
- Include environment, service, route, version and time window.
- Add a traffic or event-volume guard where percentages could mislead.
- Include the first investigation step and link to the incident runbook.
- Prefer user-impact and business-impact signals over isolated infrastructure noise.
- Record alert acknowledgements and mitigation actions in the incident timeline.
- Review noisy alerts after every incident.

## 1. Production API 5xx Rate

**Condition:** Production API 5xx rate is greater than 5% for 5 consecutive minutes and traffic is at least 100 requests per minute.

**Severity:** Critical when customer-facing traffic is affected.

**Who responds:** Application or platform on-call engineer.

**Initial investigation:** Confirm the time range, affected routes, deployment version and structured events with `status=500`, `event` and `errorType`.

**Possible mitigation:** Stop the rollout, disable an isolated feature or roll back when application and database compatibility are confirmed. Verify error rate and user-facing flows afterward.

## 2. Production API p95 Latency

**Condition:** API p95 latency is greater than 1 second for 10 consecutive minutes with normal or elevated traffic.

**Severity:** Warning initially; critical when paired with checkout failures, timeouts or rising 5xx responses.

**Who responds:** Application on-call, with platform support when saturation is suspected.

**Initial investigation:** Compare p50, p95 and p99 by route. Check slow database operations, external-provider latency, retries, connection pools and recent changes.

**Possible mitigation:** Pause the rollout, disable an expensive feature, reduce load or increase capacity when justified. Roll back a release only after checking compatibility.

## 3. Health or Readiness Failure

**Condition:** A production instance fails readiness checks continuously for 3 minutes, or healthy capacity falls below the availability target.

**Severity:** Critical when traffic cannot be served safely.

**Who responds:** Platform or on-call engineer.

**Initial investigation:** Determine whether one instance or the whole service is affected. Inspect startup logs, dependency checks, configuration, DNS, network policy and resource limits.

**Possible mitigation:** Remove the unhealthy instance from traffic, replace it, restore known-good configuration or stop the rollout. Do not treat a restarted container as recovered until the readiness check and user flow pass.

## 4. MongoDB Connectivity Failure

**Condition:** MongoDB connection or server-selection failures occur across the API for 5 minutes, or the connection pool cannot maintain the required healthy connections.

**Severity:** Critical for orders, checkout or authentication; warning for an isolated non-critical worker.

**Who responds:** Application on-call with database or platform support.

**Initial investigation:** Search for `MongoServerSelectionError`, compare the deployment version, inspect the configured hostname without exposing credentials, and check DNS, network access, TLS, credentials and MongoDB health.

**Possible mitigation:** Correct invalid configuration, restore network access, fail over according to the database procedure or roll back when safe. Never print the complete connection string into shared output.

## 5. Checkout Success Rate

**Condition:** Checkout success falls below the agreed baseline, for example below 95% for 10 minutes with a minimum transaction volume.

**Severity:** Critical because this is direct business impact.

**Who responds:** Application and product on-call, with payment and platform owners as needed.

**Initial investigation:** Compare checkout steps, payment-provider responses, timeout logs, release versions, feature flags and affected regions. Trace representative failures using request IDs.

**Possible mitigation:** Disable the isolated payment feature, switch to an approved fallback, stop promotion or roll back when safe. Confirm that retries have not created duplicate orders or charges.

## Alert Review Checklist

- [ ] Is the condition actionable?
- [ ] Does it include duration and traffic or volume context?
- [ ] Is severity based on user or business impact?
- [ ] Is the responder named?
- [ ] Is the first investigation step documented?
- [ ] Does it link to a dashboard and `INCIDENT-RUNBOOK.md`?
- [ ] Does it include deployment or feature-flag context?
- [ ] Has it been tested with a safe simulation?
- [ ] Was it reviewed for noise after use?
