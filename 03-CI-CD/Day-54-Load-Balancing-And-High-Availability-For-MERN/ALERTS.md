# MERN High-Availability Alert Catalog

These alerts are starting points. Tune thresholds against traffic volume, availability targets, normal recovery behavior, dependency limits and business impact. Every alert should identify service, environment, instance, version and the linked runbook.

## Alert Design Rules

- Alert on conditions that require action.
- Include duration and request volume.
- Distinguish one bad replica from loss of total healthy capacity.
- Prefer user and business impact over isolated resource noise.
- Include the first investigation step and likely mitigation.
- Avoid paging repeatedly for the same dependency outage.
- Review alert noise after every incident.

## 1. Backend Readiness Failure

**Condition:** One API instance returns HTTP 503 from `/ready` continuously for 3 minutes, or ready capacity falls below the service target.

**Severity:** Warning for one isolated instance with spare capacity; critical when users may be affected.

**Initial investigation:** Compare readiness by instance, MongoDB connectivity, configuration, DNS, network policy, image version and recent deployment events.

**Mitigation:** Keep the instance out of new traffic, protect healthy capacity and repair or replace only the affected instance. Do not restart every replica automatically.

## 2. Intermittent 5xx Concentrated on One Instance

**Condition:** One instance has an error rate materially above the fleet baseline, for example 18% versus less than 1% on peers.

**Severity:** Critical when it causes user-visible failures; warning when traffic can be isolated without impact.

**Initial investigation:** Group logs by instance, version, route and request ID. Compare direct health results and deployment metadata.

**Mitigation:** Remove the instance from traffic, pause the rollout and preserve logs before repair or replacement.

## 3. Healthy Capacity Below Target

**Condition:** Ready API capacity remains below the minimum required level for 2 minutes.

**Severity:** Critical.

**Initial investigation:** Compare load, latency, 5xx rate, readiness failures, restart counts, MongoDB health and recent changes.

**Mitigation:** Stop deployments, scale healthy capacity when safe, restore dependencies and follow the recovery runbook.

## 4. Replica Version Drift

**Condition:** Instances run different versions outside an approved rolling or canary window.

**Severity:** Warning initially; critical when the older version produces errors or is incompatible with the schema.

**Initial investigation:** Check deployment controller state, image digests, readiness history and database migration compatibility.

**Mitigation:** Pause promotion, remove the unhealthy version and complete a compatible rollout or rollback.

## 5. MongoDB Connection Pressure

**Condition:** API connection pools approach the configured limit, connection errors increase or query latency exceeds the agreed threshold.

**Severity:** Critical for checkout, authentication and order workflows; warning for low-volume optional routes.

**Initial investigation:** Inspect active connections, pool settings, slow queries, indexes, CPU, memory, replication and recent replica scaling.

**Mitigation:** Stop uncontrolled API scaling, reduce connection pressure, optimize the bottleneck and use the approved database failover procedure.

## 6. Canary Degradation

**Condition:** A canary version exceeds the baseline 5xx or latency threshold during a stability window.

**Severity:** Critical for user or business impact.

**Initial investigation:** Compare canary and baseline by route, instance, dependency, version and business outcome.

**Mitigation:** Set canary traffic to 0%, pause promotion and use a compatible rollback or fix-forward plan.

## 7. Graceful Shutdown Timeout

**Condition:** More than 1% of replacements exceed the termination grace period or active requests are terminated.

**Severity:** Warning initially; critical when it causes dropped requests, duplicate work or failed deployments.

**Initial investigation:** Review SIGTERM logs, request duration, draining behavior, queue acknowledgements and dependency closure.

**Mitigation:** Stop the rollout, fix shutdown handling and retest with active requests.

## 8. Business Availability Degradation

**Condition:** Checkout, login or order success falls below its agreed baseline despite API instances appearing healthy.

**Severity:** Critical.

**Initial investigation:** Compare release version, database latency, payment provider health, feature flags, retries, duplicate operations and route-level errors.

**Mitigation:** Disable the affected feature, use an approved fallback, stop promotion or roll back only after checking schema and data compatibility.

## Alert Review Checklist

```text
[ ] The alert is actionable
[ ] Duration and volume context are included
[ ] Service, instance and version are included
[ ] Severity reflects user or business impact
[ ] A responder is named
[ ] First investigation steps are documented
[ ] The alert links to HIGH-AVAILABILITY.md
[ ] Dependency outages do not cause duplicate pages
[ ] The condition has been safely tested
[ ] Thresholds were reviewed after an incident
```
