# MERN Performance Runbook

## Purpose

Use this runbook when a MERN application becomes slow, when a performance alert fires, or when validating a performance-related release. The goal is to identify the actual bottleneck with evidence, apply the smallest useful change, and verify that the change improves the production workload without creating a new failure mode.

Do not scale API replicas automatically. First determine whether the constraint is the API, MongoDB, a cache, the CDN, an external dependency, the network, or a connection pool.

---

## 1. Performance SLOs

Define targets before optimizing. Example starting targets:

| Indicator | Target | Alert threshold | Notes |
| --- | ---: | ---: | --- |
| Product-list API p95 | <= 500 ms | > 750 ms for 10 minutes | Exclude intentional background jobs |
| Product-list API p99 | <= 1,000 ms | > 1,500 ms for 10 minutes | Investigate tail latency |
| API availability | >= 99.9% | < 99.9% over rolling window | Use successful eligible requests |
| 5xx rate | < 0.5% | >= 1% for 5 minutes | Separate dependency failures |
| CDN cache hit ratio | >= 90% for static assets | < 80% | Check cache headers and invalidations |
| Catalog cache hit ratio | >= 80% | < 60% | Confirm workload and key cardinality |
| MongoDB query time | <= 250 ms for target query | > 500 ms | Measure at the database and API |

Document the scope of each SLO: route, region, tenant, traffic class, time window and exclusions. A target without a measurement definition is not actionable.

---

## 2. Key Latency Metrics

Monitor the following dimensions together:

```text
API:       request rate, p50, p95, p99, 5xx, response size,
           in-flight requests and connection-pool wait
MongoDB:   query latency, CPU, memory, connections, disk I/O,
           documents examined, keys examined and slow queries
Cache:     hit rate, miss rate, evictions, memory, command latency
CDN:       edge latency, origin latency, hit ratio, origin requests
External:  dependency latency, timeout rate, retry count and errors
```

Always compare the current window with a known healthy baseline. Record deployment version, traffic volume, dataset size and region so that comparisons are meaningful.

Useful latency breakdown:

```text
Total request latency
  = queue and pool wait
  + application processing
  + cache lookup
  + MongoDB query
  + external dependency
  + serialization and network transfer
```

---

## 3. API Bottleneck Investigation

### First response

1. Confirm the alert and affected route.
2. Check whether the increase affects p50, p95, p99 or only one percentile.
3. Compare request rate and response size with the baseline.
4. Check 5xx, timeout and cancellation rates.
5. Check API CPU, memory, event-loop lag and instance health.
6. Inspect traces and structured logs for dependency timing.
7. Identify whether the API is computing, waiting, retrying or queued.

### Interpretation

```text
High API CPU + high latency
  -> inspect application code, serialization, compression and event-loop work

Normal API CPU + high latency
  -> inspect MongoDB, cache, network, external APIs and connection pools

High response size + high latency
  -> inspect projection, pagination, compression and frontend contract

Latency only after a deployment
  -> compare query plan, feature flags, cache keys and configuration
```

Do not treat low CPU as proof that the service is healthy. A process waiting on I/O can have low CPU while users experience high latency.

---

## 4. MongoDB Investigation

Start with the exact query shape and production-like data. Record:

```text
Filter
Sort
Projection
Limit
Collection size
Index definitions
Execution time
Documents examined
Keys examined
Documents returned
Winning plan
```

Run a read-only investigation such as:

```javascript
db.products
  .find({ category: "shoes" })
  .project({ name: 1, price: 1, thumbnail: 1 })
  .sort({ _id: -1 })
  .limit(20)
  .explain("executionStats");
```

Look for collection scans, excessive documents examined, an incompatible sort, low-selectivity indexes, large documents, unbounded result sets and connection-pool waits.

Do not add an index blindly. Check write volume, index size, available memory and whether the index supports the complete filter and sort. Test the candidate index against representative data before deploying it.

After the change, verify:

```text
Query latency decreases
Documents examined decreases
Returned result is correct
Write latency does not regress materially
MongoDB CPU and disk remain healthy
Index fits the intended workload
```

---

## 5. Cache Strategy

For each cached resource, document:

```text
Resource
Cache key
Owner
TTL
Maximum stale-data window
Invalidation event
Serialization format
Maximum value size
Sensitive-data policy
Fallback behavior
```

Recommended default for a public catalog:

```text
Pattern: cache-aside
TTL: 30-120 seconds, based on freshness requirements
Key: resource + normalized filters + pagination cursor
Miss behavior: query MongoDB, populate cache, return result
Failure behavior: bypass cache and use MongoDB with rate protection
```

Never omit a result-changing input from the cache key. Include tenant, locale, authorization scope, filter, sort and pagination state where applicable.

When a write changes cached data, either invalidate the affected key, update it safely, or accept a documented TTL window. Protect popular keys from stampedes with locking, stale-while-revalidate behavior or regeneration limits.

Track:

```text
Hit rate
Miss rate
Evictions
Cache memory
Cache latency
Backend requests caused by misses
Stale-data incidents
```

A cache is an optimization, not the source of truth. The application must continue to behave correctly when the cache is empty or temporarily unavailable.

---

## 6. CDN Strategy

Use the CDN for public, cacheable assets and responses:

```text
JavaScript
CSS
Images
Fonts
Static files
Public catalog responses with explicit freshness rules
```

For versioned static assets:

```http
Cache-Control: public, max-age=31536000, immutable
```

For HTML or frequently changed content, use a shorter TTL and validators such as `ETag`.

Before enabling shared caching for an API response, verify:

- The response contains no user-specific or sensitive data.
- The cache key includes every result-changing input.
- Authentication and authorization do not leak one user's data to another.
- Purge or invalidation behavior is documented.
- Stale-data behavior is acceptable.

Investigate CDN regressions by comparing hit ratio, origin request rate, edge latency, origin latency, cache headers and recent invalidations.

---

## 7. Query Optimization Process

Use this sequence:

1. Capture a baseline under representative traffic.
2. Identify the exact slow query and endpoint.
3. Check filter, sort, projection and pagination.
4. Run `explain("executionStats")`.
5. Compare documents and keys examined with documents returned.
6. Review existing indexes and write costs.
7. Make one focused query, index or data-shaping change.
8. Re-run correctness tests.
9. Load test with realistic data and concurrency.
10. Compare p50, p95, p99, throughput, errors and database health.
11. Deploy gradually.
12. Keep the rollback option available until the observation window is complete.

Common improvements include:

```text
Add a query-supporting compound index
Project only required fields
Use bounded pagination
Replace deep offsets with a cursor
Batch related lookups
Remove N+1 queries
Reduce unnecessary aggregation stages
Cache safe, repeated reads
```

Do not optimize based only on a local dataset. A query that is fast with 100 documents may fail with 100 million documents.

---

## 8. Load-Testing Process

### Prepare

Define:

```text
Endpoint mix
Request rate
Concurrency
Payload sizes
Authentication behavior
Dataset size and distribution
Cache state: warm or cold
Test duration
Success criteria
```

Include realistic filters and pagination patterns. A test containing only cache hits can hide a database bottleneck.

### Execute

Run a baseline and candidate version with the same:

```text
Traffic profile
Dataset
Region
Instance count
Database tier
Cache state
Duration
```

Warm-up traffic should be separated from the measured window. Avoid testing production without an approved change window and traffic-safety controls.

### Evaluate

Record:

```text
Throughput
p50, p95, p99
5xx and timeout rate
API CPU and memory
MongoDB CPU, memory and query latency
Cache hit rate and evictions
Connection-pool wait
Cost or resource increase
```

A change passes only when it meets the target without an unacceptable regression in error rate, writes, consistency, cost or another route.

---

## 9. Performance Regression Detection

Add performance checks to the delivery process where practical:

- Store baseline latency and throughput for important endpoints.
- Run a repeatable smoke load test for every high-risk query or index change.
- Track p95 and p99 by route and deployment version.
- Alert on sustained regressions rather than one noisy sample.
- Compare query plans after schema, index or database-version changes.
- Monitor response-size changes.
- Use canary releases for high-risk performance changes.
- Keep performance dashboards linked to deployment markers.

Regression examples:

```text
p95 increases 40 percent after deployment
Query plan changes from IXSCAN to COLLSCAN
Cache hit rate falls after a key-format change
Response size doubles after a new field is added
Write latency rises after adding several indexes
```

A performance test is most useful when it fails close to the change that caused the regression and includes enough evidence to reproduce it.

---

## 10. Rollback Procedure for Bad Optimizations

Use rollback when the change causes elevated errors, unacceptable latency, incorrect data, write degradation, cache leakage, resource exhaustion or a worse user experience.

### Application or configuration change

1. Confirm the regression and identify the deployment version or flag.
2. Stop or pause progressive rollout.
3. Disable the feature flag or route traffic to the last known-good version.
4. Verify p95, p99, 5xx, dependency latency and database health.
5. Preserve logs, traces, metrics and the query plan for investigation.
6. Open a follow-up task with the failed hypothesis and evidence.

### Index change

Do not remove an index during an active incident unless the effect and operational risk are understood. First disable the dependent application path or roll back application traffic when possible. Schedule index removal with a plan for:

```text
Replica or staging validation
Write-impact review
Disk and memory review
Maintenance window
Post-removal query verification
```

### Cache change

1. Disable the new cache path if it serves incorrect or stale data.
2. Purge affected keys or the affected CDN path when safe.
3. Bypass the cache with rate protection if MongoDB can handle the load.
4. If MongoDB cannot handle the miss storm, restore the last known-good cache behavior or serve a controlled fallback.
5. Verify authorization and tenant isolation.

### Rollback success criteria

```text
Latency returns toward baseline
Error rate is within SLO
Database CPU and query latency recover
Cache behavior is correct
No stale or unauthorized data is served
Traffic is stable for the observation window
```

A rollback restores service; it does not replace root-cause analysis. Record what was changed, what was observed, why the hypothesis failed and what evidence is needed before a future retry.

---

## 11. Investigation Evidence Template

Copy this template into the incident or change record:

```text
Date and time:
Incident or change ID:
Deployment version:
Affected endpoint(s):
Affected region or tenant:
Traffic rate:
Dataset size:
Cache state:

Baseline:
  p50:
  p95:
  p99:
  5xx rate:
  response size:
  MongoDB query time:
  documents examined:
  documents returned:
  cache hit rate:

Observed change:

Hypothesis:

Action taken:

Validation:
  p50:
  p95:
  p99:
  5xx rate:
  API CPU:
  MongoDB CPU:
  query time:
  cache hit rate:

Decision:
Rollback plan:
Follow-up owner:
```

---

## 12. Quick Decision Tree

```text
Is p95 or p99 above target?
  |
  +-> No: continue normal observation
  |
  +-> Yes
        |
        +-> Is API CPU or event-loop lag high?
        |     +-> Yes: inspect application work and serialization
        |     +-> No: inspect dependencies
        |
        +-> Is MongoDB query latency or CPU high?
        |     +-> Yes: inspect explain plan, indexes and query shape
        |
        +-> Is cache hit rate unexpectedly low?
        |     +-> Yes: inspect key design, TTL, evictions and invalidation
        |
        +-> Is CDN origin traffic high?
        |     +-> Yes: inspect cache headers, keys and purge behavior
        |
        +-> Is an external dependency slow?
              +-> Yes: inspect timeout, retry and circuit-breaker behavior
```

Always finish with a before-and-after comparison. Performance work is complete only when the measured bottleneck improves and the surrounding system remains healthy.
