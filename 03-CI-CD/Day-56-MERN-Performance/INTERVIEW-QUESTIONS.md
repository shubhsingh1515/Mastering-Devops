# Day 56 Interview Questions: MERN Performance

## 1. Your API latency increased, but CPU is normal. What do you investigate?

Inspect downstream dependencies and latency breakdowns, especially MongoDB queries, cache performance, external APIs, network latency and connection pools. Normal CPU can mean the application is waiting on a slow dependency.

## 2. How would you optimize a slow MongoDB query?

Reproduce and measure it, inspect the query plan with `explain`, compare documents and keys examined with documents returned, verify indexes, reduce unnecessary fields, improve pagination or query structure, and validate the change under realistic load.

## 3. Would you add Redis immediately if MongoDB is slow?

No. First determine why MongoDB is slow. The problem may be a missing index, inefficient query, excessive data retrieval or a capacity issue. Caching can help read-heavy workloads, but it adds consistency and invalidation complexity.

## 4. Why should API instances remain stateless where practical?

Stateless instances allow any healthy replica behind the load balancer to handle a request and simplify scaling and failover. Shared state should live in an appropriate durable or shared service.

## 5. Why are p95 and p99 useful in addition to average latency?

They expose tail latency and degraded experiences affecting a subset of requests that an average can hide.

## 6. Why should you not automatically add API instances when latency increases?

The bottleneck could be MongoDB, a cache, an external service, network latency or another shared dependency. More API instances can increase pressure on the shared dependency.

## 7. What should happen when an instance becomes unready?

It should normally stop receiving new traffic while the issue is investigated or recovered. Liveness and readiness have different purposes: liveness helps detect a process that needs recovery, while readiness controls whether it should receive traffic.

## 8. Why move email or analytics processing to background workers?

To keep slow, non-critical work out of the synchronous request path and absorb workload bursts. The API can acknowledge the request after the required durable state is recorded, while workers handle retryable work.

## 9. What does a CDN primarily improve?

A CDN serves cacheable content from edge locations closer to users, reducing origin traffic and often reducing latency for static assets and safe public responses.

## 10. What is a cache hit?

A cache hit occurs when the requested key is found in the cache and can be returned without retrieving the value from the slower backend.

## 11. What is a major cost of database indexes?

Indexes consume disk and memory and add write and maintenance overhead. They should be created for measured query patterns rather than added to every field.

## 12. What does MongoDB `explain()` help investigate?

It shows query execution behavior, including the winning plan, execution time, keys examined, documents examined and documents returned.

## 13. Why is returning 500,000 records from an API generally bad?

It increases database work, serialization, memory usage, network transfer and frontend rendering work. Pagination and projection return a bounded response containing only the data required by the client.

## 14. What is an N+1 query problem?

It is one initial query followed by many unnecessary related queries, such as one order query plus one customer query for each of 100 orders. Batching, aggregation, careful preloading or denormalization may help.

## 15. Why is caching not always the answer?

Caching introduces stale data, invalidation complexity, memory usage, cache stampedes and consistency questions. It should be used for demonstrated read workloads with an acceptable freshness window.

## 16. If API CPU is 30 percent but MongoDB query latency is very high, what should you investigate?

Investigate database query shape, indexes, execution plans, documents examined, pagination, projections, connections, database capacity and cache effectiveness.

## 17. Walk through a p95 latency incident

> My MERN API has a p95 latency of 2.5 seconds, but CPU is only 35 percent. What do you do?

A strong answer is:

> I would not scale the API immediately because CPU is not showing saturation. I would break down request latency and inspect downstream dependencies, especially MongoDB, caches and external services. I would check p95 and p99, database query latency, connection pools, slow queries, cache hit rate and structured logs. For MongoDB, I would inspect query plans and documents examined versus returned, then look for missing or ineffective indexes, excessive data retrieval or poor pagination. I would validate the optimization with realistic load testing and compare latency, error rate, database utilization and throughput before and after the change.

This demonstrates evidence-based performance engineering rather than tool-driven guessing.

## Interview Answer Framework

Use this structure for performance questions:

```text
1. Confirm and quantify the symptom.
2. Break down latency by dependency.
3. Find the saturated or inefficient component.
4. Make the smallest evidence-based change.
5. Load test with realistic data and traffic.
6. Deploy gradually with rollback available.
7. Compare latency, errors, throughput and resource health.
```
