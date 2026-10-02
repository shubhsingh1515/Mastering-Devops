# Day 55 Interview Questions: Scaling and Capacity Planning

## 1. Your MERN application is slow. How do you decide what to scale?

I would not scale blindly. I would use metrics, traces and dependency dashboards to identify whether the constraint is API CPU or memory, MongoDB latency or connections, cache performance, queue depth, worker throughput or an external service. I would optimize or scale the constrained component and validate the change with realistic load testing.

## 2. Why does adding more API instances not always improve performance?

Another tier may be the bottleneck. More API instances can increase MongoDB connections, query concurrency, external API calls or network pressure. Scaling should follow evidence and dependency behavior rather than instance count alone.

## 3. What is capacity planning?

Capacity planning is estimating how much workload the system can handle, identifying when service objectives will be threatened and preparing the right capacity before that point. It uses technical metrics, business workload, growth forecasts, load tests and cost constraints.

## 4. What is the difference between vertical and horizontal scaling?

Vertical scaling gives one instance more CPU, memory or other resources. Horizontal scaling adds more instances. Horizontal scaling is usually a good fit for stateless APIs, but shared dependencies must also support the extra concurrency.

## 5. When would you use a cache?

I would cache repeated, relatively stable and safe-to-share reads such as product catalogs or computed public data. I would define TTL and invalidation behavior first, because stale or incorrectly authorized data can be worse than a slower database query.

## 6. What is cache-aside?

The application checks the cache first. On a miss, it reads the source of truth, stores the result in the cache and returns it. On writes, the application must use a deliberate invalidation or update strategy.

## 7. Why use a queue?

A queue separates user-facing request handling from asynchronous work, absorbs bursts and lets workers scale independently. It does not eliminate work; continuous queue growth means incoming work exceeds processing capacity.

## 8. What work belongs behind a queue?

Work that does not need to complete before the user response, such as email, analytics, invoice generation or notifications, is often a good candidate. The boundary depends on the business contract. Payment authorization, inventory reservation or durable order creation may need synchronous guarantees or careful idempotent workflows.

## 9. Why can retries make an incident worse?

If many requests retry a slow or failing dependency, they multiply downstream load and can create a retry storm. I would use short timeouts, bounded retries, exponential backoff, jitter and idempotency protection, plus a circuit breaker when appropriate.

## 10. How do you know workers need scaling?

Queue depth, oldest-job age and failure rate rise while workers are saturated or processing throughput is below incoming work. I would add workers or optimize jobs after checking downstream limits and duplicate-processing behavior.

## 11. How would you prepare for ten times current traffic?

I would establish a baseline and load-test progressively toward the expected peak. I would keep the API stateless, scale it behind a health-aware load balancer, assess MongoDB queries, indexes, connections and capacity, add safe caching, move suitable work to workers behind a queue, verify external quotas and configure bounded timeouts and retries. I would monitor technical and business metrics during the event and keep rollback and incident procedures ready.

## 12. What metrics matter in a capacity plan?

Requests per second, p95 and p99 latency, 5xx and timeout rate, API CPU and memory, MongoDB CPU, connections and query latency, cache hit rate, queue depth and oldest-job age, worker throughput, external dependency latency and business success metrics.

## 13. How do you prove a scaling change worked?

I would compare an equivalent before-and-after workload and verify throughput, latency, errors, dependency pressure, cache behavior, queue behavior and business outcomes. I would also confirm that the change did not simply move the bottleneck to another tier.

## 14. What is backpressure?

Backpressure prevents a system from accepting more work than it can safely process. Examples include bounded queues, rate limits, concurrency limits, load shedding and returning a controlled response instead of allowing unbounded memory or connection growth.
