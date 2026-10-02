# MERN Scaling and Capacity Alert Catalog

Tune these alerts against measured baselines, service objectives, dependency limits and business impact. Every alert should identify the service, environment, owner, dashboard and action.

## Alert Design Rules

- Alert on sustained, actionable conditions.
- Include traffic volume and duration.
- Distinguish a single instance from fleet-wide saturation.
- Prefer user and business impact over isolated resource noise.
- Identify the constrained tier and the first investigation step.
- Avoid paging repeatedly for the same underlying dependency outage.
- Review thresholds after load tests and incidents.

## 1. API Saturation

**Condition:** API CPU, event-loop delay, memory or request concurrency remains above the service threshold while latency or error rate degrades.

**Initial investigation:** Compare API replicas, route latency, request rate, garbage collection, downstream latency and recent releases.

**Mitigation:** Scale API capacity when dependencies have headroom, optimize the hot path or apply rate limiting and backpressure.

## 2. MongoDB Query Latency

**Condition:** p95 or p99 query latency exceeds the agreed threshold for a sustained period, or application timeouts increase.

**Initial investigation:** Inspect slow queries, indexes, query shapes, database CPU, memory, disk and replication health.

**Mitigation:** Optimize queries or indexes, cache suitable reads, adjust capacity and avoid adding API replicas until database pressure is understood.

## 3. MongoDB Connection Pressure

**Condition:** Connection pools approach their limit or MongoDB connections remain above the safe operating target.

**Initial investigation:** Compare replica count, pool sizes, active requests, leaked connections and recent scaling changes.

**Mitigation:** Stop uncontrolled scaling, tune pools safely, reduce unnecessary connections and scale or repair the database layer.

## 4. Cache Hit-Rate Degradation

**Condition:** Cache hit rate falls below the workload baseline while database reads and latency rise.

**Initial investigation:** Check key generation, TTL, eviction, cache memory, deployment changes and workload mix.

**Mitigation:** Correct cache keys or TTL, restore capacity, add safe cache candidates or accept the database load deliberately. Verify authorization and freshness behavior.

## 5. Queue Growth

**Condition:** Queue depth or oldest-job age grows continuously for the defined window.

**Initial investigation:** Compare incoming job rate, worker throughput, worker errors, downstream latency, retries and dead-letter jobs.

**Mitigation:** Add workers if downstream capacity allows, optimize processing, limit incoming work or apply backpressure. Do not acknowledge failed work as complete.

## 6. Worker Failure Rate

**Condition:** Job failures or retries exceed the normal baseline.

**Initial investigation:** Group failures by job type, version, dependency, payload class and worker instance.

**Mitigation:** Pause the affected job type when safe, fix or roll back the worker, move poison jobs to a dead-letter path and preserve idempotency.

## 7. External Dependency Saturation

**Condition:** Provider latency, rate-limit responses or error rate exceeds the agreed threshold.

**Initial investigation:** Check provider status, quotas, request volume, retry amplification and circuit state.

**Mitigation:** Enforce timeouts, bounded retries, backoff and circuit breaking. Use a fallback or queue work when the business contract permits.

## 8. Load-Test Regression

**Condition:** A new build or configuration reduces throughput or increases p95/p99 latency under the same approved workload.

**Initial investigation:** Compare build, query plans, cache behavior, connection use, worker throughput and external calls.

**Mitigation:** Stop promotion, revert or fix the change and rerun the same workload before approval.

## 9. Business Capacity Degradation

**Condition:** Checkout success, order throughput, login success or other key business signal falls below baseline.

**Initial investigation:** Correlate business metrics with route latency, database errors, provider limits, queue age, release version and feature flags.

**Mitigation:** Disable the affected feature, reduce traffic or workload, use an approved fallback and start incident response.

## Alert Review Checklist

```text
[ ] The condition is actionable
[ ] Duration and volume context are included
[ ] The constrained tier is identifiable
[ ] User or business impact determines severity
[ ] An owner and dashboard are named
[ ] First investigation steps are documented
[ ] A safe mitigation is documented
[ ] Cost and capacity limits are considered
[ ] The alert has been tested
[ ] Thresholds were reviewed after use
```
