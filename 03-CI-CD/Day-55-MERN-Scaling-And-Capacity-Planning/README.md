# DevOps Mentorship Program - Day 55

## Phase 3: Production Operations

### MERN Scaling and Capacity Planning
 
**Focus:** Horizontal scaling, bottlenecks, caching, background workers, capacity planning and production interview preparation

Yesterday, Day 54, you built the high-availability foundation:

```text
Users
  |
  v
Load Balancer
  |
  +-> API-1
  +-> API-2
  +-> API-3
          |
          v
       MongoDB
```

You also learned that adding API instances does not automatically solve every performance problem.

Today we answer:

> When traffic grows, what should you scale, and how do you know what is actually limiting the system?

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain capacity planning.
- Distinguish scaling the API from scaling the database.
- Identify common production bottlenecks.
- Explain vertical and horizontal scaling.
- Understand caching and when to use it.
- Separate synchronous API work from background work.
- Explain why queues help absorb traffic spikes.
- Design a scalable MERN architecture.
- Use metrics and load tests to validate scaling decisions.
- Answer scaling questions in DevOps interviews.

---

## 2. The Golden Rule of Scaling

Do not start with:

> Add more servers.

Start with:

> What is the bottleneck?

Suppose the application looks like this:

```text
Users
  |
  v
Load Balancer
  |
  v
API x 3
  |
  v
MongoDB
```

If the API grows from three to ten instances while MongoDB is already saturated, the result may be worse:

```text
More API capacity
        |
        v
More database requests and connections
        |
        v
MongoDB saturation
        |
        v
Higher latency and more timeouts
```

You scaled the wrong layer. Capacity engineering means measuring the complete request path, finding the constrained component and then changing that component.

---

## 3. Capacity Planning

Capacity planning estimates how much workload the system can handle, when it will approach its limits and what capacity should be added before users are affected.

Useful technical measurements include:

```text
Requests per second
CPU and memory
p95 and p99 latency
Database connections
Database query latency
Queue depth and job age
Cache hit rate
Error rate
```

For a MERN application, business workload matters too:

```text
Checkout requests per second
Orders per minute
Login requests per second
Product-search requests per second
Notifications per minute
```

A system may have acceptable average latency while checkout success is falling. Technical and business metrics should be analyzed together.

A simple planning process is:

```text
Measure current baseline
        |
        v
Estimate expected peak and growth
        |
        v
Find the first constrained component
        |
        v
Optimize or add capacity
        |
        v
Load-test the new design
        |
        v
Set alerts and repeat
```

---

## 4. Vertical and Horizontal Scaling

### Vertical scaling

Vertical scaling makes one instance larger:

```text
API server: 4 CPU / 8 GB RAM
                 |
                 v
API server: 8 CPU / 32 GB RAM
```

Advantages:

- Simple operational model.
- Often easy for an early workload.
- May improve a CPU or memory bottleneck quickly.

Limitations:

- The machine has a maximum size.
- It remains a failure domain.
- Larger instances may cost disproportionately more.
- Some changes require a restart or maintenance window.

### Horizontal scaling

Horizontal scaling adds instances:

```text
API-1
API-2
API-3
  |
  v
API-4
API-5
```

It is a natural fit for stateless HTTP APIs behind a load balancer. It also allows API, worker and cache capacity to be adjusted independently.

Horizontal scaling still has limits. Shared dependencies such as MongoDB, Redis, payment providers and network links must support the extra concurrency.

---

## 5. Finding the Bottleneck

Imagine this production snapshot:

```text
Traffic:        2,000 requests/second
API CPU:        40%
API memory:     50%
API p95:        250 ms
MongoDB CPU:    95%
Connections:    near limit
Query latency:  increasing
```

The likely bottleneck is MongoDB, not the API tier. Investigate:

```text
Indexes
Query patterns
Slow queries
Connection-pool settings
Read/write distribution
Database capacity
Cache opportunities
```

Use a request-path model:

```text
User
  |
  v
Load Balancer
  |
  v
API
  |
  +-> Cache
  |
  +-> MongoDB
  |
  +-> External services
  |
  +-> Queue
```

Possible evidence and response:

| Evidence | Likely investigation |
|---|---|
| API CPU is high while dependencies are healthy | Scale or optimize API code. |
| MongoDB query latency is high | Inspect indexes, queries and database capacity. |
| Cache hit rate is low for repeated reads | Review cache keys, TTL and cache placement. |
| Payment provider latency is high | Add timeouts, bounded retries and a circuit breaker. |
| Queue depth grows continuously | Add workers or reduce job processing time. |

A bottleneck can move after an optimization, so re-measure the whole path after every change.

---

## 6. Caching

Suppose a storefront receives 10,000 requests per minute for the same product catalog:

```text
10,000 requests
        |
        v
10,000 database queries
```

A cache can reduce repeated database work:

```text
Request
  |
  v
Cache lookup
  +-> HIT  -> return data
  +-> MISS -> MongoDB -> store result -> return data
```

A cache hit is a requested value found in the cache. A cache miss requires the application to load the value from MongoDB or another source.

Good cache candidates may include:

- Product catalogs.
- Popular products.
- Category lists.
- Public API responses.
- Configuration.
- Expensive computed data.

Be careful with:

- Sensitive user-specific data.
- Frequently changing transactional state.
- Data where stale values are unacceptable.
- Values that require strict authorization checks.

The goal is not to cache everything. The goal is to reduce expensive repeated work while preserving an acceptable consistency contract.

---

## 7. Cache-Aside and Invalidation

Cache-aside is a common pattern:

```text
Application
    |
    v
Check cache
  +-> Hit  -> return value
  +-> Miss -> read MongoDB
                |
                v
             write cache
                |
                v
             return value
```

The application controls when data is loaded into the cache.

Caching introduces a consistency problem. If the database price changes from 1,000 to 1,200 while the cache still holds 1,000, users may see stale data.

Common invalidation strategies include:

- TTL expiration.
- Explicit invalidation after writes.
- Write-through updates.
- Cache-aside with short TTLs.
- Event-driven invalidation.

Caching improves performance but introduces invalidation and consistency complexity. Document the acceptable stale-data window for each cached value.

---

## 8. Background Jobs and Workers

Not every operation belongs in the synchronous HTTP request path.

A single order request might otherwise wait for:

```text
Create order
  -> Charge payment
  -> Send email
  -> Generate invoice
  -> Update analytics
  -> Notify warehouse
  -> Return response
```

Slow external work increases request latency and creates more failure points. A better design moves appropriate non-critical work behind a queue:

```text
Browser
   |
   v
API
   |
   +-> Create durable order record
   |
   +-> Publish job to queue
              |
              +-> Email worker
              +-> Invoice worker
              +-> Analytics worker
```

The API can return after the required synchronous work is complete, while workers process asynchronous tasks independently.

The boundary must be explicit. Payment authorization or inventory reservation may be critical to the request, while email and analytics may be asynchronous. Do not acknowledge work as complete until the durable state and retry behavior match the business contract.

---

## 9. Why Queues Help With Spikes

Suppose normal background workload is 100 jobs per minute and a promotion creates 10,000 jobs per minute.

Without a queue, the API may try to perform every job immediately and become overloaded.

With a queue:

```text
API
  |
  v
Queue absorbs burst
  |
  v
Workers process at sustainable rate
```

Monitor:

```text
Queue depth
Job age
Processing rate
Failure rate
Retry count
Dead-letter jobs
```

A queue absorbs bursts; it does not make infinite work disappear. If incoming work remains above processing capacity, queue depth grows continuously. Add workers, improve processing efficiency, reduce work or apply backpressure.

API scaling and worker scaling are separate dimensions:

```text
HTTP requests  -> API instances
Background jobs -> Worker instances
```

A system might need ten API replicas and three workers, or five API replicas and twenty workers, depending on its workload.

---

## 10. Timeouts, Retries and Circuit Breakers

An API should not wait forever for a payment provider or other external service:

```text
Request
  |
  v
Bounded timeout
  |
  v
Success or controlled failure
```

Retries can help with transient failures, but uncontrolled retries amplify outages:

```text
1,000 original requests
        |
        v
3 attempts each
        |
        v
3,000 downstream requests
```

Use bounded retry counts, exponential backoff and jitter. Retry only operations that are safe to repeat or have idempotency protection.

A circuit breaker can stop repeated calls to a dependency that is failing:

```text
Dependency failures increase
        |
        v
Circuit opens
        |
        v
Fail fast or use fallback
        |
        v
Periodic probe after recovery window
```

Timeouts, retries and circuit breakers protect both the application and the dependency. They must be paired with useful error handling and observability.

---

## 11. Scalable MERN Architecture

A mature architecture may look like:

```text
                         Users
                           |
                           v
                    Load Balancer
                           |
                  +--------+--------+
                  |        |        |
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

Each tier can scale according to its own bottleneck:

- API instances for HTTP concurrency.
- Cache capacity for repeated reads.
- MongoDB capacity for durable data and queries.
- Workers for asynchronous processing.
- Queue capacity and retention for bursts.

The architecture should also define backpressure, failure handling, data consistency and cost limits.

---

## 12. Capacity Planning Example

Current baseline:

```text
Traffic:       1,000 requests/second
API p95:       300 ms
API CPU:       55%
MongoDB CPU:   65%
```

Expected peak is 2,000 requests per second. Do not simply double API instances. Ask:

```text
Can the API handle 2x traffic?
Can MongoDB handle 2x reads and writes?
Will connection pools exceed limits?
Can the cache absorb repeated reads?
Do payment and email providers have quotas?
Will workers and queues keep up?
```

Use a load test to establish the point where service objectives degrade. Capacity planning is an evidence-based estimate, not a guess about server count.

---

## 13. Load Testing

Test progressively with realistic traffic and data:

```text
500 requests/second
1,000 requests/second
1,500 requests/second
2,000 requests/second
```

Observe:

- CPU and memory.
- p50, p95 and p99 latency.
- HTTP 5xx and timeout rate.
- MongoDB query latency and connections.
- Cache hit rate.
- Queue depth, job age and processing rate.
- External dependency latency and error rate.
- Business success metrics.

A useful test identifies the capacity boundary and the first failing component. It should also verify recovery after the load decreases. Never run an unapproved load test against production.

---

## 14. Scaling Triggers

Set triggers based on baselines and service objectives rather than copying arbitrary numbers:

```text
If API CPU remains above the agreed threshold
    -> add API capacity or optimize hot paths

If MongoDB query latency or connection use rises
    -> investigate queries, indexes, pools and database capacity

If cache hit rate falls
    -> investigate keys, TTL, eviction and workload changes

If queue depth and oldest-job age grow continuously
    -> add workers, optimize jobs or apply backpressure

If external latency rises
    -> enforce timeouts, bounded retries and circuit breaking
```

Every trigger should have an owner, a dashboard, an action and a rollback or safety limit.

---

## 15. Interview Answers

### Your MERN application is slow. How do you decide what to scale?

I would not scale blindly. I would use metrics and traces to identify the bottleneck: API CPU or memory, database latency and connections, cache performance, queue depth or external dependency latency. I would optimize or scale the constrained component, then validate the change with realistic load testing.

### Why does adding API instances not always improve performance?

Another layer may be the bottleneck. More API instances can increase MongoDB connections, query concurrency or load on an external service. Scaling should follow evidence from metrics and dependency behavior.

### Why use a queue?

A queue decouples user-facing request handling from asynchronous work, absorbs bursts, allows workers to scale independently and prevents slow background operations from blocking the request path.

### How would you prepare a MERN application for 10x traffic?

I would establish a baseline, load-test progressively, identify bottlenecks, keep the API stateless and scale it behind a health-aware load balancer. I would evaluate MongoDB queries, indexes, connections and capacity, add caching for suitable reads, move non-critical work to workers behind a queue, verify external service quotas and configure bounded timeouts and retries. During the event I would monitor technical and business metrics and keep rollback and incident procedures ready.

---

## 16. Practical Exercise

Create both project deliverables:

- `CAPACITY-PLAN.md`: record current capacity, expected peak, measurements and scaling triggers.
- `SCALING-ARCHITECTURE.md`: document the API, database, cache, queue, worker, dependency and load-testing design.

Use realistic thresholds for your environment. A threshold is useful only when it is connected to an action and a measurable user or system outcome.

---

## Day 55 Summary

Your scaling model should now look like:

```text
                    TRAFFIC
                       |
                       v
                Load Balancer
                       |
              +--------+--------+
              v        v        v
            API-1    API-2    API-3
              |        |        |
              +----+---+----+---+
                   |       |
                   v       v
                Cache   MongoDB
                           |
                           v
                         Queue
                           |
                        Workers
```

The decision process is:

```text
Measure
  |
  v
Find bottleneck
  |
  v
Optimize
  |
  v
Scale the constrained component
  |
  v
Load-test
  |
  v
Observe and repeat
```

> Never answer "scale horizontally" without first asking what the bottleneck is.

A mature DevOps engineer thinks in terms of capacity, dependencies, bottlenecks, cost, reliability and user impact, not simply server count.

### Next Lesson

Day 56: MERN caching and performance optimization, including browser caching, CDN behavior, API caching, database indexes, query optimization and cache invalidation.
