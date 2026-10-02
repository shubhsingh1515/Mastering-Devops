# Day 55 Assignment: Create a MERN Capacity Plan

## Objective

Use measurements to decide what should scale in your MERN application. Your design must distinguish API capacity, database capacity, cache capacity, queue capacity and worker capacity.

## Scenario

Production metrics show:

```text
Traffic:             2,000 requests/second
API CPU:              42%
API memory:           55%
API p95 latency:     280 ms
MongoDB CPU:          96%
MongoDB connections:  98% of limit
Query latency:        1.8 seconds
Cache hit rate:       82%
HTTP 5xx:             0.8%
```

## Required Deliverables

1. Create `CAPACITY-PLAN.md` with current baseline, expected peak, bottleneck analysis, thresholds and actions.
2. Create `SCALING-ARCHITECTURE.md` covering API, MongoDB, cache, queue, workers and external dependencies.
3. Identify the likely bottleneck in the scenario.
4. Explain why adding ten API instances immediately is unsafe.
5. List MongoDB queries, indexes, pool settings and capacity signals to investigate.
6. Identify read-heavy data that could be cached and describe its consistency policy.
7. Identify non-critical work that could move behind a queue.
8. Define a progressive load-testing plan.
9. Define at least five scaling triggers with owners and actions.
10. Define rollback and backpressure behavior when capacity limits are reached.

## Design Questions

1. What is capacity planning?
2. How do you distinguish an API bottleneck from a database bottleneck?
3. What is the difference between vertical and horizontal scaling?
4. What makes a good cache candidate?
5. What consistency risk does caching introduce?
6. Why should email and analytics often be asynchronous?
7. What does a queue absorb, and what does it not solve?
8. Why can retries amplify a downstream outage?
9. What signals indicate that workers need to scale?
10. How do you prove a scaling change worked?

## Completion Criteria

- The plan begins with a measured baseline.
- The likely bottleneck is identified from evidence.
- API and worker scaling are treated as separate dimensions.
- Cache candidates and invalidation policy are documented.
- MongoDB connection and query limits are considered.
- Queue depth and oldest-job age are monitored.
- External dependency quotas and timeouts are included.
- Load testing uses progressive, realistic traffic.
- Every trigger has an action, owner and safety limit.
- The design includes cost, failure and rollback considerations.
