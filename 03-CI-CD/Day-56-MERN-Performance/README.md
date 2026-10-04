# DevOps Mentorship Program - Day 56

## Phase 3: Production Operations

### MERN Performance: Caching, CDN, Database Indexes and Query Optimization
 
**Focus:** Performance engineering, caching layers, MongoDB indexes, query optimization and interview troubleshooting

Yesterday, Day 55, you learned to scale based on measured bottlenecks instead of simply adding servers.

Today we go one layer deeper:

> How do you make a MERN application faster before throwing more infrastructure at it?

The performance path is:

```text
Browser
  |
  v
CDN
  |
  v
Load Balancer
  |
  v
API
  |
  v
Cache
  |
  v
MongoDB
  |
  v
Optimized queries and indexes
```

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Identify common MERN performance bottlenecks.
- Explain browser caching and CDN caching.
- Explain API and server-side caching.
- Understand MongoDB indexes and their costs.
- Use `explain()` to investigate query behavior.
- Recognize inefficient queries, over-fetching and N+1 query patterns.
- Choose between offset and cursor pagination.
- Explain why caching is not a replacement for query optimization.
- Connect performance metrics to DevOps decisions.
- Troubleshoot a slow production MERN endpoint.
- Validate an optimization with load testing and before/after metrics.
- Answer performance questions in interviews.

---

## 2. Measure Before Optimizing

When users say, "The website is slow," do not immediately add servers, increase CPU, install Redis or resize MongoDB. First collect evidence.

Useful measurements include:

```text
Request rate
p50 latency
p95 latency
p99 latency
5xx rate
Database query latency
Database CPU and memory
Cache hit rate
External API latency
Connection-pool utilization
Response size
```

For example:

```text
API p95       = 2.4 seconds
Mongo query   = 2.1 seconds
API CPU       = 35 percent
MongoDB CPU   = 88 percent
```

This evidence points toward the database query, not API CPU. Adding API replicas could increase the number of concurrent database queries and make the actual bottleneck worse.

A useful request timing breakdown is:

```text
Total request time
  = queue time
  + application time
  + database time
  + cache time
  + external dependency time
  + network and serialization time
```

### A small Express timing middleware

```javascript
app.use((request, response, next) => {
  const startedAt = process.hrtime.bigint();

  response.on("finish", () => {
    const durationMs = Number(process.hrtime.bigint() - startedAt) / 1_000_000;

    console.log(JSON.stringify({
      route: request.route?.path ?? request.path,
      method: request.method,
      statusCode: response.statusCode,
      durationMs: Math.round(durationMs)
    }));
  });

  next();
});
```

In production, send structured timing data to your metrics and log platform instead of relying only on console output.

---

## 3. Percentiles Matter

An average can hide a poor experience for a subset of users.

Suppose 100 requests have this distribution:

```text
99 requests -> 100 ms
1 request   -> 10 seconds
```

The average is approximately 199 ms, which may look acceptable. One user still waited ten seconds.

Production systems commonly monitor:

- **p50:** the median request latency; half of requests are faster.
- **p95:** 95 percent of requests are faster than this value.
- **p99:** 99 percent of requests are faster than this value; it exposes tail behavior.

Track p50, p95 and p99 together. If p50 is stable but p99 rises, a smaller group of requests is being delayed by a slow query, exhausted pool, lock, cold cache, external dependency or noisy neighbor.

---

## 4. Browser Caching

A React application contains assets such as:

```text
main.js
styles.css
logo.svg
product images
fonts
```

The browser should not download unchanged assets on every visit. A first visit downloads an asset and stores it locally; later visits can reuse the cached copy.

```text
First visit
  |
  +-> Download asset
  +-> Store asset locally

Later visit
  |
  +-> Reuse cached asset when still fresh
```

Browser caching reduces network traffic, origin load and page latency. It is especially effective for immutable, versioned assets.

### Cache-Control and content hashing

A production build commonly creates names such as:

```text
main.8f31c2.js
styles.4e19aa.css
```

If the file contents change, the build creates a new name:

```text
main.a91e22.js
```

That allows a long cache lifetime for the old immutable file without preventing users from receiving a new build.

Conceptual response headers:

```http
Cache-Control: public, max-age=31536000, immutable
ETag: "8f31c2"
```

Do not apply a year-long immutable policy to an unversioned file such as `main.js`, because users may continue to run an old deployment. HTML usually needs a shorter cache lifetime so it can point to the newest asset names.

Other useful validators include:

```http
ETag
Last-Modified
If-None-Match
If-Modified-Since
```

A conditional request can return `304 Not Modified`, avoiding a full response body while still checking freshness.

---

## 5. CDN Caching

A CDN places cacheable content at edge locations closer to users.

Without an edge cache:

```text
User in Mumbai
      |
      v
Origin server in another region
```

With a CDN:

```text
User
  |
  v
Nearby CDN edge
  |  cache hit
  v
Cached asset
```

A CDN is useful for:

- JavaScript and CSS.
- Images and video segments.
- Fonts.
- Static files.
- Public, cacheable API responses when freshness rules are clear.

CDNs reduce origin traffic and often reduce latency, but they do not automatically fix slow dynamic queries. A CDN miss still reaches the origin, and incorrect cache headers can serve stale or private data.

For public content, define:

```text
Cache key
TTL
Purge or invalidation process
Compression policy
Stale-data tolerance
Security boundary
```

Never cache a response containing user-specific or sensitive data in a shared CDN cache unless the cache key and privacy behavior are explicitly designed for it.

---

## 6. Multi-Layer Caching

A mature application may have several layers:

```text
Browser cache
      |
      v
CDN cache
      |
      v
Application cache
      |
      v
MongoDB
```

For `GET /api/products`, the request may be checked in this order:

```text
Browser?       -> hit: return locally
                  miss: continue
CDN?           -> hit: return edge response
                  miss: continue
API cache?     -> hit: return cached value
                  miss: continue
MongoDB        -> query, shape response, populate cache
```

Each layer needs an owner, a cache key, a TTL and an invalidation rule. A cache hierarchy without those decisions is difficult to troubleshoot because different users can observe different versions of the same data.

---

## 7. API Caching and Cache-Aside

Suppose `GET /api/products` changes only every few minutes.

Without caching:

```text
1,000 requests
      |
      v
1,000 MongoDB queries
```

With cache-aside behavior:

```text
1,000 requests
      |
      v
Cache
  +-> 950 hits
  +-> 50 misses -> MongoDB -> write cache
```

A simplified cache-aside flow is:

```javascript
async function getProducts({ category, page, limit }) {
  const cacheKey = `products:${category ?? "all"}:${page}:${limit}`;
  const cached = await redis.get(cacheKey);

  if (cached) {
    return JSON.parse(cached);
  }

  const products = await Product.find({ category })
    .select("name price thumbnail")
    .sort({ _id: -1 })
    .limit(limit)
    .lean();

  await redis.set(cacheKey, JSON.stringify(products), { EX: 60 });
  return products;
}
```

The exact Redis client API varies by package. The important behavior is:

1. Build a complete key from every input that changes the result.
2. Read the cache.
3. Query MongoDB only on a miss.
4. Store the result with a bounded TTL.
5. Return the result.

A missing filter, tenant ID, authorization scope or locale in the cache key can cause incorrect data to be served.

### Do not cache everything

Caching introduces:

```text
Stale data
Invalidation complexity
Memory usage
Consistency questions
Cache stampedes
Operational complexity
```

Good candidates are frequently requested, expensive, and safe to serve slightly stale:

```text
Product catalog
Popular products
Public categories
Configuration
Expensive computed results
```

Poor candidates often include:

```text
Highly volatile transaction state
Sensitive user-specific data
Data requiring strict freshness
```

Ask:

> What is expensive to calculate or retrieve, frequently requested, and safe to serve slightly stale?

---

## 8. Cache Invalidation and Stampedes

Suppose the cached product price is `999`, but the database is updated to `1,099`. Without invalidation, users continue to see the old value.

Common strategies include:

- **TTL:** expire values after a defined period.
- **Explicit invalidation:** delete or update a key after a successful write.
- **Cache-aside:** the application reads on misses and populates the cache.
- **Write-through:** a write updates the cache and durable store through a coordinated path.
- **Event-driven invalidation:** publish a change event consumed by cache invalidators.

For every cached resource, document its acceptable stale-data window.

A **cache stampede** occurs when a popular key expires and many requests query MongoDB simultaneously. Mitigations include:

- TTL jitter so related keys do not expire at the same instant.
- A short distributed lock around regeneration.
- Serving stale data while one request refreshes it.
- Warming high-value keys before a predictable traffic event.
- Limiting regeneration concurrency.

Caching improves performance at the cost of consistency and operational complexity. It should be added where measured workload and freshness requirements justify it.

---

## 9. MongoDB Indexes

Suppose the application frequently executes:

```javascript
db.orders.find({ userId: "123" });
```

Without a suitable index, MongoDB may inspect many documents. An index provides an ordered access path to relevant records.

```text
Without index:
Query -> scan many documents -> find matches

With index:
Query -> index lookup -> fetch relevant documents
```

A basic index might be:

```javascript
db.orders.createIndex({ userId: 1 });
```

For a query that filters by `userId` and sorts by newest order, a compound index may be more appropriate:

```javascript
db.orders.createIndex({ userId: 1, createdAt: -1 });
```

The correct index depends on the real query shape, selectivity, sort and projection. Do not create indexes merely because a field exists.

### Index costs

Every index consumes:

```text
Disk
Memory
Write bandwidth
Maintenance work
Deployment and migration time
```

When a document is inserted, updated or deleted, relevant indexes may also need updating. Too many indexes can slow writes and consume memory needed for useful working sets.

> Create indexes based on real query patterns and verify them with execution statistics.

In Mongoose, define an index near the schema when that matches the project convention:

```javascript
orderSchema.index({ userId: 1, createdAt: -1 });
```

Index creation should be planned for production data size and deployment behavior. Do not assume a development database with a few records represents production performance.

---

## 10. Explain Query Performance

MongoDB provides `explain()` to inspect query execution.

```javascript
db.products
  .find({ category: "shoes" })
  .project({ name: 1, price: 1, thumbnail: 1 })
  .sort({ _id: -1 })
  .limit(20)
  .explain("executionStats");
```

Investigate:

```text
Winning plan
Execution time
Documents examined
Keys examined
Documents returned
Collection scans
Index scans
```

The goal is to answer:

> Is the database doing far more work than this query should require?

A suspicious result might look like:

```text
Documents examined: 900,000
Documents returned: 20
```

That suggests a missing or ineffective index, a low-selectivity filter, an incompatible sort, a broad query or a data-model problem. The fix should be based on the plan, not on guessing.

A useful ratio is:

```text
Work ratio = documents examined / documents returned
```

A high ratio is a signal to investigate, not an automatic proof that an index is missing. Some queries legitimately examine more records because of their business semantics.

---

## 11. Pagination

Returning 500,000 products from one API response creates unnecessary database, memory, network and frontend work.

Use bounded responses:

```text
GET /api/products?page=1&limit=20
```

Validate and cap the requested limit on the server:

```javascript
const page = Math.max(Number.parseInt(request.query.page, 10) || 1, 1);
const requestedLimit = Number.parseInt(request.query.limit, 10) || 20;
const limit = Math.min(Math.max(requestedLimit, 1), 100);
const skip = (page - 1) * limit;
```

### Offset pagination

```text
GET /products?page=10&limit=20
```

Offset pagination is simple and useful for smaller datasets or interfaces that need numbered pages. Deep offsets can become less efficient because the database may walk past many earlier records.

### Cursor pagination

```text
GET /products?after=<cursor>&limit=20
```

A cursor identifies the last item in a stable ordering. For example, a descending `_id` or `createdAt` cursor can let the database continue from a known position:

```javascript
const filter = after
  ? { _id: { $lt: new ObjectId(after) }, category }
  : { category };

const products = await Product.find(filter)
  .select("name price thumbnail")
  .sort({ _id: -1 })
  .limit(limit + 1)
  .lean();

const hasNextPage = products.length > limit;
const items = products.slice(0, limit);
const nextCursor = hasNextPage ? items.at(-1)._id : null;
```

Cursor pagination is often useful for large datasets, feeds and infinite scrolling. The ordering field must be indexed and stable, and the cursor should be opaque to clients when exposing internal details is undesirable.

---

## 12. Avoid Over-Fetching

A document may contain:

```text
name
price
description
images
reviews
internalMetadata
auditData
```

A product-list endpoint may need only:

```text
name
price
thumbnail
```

Use projection and response shaping:

```javascript
const products = await Product.find({ category })
  .select("name price thumbnail")
  .lean();
```

The preferred flow is:

```text
Database -> required fields -> API -> small response -> browser
```

Less data means less database transfer, serialization, memory, network usage and browser work. `lean()` can also avoid the overhead of creating full Mongoose documents when the endpoint only needs plain read-only objects.

Do not treat `lean()` as a universal optimization. Confirm that the route does not depend on document methods, getters, virtuals or middleware behavior that would change the response.

---

## 13. The N+1 Query Problem

Suppose the API loads 100 orders and then queries the customer separately for each order:

```text
1 query for orders
+ 100 customer queries
= 101 queries
```

This N+1 pattern increases database round trips and can produce severe latency under load.

Possible solutions depend on the data model:

- Fetch related records in a batch using `$in`.
- Use an aggregation pipeline with `$lookup` when appropriate.
- Use Mongoose population carefully and measure the generated queries.
- Denormalize stable display fields when read performance is more important than avoiding duplicate data.
- Cache frequently reused related records.
- Preload data before rendering the response.

The correct solution depends on consistency requirements, cardinality, index design and response shape. The DevOps lesson is that application architecture can become an infrastructure performance problem.

---

## 14. Performance Monitoring

Dashboards should make the bottleneck visible.

### API

```text
Request rate
p50, p95 and p99 latency
5xx rate
Response size
In-flight requests
Connection-pool wait time
```

### MongoDB

```text
CPU
Memory
Connections
Query latency
Slow queries
Documents and keys examined
Disk I/O
Lock or queue behavior
```

### Cache

```text
Hit rate
Miss rate
Evictions
Memory usage
Key count
Command latency
Connection errors
```

### CDN

```text
Cache hit ratio
Origin requests
Bandwidth
Edge latency
Origin latency
Invalidations and purge failures
```

Correlate signals:

```text
API p95 rises
      |
      +-> Mongo query latency rises
      +-> Cache hit rate falls
      +-> Connection-pool wait rises
```

That correlation gives a much stronger troubleshooting path than looking at API CPU alone.

---

## 15. Production Troubleshooting Scenario

Alert:

```text
API p95 > 2 seconds
```

Observed metrics:

```text
API CPU       = 35 percent
API memory    = 50 percent
MongoDB CPU   = 90 percent
Query latency = 1.7 seconds
Route         = /api/products
Duration      = 2.4 seconds
```

Query analysis:

```text
Documents examined: 900,000
Documents returned: 20
```

### Diagnosis

The API is not obviously CPU-bound. The database query is doing excessive work.

### Investigation and remediation

```text
Measure request breakdown
        |
        v
Inspect slow-query logs
        |
        v
Run explain("executionStats")
        |
        v
Review filter, sort, projection and index
        |
        v
Add or adjust the smallest useful index
        |
        v
Add pagination and reduce fields if needed
        |
        v
Load test with realistic data
        |
        v
Deploy gradually and monitor
```

Do not immediately scale the API from five to twenty instances. More replicas may create more database pressure while leaving the query inefficient.

---

## 16. A Repeatable Optimization Process

Use this process for a slow endpoint:

1. Define the endpoint, workload, users affected and target latency.
2. Capture baseline p50, p95, p99, throughput, error rate and response size.
3. Break down latency into application, cache, database, network and external dependency time.
4. Inspect logs and traces for the slow route.
5. Reproduce the query with production-like data.
6. Run `explain("executionStats")` and record the plan.
7. Review indexes, filters, sort order, projections and pagination.
8. Make one focused change at a time.
9. Load test the changed path at expected and peak traffic.
10. Compare database CPU, query time, cache behavior, latency and errors.
11. Deploy gradually with a rollback plan.
12. Continue observing after the change.

Do not claim success because a local request became faster. The optimization is successful only if the production-relevant workload improves without unacceptable write, consistency or cost regressions.

---

## Additional Day 56 Resources

- [Commands](COMMANDS.md): MongoDB indexes, `explain()`, cache inspection, HTTP checks, load testing and Docker commands.
- [Interview questions](INTERVIEW-QUESTIONS.md): performance, caching, MongoDB, scaling and troubleshooting answers.
- [Quiz](QUIZ.md): ten-question knowledge check, answer key and self-review prompts.

---

## 17. Practical Exercise: Optimize a MERN Endpoint

Choose:

```text
GET /api/products
```

Document the current state:

```text
Current p95:
Current p99:
Requests per second:
Response size:
MongoDB query time:
Documents examined:
Documents returned:
Relevant indexes:
Cache hit rate:
```

Identify three opportunities, for example:

```text
1. Add or adjust an index based on the actual query pattern.
2. Return only fields required by the product-list screen.
3. Add appropriate pagination and cache public catalog results.
```

Define the proof of improvement before making the change:

```text
Before:
p95 = 1.8 seconds
Mongo query = 1.4 seconds

Target:
p95 <= 500 ms
Mongo query <= 250 ms
No increase in 5xx rate
```

Do not claim success until you measure the same workload before and after the change.

---

## 18. Practical Assessment: Find the Real Bottleneck

Production metrics:

```text
Traffic:          1,500 requests/second
API CPU:             32 percent
API memory:          48 percent
p95 latency:          2.2 seconds
p99 latency:          4.8 seconds
MongoDB CPU:          91 percent
Query latency:        1.7 seconds
Cache hit rate:       18 percent
5xx rate:              0.5 percent
```

### Questions and answers

**1. Is API CPU obviously the bottleneck?**  
No. API CPU is modest while MongoDB is highly utilized and query latency is high.

**2. What stands out?**  
MongoDB query latency, MongoDB CPU and the very low cache hit rate.

**3. What should you investigate first?**  
Slow queries, indexes, query plans, cache effectiveness, database connections and data returned.

**4. Should you immediately scale the API from five to twenty instances?**  
No. That could increase database pressure without fixing the query.

**5. How do you validate the fix?**

```text
Load test
  |
  v
Measure DB query latency
  |
  v
Measure API p95 and p99
  |
  v
Check error rate
  |
  v
Check DB CPU and connections
  |
  v
Compare before and after
```

---

## 19. Monthly Cumulative Project: Performance Milestone

The cumulative MERN DevOps project now includes:

```text
CI/CD
  Tests, Docker, security scanning, registry, staging and promotion

Release safety
  Rolling, canary, feature flags, rollback and health gates

Observability
  Metrics, logs, centralized logging and alerts

Incident response
  Triage, mitigation, recovery, RCA and corrective actions

Reliability
  Liveness, readiness, restart policies, graceful shutdown,
  load balancing, stateless API and high availability

Scalability
  Capacity planning, horizontal scaling, caching, queues,
  workers and load testing

Performance
  CDN, HTTP caching, API caching, MongoDB indexes,
  query optimization, pagination and performance metrics
```

### New deliverable

Create:

```text
PERFORMANCE-RUNBOOK.md
```

The runbook must include:

1. Performance SLOs.
2. Key latency metrics.
3. API bottleneck investigation.
4. MongoDB investigation.
5. Cache strategy.
6. CDN strategy.
7. Query optimization process.
8. Load-testing process.
9. Performance regression detection.
10. Rollback procedure for bad optimizations.

### Suggested runbook outline

```text
# MERN Performance Runbook

## Performance SLOs
## Key Metrics and Dashboards
## API Bottleneck Investigation
## MongoDB Investigation
## Cache Strategy
## CDN Strategy
## Query Optimization Process
## Load Testing Process
## Regression Detection
## Rollback Procedure
## Evidence Template
```

A useful evidence record includes the deployment version, endpoint, traffic shape, dataset size, p50/p95/p99, error rate, database CPU, query plan, cache hit rate and decision made.

---

## 20. Day 56 Summary

The performance model is:

```text
                         USER
                           |
                           v
                    Browser cache
                           |
                           v
                          CDN
                           |
                           v
                    Load balancer
                           |
                 +---------+---------+
                 |         |         |
                 v         v         v
               API-1     API-2     API-3
                 |         |         |
                 +----+----+----+----+
                      |         |
                      v         v
                    Cache     MongoDB
                                  |
                                  v
                         Optimized queries
                           and indexes
```

The troubleshooting process is:

```text
Measure
  |
  v
Find the bottleneck
  |
  v
Optimize the smallest useful part
  |
  v
Load test
  |
  v
Deploy safely
  |
  v
Observe
  |
  v
Compare before and after
```

### Interview takeaway

> Performance engineering starts with evidence, not infrastructure changes.

A slow API does not automatically need more API servers. The constraint may be a slow query, missing index, poor pagination, cache miss, external dependency, network latency, database saturation or N+1 queries. The DevOps responsibility is to identify the actual constraint and improve the system without simply moving the bottleneck elsewhere.

---

## 21. Next Lesson

Day 57 connects performance to **Service Level Objectives (SLOs), Service Level Indicators (SLIs), Service Level Agreements (SLAs) and error budgets**, turning MERN production metrics into measurable reliability targets.
