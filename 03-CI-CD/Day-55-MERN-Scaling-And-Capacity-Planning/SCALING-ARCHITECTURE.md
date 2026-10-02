# MERN Scaling Architecture

This document defines how the application should scale across API, database, cache, queue, workers and external dependencies. Replace assumptions with measurements from `CAPACITY-PLAN.md`.

## 1. Current Architecture

```text
                         Users
                           |
                           v
                    Load Balancer
                           |
                  +--------+--------+
                  v        v        v
                API-1    API-2    API-3
                  |        |        |
                  +----+---+----+---+
                       |        |
                       v        v
                    Cache   MongoDB
                                |
                                v
                              Queue
                                |
                       +--------+--------+
                       v        v        v
                    Worker-1 Worker-2 Worker-3
```

API instances are stateless and can scale independently from workers. MongoDB is the durable source of truth. Cache data is disposable and must have an explicit freshness policy. The queue separates asynchronous work from user-facing requests.

## 2. Expected Traffic

```text
Current average requests/second:
Current peak requests/second:
Expected peak requests/second:
Expected growth multiplier:
Peak duration:
Orders/minute at peak:
Checkout requests/second at peak:
```

Forecast assumptions:

```text
- Identify the source of the traffic forecast.
- Include burst duration and concurrency.
- Include changes in read/write ratio.
- Include expected cacheable and non-cacheable traffic.
```

## 3. API Scaling Strategy

Run compatible stateless API replicas behind a health-aware load balancer. Scale replicas when sustained request concurrency, CPU, event-loop delay or latency indicates API pressure and MongoDB and external dependencies have headroom.

Scale-out sequence:

1. Start a new replica with the approved image and configuration.
2. Verify liveness and readiness.
3. Run smoke checks.
4. Add it to the load balancer.
5. Monitor latency, errors, database connections and business success.

Scale-down only when there is enough failover capacity and the remaining replicas can handle the workload. Use graceful shutdown to drain active work.

API scaling trigger:

```text
Signal:
Threshold:
Duration:
Action:
Owner:
Rollback:
```

## 4. Database Scaling Considerations

MongoDB may be the first constrained tier when API capacity increases. Monitor:

```text
CPU and memory
active and pooled connections
read and write latency
slow queries
index usage
disk and I/O
replication health and lag
```

Before adding API replicas:

- Check connection-pool multiplication across replicas.
- Review query shapes and indexes.
- Separate read-heavy from write-heavy workloads where appropriate.
- Confirm replication and failover behavior.
- Load-test with production-like data volume.

Do not treat database scaling as an automatic consequence of API scaling. The database has its own capacity plan and consistency constraints.

## 5. Cache Strategy

Use cache-aside for repeated and relatively stable reads:

```text
API -> cache lookup
       +-> hit  -> return
       +-> miss -> MongoDB -> cache result -> return
```

Candidate data:

```text
product catalog
category lists
popular products
public configuration
expensive computed responses
```

For each cached value define:

```text
cache key
TTL
maximum stale-data window
write invalidation event
fallback on cache outage
authorization rules
```

A cache outage should normally degrade to the source of truth if the database has capacity. If the cache stores required sessions, locks or rate-limit state, document that dependency separately.

## 6. Queue and Worker Strategy

Move suitable non-critical work out of the synchronous request path:

```text
API
  +-> durable business record
  +-> queue job
          +-> email worker
          +-> invoice worker
          +-> analytics worker
```

The queue must provide the behavior the business needs:

```text
retry policy
visibility timeout
idempotency
dead-letter handling
retention
ordering requirements
backpressure
```

Scale workers independently according to job arrival rate, processing rate, queue depth and oldest-job age. Do not add workers faster than downstream systems can handle.

Worker scaling trigger:

```text
Signal:
Threshold:
Duration:
Action:
Owner:
Duplicate-processing protection:
```

## 7. External Dependency Limits

For each external provider document:

| Dependency | Quota | Timeout | Retry limit | Backoff | Circuit behavior | Fallback |
|---|---:|---:|---:|---|---|---|
| Payment | TBD | TBD | TBD | Exponential + jitter | Fail fast when open | TBD |
| Email | TBD | TBD | TBD | Queue retry | Queue while unavailable | TBD |
| Object storage | TBD | TBD | TBD | Bounded | Protect API | TBD |

Use idempotency keys for operations that may be safely retried. Timeouts must be finite. Retry storms must be prevented with bounded attempts, backoff, jitter and circuit breaking.

## 8. Load-Testing Plan

Test in an approved production-like environment:

```text
baseline
1.25x baseline
1.5x baseline
2x baseline
expected peak
short controlled burst above peak
```

The test should use realistic routes, payloads, data size and read/write ratios. Measure:

```text
throughput
p50/p95/p99 latency
5xx and timeout rate
API CPU and memory
MongoDB query latency and connections
cache hit rate and evictions
queue depth and oldest-job age
worker throughput and failures
external dependency latency
checkout/order success
```

Define a stop condition before starting. Stop if error rate, latency, database pressure, provider quota use or queue growth exceeds the approved limit.

## 9. Scaling Triggers

| Component | Evidence | Trigger example | Action |
|---|---|---|---|
| API | CPU, concurrency, event-loop delay and p95 | Sustained saturation with dependency headroom | Add replicas or optimize code |
| MongoDB | Query latency, CPU and connections | Query latency or connection use above target | Optimize queries, indexes, pools or DB capacity |
| Cache | Hit rate, evictions and memory | Hit rate below baseline with DB pressure | Fix keys/TTL or add cache capacity |
| Queue | Depth and oldest-job age | Continuous growth | Add workers, optimize jobs or apply backpressure |
| External provider | Latency, errors and quotas | Provider limit or failure | Timeout, circuit break, queue or fallback |

Each trigger must have a dashboard, owner, action, cost boundary and rollback plan.

## 10. Failure Scenarios

### MongoDB saturation

Symptoms: high query latency, connection pressure and API timeouts while API CPU is moderate.

Response: stop uncontrolled API scale-out, inspect queries and pools, use cache for suitable reads, protect writes and scale the database according to its runbook.

### Cache outage

Symptoms: cache errors and a sudden rise in database reads.

Response: determine whether the source of truth can absorb the load, disable non-essential cache operations, protect MongoDB with rate limits and restore cache capacity safely.

### Queue backlog

Symptoms: queue depth and oldest-job age grow continuously.

Response: compare arrival and processing rates, inspect worker failures, add workers only if downstream capacity exists, optimize jobs and apply backpressure if required.

### External payment outage

Symptoms: provider timeouts and retries increase.

Response: enforce timeouts, open the circuit, stop retry amplification, use an approved payment state and do not report an order as paid until the business contract is satisfied.

### API overload

Symptoms: high API CPU or event-loop delay, rising p95 and 5xx rate.

Response: scale API replicas if dependencies have headroom, rate-limit or shed non-critical work, investigate hot routes and preserve healthy capacity.

### Load-test regression

Symptoms: the same workload produces worse latency or errors after a release.

Response: stop promotion, compare versions and dependency metrics, revert or fix the change, then rerun the same test.

## 11. Scaling Decision Record

```text
Date:
Change:
Observed bottleneck:
Evidence:
Component changed:
Expected result:
Actual result:
User/business impact:
Cost impact:
Rollback:
Follow-up:
```

The decision record prevents scaling by guesswork and makes future capacity reviews cumulative rather than anecdotal.
