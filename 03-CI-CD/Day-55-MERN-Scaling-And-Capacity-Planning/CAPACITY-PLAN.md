# MERN Capacity Plan

This is a template for recording measured capacity and deciding when to scale. Replace the placeholders with project-specific measurements and dates. Do not copy thresholds blindly.

## 1. Planning Metadata

```text
Service:
Environment:
Owner:
Plan version:
Measurement window:
Last reviewed:
Expected peak date:
Expected traffic growth:
```

## 2. Current Workload Baseline

```text
Average requests/second:
Peak requests/second:
Peak concurrent users:
Orders/minute:
Checkout requests/second:
Login requests/second:
Product-search requests/second:
```

Record both average and peak workload. Average traffic hides burst behavior.

## 3. Current API Capacity

```text
API replica count:
CPU average / peak:
Memory average / peak:
Event-loop delay:
p50 latency:
p95 latency:
p99 latency:
5xx rate:
Timeout rate:
Requests per replica:
```

Interpret these values together. High CPU with healthy dependencies may justify API scaling. Low API CPU with high database latency suggests that API replicas are not the first solution.

## 4. MongoDB Capacity

```text
MongoDB topology:
CPU average / peak:
Memory pressure:
Active connections:
Connection-pool utilization:
Read query latency:
Write query latency:
Slow-query rate:
Disk and I/O pressure:
Replication lag:
Index usage:
```

Current database bottleneck assessment:

```text
Likely bottleneck:
Evidence:
First action:
Owner:
Safety limit:
```

## 5. Cache Capacity

```text
Cache technology:
Memory utilization:
Hit rate:
Miss rate:
Eviction rate:
Average cache latency:
Key TTL policy:
Expected stale-data window:
```

Document cached data:

| Data | Cache key | TTL | Invalidation | Acceptable stale window |
|---|---|---|---|---|
| Product catalog | `products:*` | TBD | On catalog update | TBD |
| Category list | `categories:*` | TBD | On category update | TBD |
| Public configuration | `config:*` | TBD | On configuration release | TBD |

Never cache sensitive or user-specific data without a documented authorization and invalidation design.

## 6. Queue and Worker Capacity

```text
Queue technology:
Incoming jobs/minute:
Processing jobs/minute:
Queue depth baseline:
Oldest-job age baseline:
Worker count:
Worker concurrency:
Job failure rate:
Retry rate:
Dead-letter count:
```

If incoming work exceeds processing capacity for a sustained period, queue depth will grow. Decide whether to add workers, optimize jobs, reduce incoming work or apply backpressure.

## 7. External Dependency Capacity

| Dependency | Limit or quota | Current use | Timeout | Retry policy | Fallback |
|---|---:|---:|---:|---|---|
| Payment provider | TBD | TBD | TBD | Bounded and idempotent | TBD |
| Email provider | TBD | TBD | TBD | Queue and retry | TBD |
| Object storage | TBD | TBD | TBD | Bounded | TBD |

Verify provider limits before a planned traffic event.

## 8. Expected Peak Model

```text
Expected peak requests/second:
Expected peak orders/minute:
Expected peak queue jobs/minute:
Expected traffic multiplier:
Expected cache hit rate:
Expected database read/write ratio:
Expected external calls/second:
```

Assumptions:

```text
- Document the source of each forecast.
- Include burst duration, not just peak rate.
- Include data size and query-shape changes.
- Include business events such as promotions or launches.
```

## 9. Load-Test Plan

Test progressively in an approved environment:

```text
Baseline workload
  -> 1.25x baseline
  -> 1.5x baseline
  -> 2x baseline
  -> expected peak
  -> controlled burst above peak
```

Capture:

```text
throughput
p50/p95/p99 latency
5xx and timeout rate
API CPU and memory
MongoDB CPU, connections and query latency
cache hits, misses and evictions
queue depth and oldest-job age
worker throughput and failures
external dependency latency
business success rate
```

Stop criteria:

```text
- Error rate exceeds the approved limit.
- Latency violates the service objective.
- MongoDB reaches its safety limit.
- External provider rate limits are reached.
- Queue growth becomes unbounded.
```

## 10. Scaling Triggers and Actions

| Signal | Trigger | Action | Owner | Safety limit |
|---|---|---|---|---|
| API CPU | Above agreed threshold for TBD minutes | Add API capacity or optimize hot path | TBD | Check MongoDB headroom first |
| MongoDB query latency | Above TBD for TBD minutes | Inspect query, index and capacity | TBD | Stop API scale-out if DB saturated |
| DB connections | Above TBD% of limit | Tune pools and reduce pressure | TBD | Do not exceed provider limit |
| Cache hit rate | Below TBD baseline | Inspect keys, TTL and eviction | TBD | Verify freshness and auth |
| Queue oldest-job age | Above TBD | Add workers or apply backpressure | TBD | Check downstream limits |
| External errors | Above TBD% | Open circuit or use fallback | TBD | Avoid retry storm |

## 11. Cost and Capacity Guardrails

```text
Maximum API replicas:
Maximum worker replicas:
Maximum database tier or budget:
Maximum cache tier or budget:
Maximum queue retention:
Scale-down conditions:
Approval required for:
```

Capacity planning should protect reliability without creating uncontrolled cost growth. Define scale-down behavior only when it will not cause oscillation or leave insufficient failover capacity.

## 12. Review Procedure

Before a major traffic event:

1. Review the latest baseline and forecast.
2. Run the approved load test.
3. Confirm database, cache, queue and provider limits.
4. Verify dashboards and alerts.
5. Confirm scaling permissions and rollback procedures.
6. Assign owners for API, database, queue and business signals.
7. Record the final capacity decision.

After the event:

1. Compare predicted and actual workload.
2. Record the first constrained component.
3. Review error, latency and business outcomes.
4. Update thresholds and forecasts.
5. Capture follow-up optimization work.
