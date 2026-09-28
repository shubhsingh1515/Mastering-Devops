# MERN Production Alert Catalog

This catalog is a starting point for production alert design. Thresholds must be tuned against real traffic, service-level objectives and historical incident data.

## Alert Design Rules

- Alert on a condition that requires action.
- Include environment, service, time window and severity.
- Add a traffic or volume guard where percentages could be misleading.
- Link to the dashboard and the relevant runbook.
- Name the responding team and first investigation step.
- Review noisy alerts after every incident or alert-tuning exercise.

## 1. Production API 5xx Rate

**Condition:** Production API 5xx rate is greater than 5% for 5 consecutive minutes and request volume is at least 100 requests per minute.

**Severity:** Critical when customer-facing traffic is affected.

**Who responds:** On-call application or platform engineer.

**Initial investigation:** Confirm the metric and time range, check the deployment timeline, identify affected routes and search logs by `status=500`, `event` and `errorType`.

**Possible recovery:** Stop the rollout, disable an isolated feature or roll back the release. Check dependencies and validate recovery with metrics and representative requests.

## 2. Production API p95 Latency

**Condition:** API p95 latency is greater than 1 second for 10 consecutive minutes with normal or elevated traffic.

**Severity:** Warning initially; critical when paired with errors or business-impact signals.

**Who responds:** On-call application engineer, with platform support if resource saturation is suspected.

**Initial investigation:** Compare p50, p95 and p99 latency by route. Search for slow database operations, external provider timeouts, retries and the deployment version.

**Possible recovery:** Reduce load, pause the rollout, disable an expensive feature, increase capacity if justified or roll back a release that introduced the regression.

## 3. Health or Readiness Failures

**Condition:** A production instance fails readiness checks continuously for 3 minutes, or the healthy instance count falls below the availability target.

**Severity:** Critical when capacity or availability is at risk.

**Who responds:** Platform or on-call engineer.

**Initial investigation:** Determine whether failures affect one instance or the whole service. Inspect container startup logs, dependency checks, configuration, DNS, network policy and resource limits.

**Possible recovery:** Remove the unhealthy instance from traffic, restore the last known-good configuration, replace the instance or roll back the deployment.

## 4. MongoDB Connectivity Failures

**Condition:** Repeated MongoDB connection or server-selection failures occur across the API for 5 minutes, or the connection pool cannot maintain the required healthy connections.

**Severity:** Critical for order, checkout or authentication paths; warning for an isolated non-critical worker.

**Who responds:** Application on-call with database/platform support.

**Initial investigation:** Search for `MongoServerSelectionError`, check the deployment version, verify `MONGO_URI`, service name, DNS, credentials, network access, TLS settings and MongoDB health.

**Possible recovery:** Correct configuration, restore network access, fail over according to the database runbook or roll back the release. Never print the full connection string while investigating.

## 5. Checkout Success Rate

**Condition:** Checkout success falls below the agreed baseline, for example below 95% for 10 minutes, with a minimum transaction volume.

**Severity:** Critical because it represents direct business impact.

**Who responds:** Application/product on-call, with payment and platform owners as needed.

**Initial investigation:** Compare checkout steps, payment-provider responses, API errors, release versions and affected regions or customer segments. Use request IDs to follow representative failed transactions.

**Possible recovery:** Disable the isolated feature, switch to an approved fallback provider, pause promotion or roll back the release. Confirm that retries do not create duplicate orders or charges.

## Alert Review Checklist

- [ ] Is the condition actionable?
- [ ] Is the threshold based on user or operational impact?
- [ ] Is there a minimum traffic or event-volume guard?
- [ ] Is the duration long enough to avoid transient noise?
- [ ] Is severity justified?
- [ ] Is the responder named?
- [ ] Are dashboard and runbook links available?
- [ ] Does the alert explain what to inspect first?
- [ ] Has the alert been tested with a safe simulation?
- [ ] Is the alert reviewed for noise after use?