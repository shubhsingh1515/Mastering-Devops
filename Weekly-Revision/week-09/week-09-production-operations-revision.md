# DevOps Mentorship Program - Week 9 Sunday Revision

## Phase 3: Production Operations

### Days 50-56 Weekly Revision

**Type:** Weekly revision, quiz and practical assessment  
**Level:** Intermediate to Professional  
**Focus:** Centralized logging, alerting, incident response, health checks, self-healing, high availability, scaling, capacity planning and performance optimization

Today is Sunday, so there is no new isolated topic. This revision connects the production-operations concepts from Days 50-56 into one production-grade MERN operating model.

This week's progression was:

```text
Day 50 -> Weekly revision
Day 51 -> Centralized logging and alerting
Day 52 -> Incident response and troubleshooting
Day 53 -> Health checks and self-healing
Day 54 -> Load balancing and high availability
Day 55 -> Scaling and capacity planning
Day 56 -> Performance optimization
```

The goal is to reason about the whole system rather than treating logs, metrics, load balancers, databases, queues and deployment processes as unrelated tools.

---

## 1. Learning Objectives

By the end of this revision, you should be able to:

- Connect logs, metrics, alerts and incident response.
- Explain how readiness checks support load balancing and failure isolation.
- Distinguish liveness from readiness.
- Explain how high availability and horizontal scaling work together.
- Identify the real bottleneck before scaling a component.
- Connect API latency to MongoDB queries, indexes, caches and capacity.
- Explain why API CPU can be normal while users experience high latency.
- Explain how queues and workers absorb asynchronous workload spikes.
- Use technical and business metrics during an incident.
- Identify when a feature flag or rollback is an appropriate mitigation.
- Troubleshoot a flash-sale incident across every production layer.
- Answer rapid-fire production operations interview questions.
- Design a production-grade MERN operating model and runbook.

---

## 2. Week in One Architecture

You should now be able to reason about this architecture:

```text
                              USERS
                                |
                                v
                         CDN / Edge Cache
                                |
                                v
                          Load Balancer
                                |
                 +--------------+--------------+
                 v              v              v
               API-1          API-2          API-3
                 |              |              |
                 +--------------+--------------+
                                |
                       +--------+--------+
                       v                 v
                     Cache            MongoDB
                                         |
                                  Optimized queries
                                         |
                                         v
                                       Queue
                                         |
                            +------------+------------+
                            v            v            v
                         Worker-1     Worker-2     Worker-3
```

Around every layer are the operational controls:

```text
Metrics
Logs
Alerts
Health checks
Incident response
CI/CD
```

This is the production system, not just the Node.js application. A healthy API process can still provide a poor user experience if MongoDB is saturated, cache misses are excessive, a queue is growing or an external dependency is timing out.

### How a request moves through the system

```text
User request
    |
    v
CDN or browser cache
    |
    +-> hit: return cached content
    +-> miss
          |
          v
      Load balancer
          |
          v
      Ready API instance
          |
          +-> application cache
          |      |
          |      +-> hit: return result
          |      +-> miss
          |
          +-> MongoDB query
          |
          +-> queue asynchronous work
          |
          v
      Response to user
```

The request path should contain only the work required to produce a correct response. Email, analytics, report generation and other suitable asynchronous work should be moved to workers.

---

## 3. Concept One: Logging, Alerting and Incident Response

The operational flow is:

```text
Application
     |
     v
Structured logs
     |
     v
Centralized logging
     |
     v
Metrics and dashboards
     |
     v
Actionable alert
     |
     v
Incident response
```

These components have different purposes:

- **Logs provide evidence.** They explain what a request or service observed.
- **Metrics show behavior over time.** They make trends, rates and saturation visible.
- **Alerts identify actionable conditions.** They should wake someone only when a response is required.
- **Incident response determines what to do.** It establishes impact, mitigates the problem, recovers service and captures learning.

### MERN example

Suppose the 5xx rate exceeds 5 percent:

```text
5xx rate > 5 percent
          |
          v
        Alert
          |
          v
   Check p95 latency
          |
          v
 Search logs by request ID
          |
          v
 Identify MongoDB errors
          |
          v
    Assess impact
          |
          v
      Mitigate
```

A request ID connects the user-visible failure, API log, database timing and downstream error. Without correlation data, engineers may see isolated symptoms instead of one request path.

### Good incident questions

```text
What changed?
When did the problem start?
Which routes or users are affected?
Is the problem regional or global?
Are technical metrics and business metrics both degraded?
What is the safest immediate mitigation?
What evidence should be preserved before changing the system?
```

Do not begin a detailed root-cause analysis while user impact is still increasing. Establish scope and reduce impact first.

---

## 4. Concept Two: Health Checks, Load Balancing and Self-Healing

This week's second major connection is health-aware traffic routing:

```text
API-2 becomes unhealthy
          |
          v
Readiness check fails
          |
          v
Load balancer removes API-2
          |
          v
API-1 and API-3 continue serving
          |
          v
Platform restarts or replaces API-2 if appropriate
          |
          v
Health check passes
          |
          v
Readiness check passes
          |
          v
API-2 returns to traffic
```

### Liveness versus readiness

- **Liveness answers:** "Am I alive enough for the platform to keep me running?"
- **Readiness answers:** "Should I receive new traffic right now?"

A readiness failure does not automatically mean the container should be restarted. An API may be alive but temporarily unable to serve safely because MongoDB is unavailable, a required configuration has not loaded or a dependency is recovering.

Example endpoints:

```text
GET /live  -> process is running
GET /ready -> instance can safely serve traffic
```

Use readiness to protect users from an unhealthy instance. Use liveness for conditions where restarting the process is likely to restore operation.

### Important caution

If every API instance fails readiness because MongoDB is unavailable, removing every instance from traffic does not fix MongoDB. Health checks isolate unsafe instances; they do not replace dependency recovery or incident response.

---

## 5. Concept Three: High Availability and Scaling

Multiple API instances provide two related benefits:

```text
Multiple healthy instances
          |
          +-> Availability: one failure need not stop service
          |
          +-> Capacity: traffic can be distributed
```

Scaling still must follow evidence.

### Bad approach

```text
Latency increases
        |
        v
Add 20 API servers
```

### Better approach

```text
Latency increases
        |
        v
Measure request and dependency timing
        |
        v
Find the constrained component
        |
        +-> API CPU or event loop
        +-> MongoDB query or capacity
        +-> Cache hit rate or latency
        +-> External API
        +-> Queue or worker capacity
        |
        v
Scale or optimize the constrained component
```

Adding API instances while MongoDB is saturated can create more database connections and queries. Horizontal scaling is useful when the API tier is the bottleneck, but it is harmful when it only amplifies load on a shared dependency.

### Stateless API requirement

APIs should remain stateless where practical:

```text
Request 1 -> API-1
Request 2 -> API-3
Request 3 -> API-2
```

Any healthy instance should be able to serve the request. Shared sessions, cache state and durable data should live in shared services rather than in one API process's memory.

---

## 6. Concept Four: Performance, Database and Caching

Suppose the metrics are:

```text
API p95           = 2.5 seconds
API CPU           = 30 percent
MongoDB query     = 2.1 seconds
```

Do not scale the API first. Investigate:

```text
Query plan
Indexes
Documents examined
Documents returned
Pagination
Projection
Cache hit rate
Database connections
Database capacity
```

A likely improvement path is:

```text
Slow query
    |
    v
Better query plan or index
    |
    v
Less database work
    |
    v
Lower database latency
    |
    v
Better API p95 and p99
```

### Cache as a measured optimization

Caching can reduce repeated reads:

```text
Repeated request
       |
       v
Cache hit -> return quickly
       |
       +-> miss -> query MongoDB -> populate cache
```

But caching introduces:

```text
Stale data
Invalidation complexity
Memory cost
Consistency tradeoffs
Cache stampedes
```

Before adding a cache, establish:

```text
What data is cached?
Who is allowed to receive it?
What is the cache key?
What is the TTL?
Is stale data acceptable?
How is the key invalidated?
What happens when the cache is unavailable?
```

Caching is not a replacement for query optimization. A cache miss still executes the underlying query, and a poorly designed cache can create correctness or privacy problems.

---

## 7. Concept Five: API Scaling, Queues and Workers

Not every operation belongs in the synchronous HTTP request path.

For example, `POST /api/orders` might need to:

```text
Create order
Charge payment
Send email
Generate invoice
Update analytics
Notify warehouse
```

Separate required synchronous work from suitable asynchronous work:

```text
API
 |
 +-> persist required order state
 |
 +-> publish durable job
          |
          v
        Queue
          |
          +-> Email worker
          +-> Invoice worker
          +-> Analytics worker
          +-> Warehouse worker
```

A queue can absorb a burst while workers process jobs at a sustainable rate.

Monitor:

```text
Queue depth
Job age
Processing rate
Failure rate
Retry count
Dead-letter jobs
```

If incoming work is greater than processing capacity:

```text
Incoming jobs > worker capacity
          |
          v
Queue depth continuously increases
```

That is a scaling signal. Possible responses include more workers, faster processing, batching, workload reduction or backpressure.

Do not confuse a growing queue with the primary MongoDB bottleneck. A queue may be a secondary symptom of slow database work in the workers.

---

## 8. How the Concepts Work Together During an Incident

A realistic failure chain may look like this:

```text
Traffic spike
    |
    v
Cache hit rate falls
    |
    v
MongoDB receives more reads
    |
    v
Database CPU and connections rise
    |
    v
API requests wait on queries
    |
    v
p95 and p99 increase
    |
    v
Checkout requests time out
    |
    v
Queue depth grows
    |
    v
One API instance fails readiness
    |
    v
Load balancer removes it
```

The response should not be "add API servers" by default. First protect users, isolate unsafe instances, identify the constrained layer and measure recovery after mitigation.

A cross-layer dashboard should make this chain visible:

```text
Traffic rate
API p50/p95/p99
5xx and timeout rate
Readiness failures
MongoDB CPU and connections
Query latency and documents examined
Cache hit rate
Queue depth and job age
Checkout success rate
```

---

## 9. Interview Rapid-Fire

Answer these aloud before reading the answers.

### 1. Why should not every error create an alert?

Because excessive low-value alerts create noise and alert fatigue. Alerts should represent meaningful conditions requiring action, such as sustained user impact, a severe SLO breach or a condition that needs immediate mitigation.

### 2. Why use readiness checks?

To prevent instances that are not currently safe to serve traffic from receiving new requests. Readiness protects the request path without necessarily restarting the process.

### 3. Why should APIs be stateless?

So any healthy instance can handle a request, making horizontal scaling, failover and rolling deployments easier. Shared state belongs in a shared, durable service when required.

### 4. Why does adding API instances not always improve performance?

A downstream dependency such as MongoDB, Redis, an external API or a queue may be the bottleneck. More API instances can increase pressure on that dependency.

### 5. Why use a queue?

To decouple suitable asynchronous work from user-facing requests and absorb workload spikes. The queue provides buffering, while workers process jobs independently.

---

## 10. Weekly Quiz

### Q1
A Node.js container is running, but its MongoDB dependency is unavailable. What should you generally consider first?

A. Restart every API instance  
B. Use readiness to prevent unsafe traffic while investigating the dependency  
C. Delete MongoDB  
D. Increase API traffic

### Q2
Your API has:

```text
CPU = 30 percent
p95 = 2.4 seconds
MongoDB query latency = 2.1 seconds
```

What is the strongest first hypothesis?

A. API CPU saturation  
B. Database or query bottleneck  
C. Load balancer failure  
D. React rendering

### Q3
What should a load balancer do with an instance whose readiness check consistently fails?

A. Send it more traffic  
B. Remove it from new traffic  
C. Ignore the failure  
D. Increase its request rate

### Q4
What is alert fatigue?

A. High CPU usage  
B. Engineers becoming desensitized to excessive low-value alerts  
C. Database replication lag  
D. Slow frontend rendering

### Q5
A production checkout feature is broken, but it is safely controlled by a feature flag. What can be an immediate mitigation?

A. Disable the feature  
B. Delete MongoDB  
C. Increase traffic  
D. Disable all monitoring

### Q6
What does horizontal scaling mean?

A. Increasing the size of one machine  
B. Adding more instances  
C. Removing instances  
D. Increasing database indexes

### Q7
Why can adding more API instances make MongoDB performance worse?

A. More API instances can create additional database load and connections  
B. APIs stop working when scaled  
C. MongoDB only supports one request  
D. Load balancers disable indexes

### Q8
What is one major downside of caching?

A. It always slows applications  
B. Cache invalidation and stale-data complexity  
C. It prevents horizontal scaling  
D. It eliminates observability

### Q9
What is the purpose of centralized logging?

A. Collect and search operational events across services and instances  
B. Replace databases  
C. Replace CI/CD  
D. Prevent all incidents

### Q10
During an incident, what should generally happen first?

A. Write a detailed root-cause analysis  
B. Reduce user impact and establish scope  
C. Delete logs  
D. Rewrite the application

### Weekly quiz answers

```text
1 -> B
2 -> B
3 -> B
4 -> B
5 -> A
6 -> B
7 -> A
8 -> B
9 -> A
10 -> B
```

### Score guide

| Score | Level |
| --- | --- |
| 9-10 | Production-ready understanding |
| 7-8 | Strong; review weak areas |
| 5-6 | Needs revision |
| Below 5 | Repeat this week's concepts |

Do not use the score alone as proof of readiness. You should also be able to explain why the correct answer is correct and what evidence would change the decision.

---

## 11. Weekly Practical Assessment: MERN Flash Sale Incident

### Scenario

Your production architecture is:

```text
CDN
  |
  v
Load Balancer
  |
  v
API x 5
  |
  v
Cache + MongoDB
  |
  v
Queue
  |
  v
Workers
```

At 14:00, traffic increases five times.

Observed metrics:

```text
API CPU:             45 percent
API p95:              2.8 seconds
API p99:              7.2 seconds

MongoDB CPU:          94 percent
DB connections:       96 percent of limit
Query latency:        2.2 seconds

Cache hit rate:       21 percent
Queue depth:          increasing

5xx rate:              1.2 percent
Checkout success:      91 percent
```

One API instance reports:

```text
/ready -> 503
```

### Task 1: Identify the bottleneck

The strongest primary bottleneck is MongoDB query or capacity pressure.

The strongest signals are:

```text
API CPU            -> 45 percent
MongoDB CPU        -> 94 percent
DB connections     -> 96 percent of limit
Query latency      -> 2.2 seconds
```

The API is waiting on a highly utilized database rather than being obviously CPU-bound.

### Task 2: Should you immediately add 20 API instances?

No. More instances could increase database pressure:

```text
More API instances
        |
        v
More database connections and queries
        |
        v
MongoDB saturation
        |
        v
Potentially worse latency
```

First investigate query plans, indexes, connection pools, cache misses, pagination and database capacity. Add API capacity only if evidence shows the API tier is also constrained and the database can support the additional load.

### Task 3: What do you do with the unhealthy API instance?

A readiness response of 503 should remove the instance from new traffic:

```text
Readiness = 503
      |
      v
Remove from traffic
      |
      v
Investigate logs and dependencies
      |
      v
Recover or recreate if appropriate
      |
      v
Wait for readiness to pass
      |
      v
Return to service
```

Do not send traffic to the instance while it cannot safely serve requests. Do not restart every healthy API instance merely because one instance is unready.

### Task 4: What do you investigate in MongoDB?

Investigate:

```text
Slow queries
Indexes
Query plans
Documents examined
Documents returned
Connection pools
Read and write patterns
Pagination
Projection
Database capacity
```

Also determine whether repeated reads are suitable for caching and whether the low cache hit rate is caused by poor keys, short TTLs, evictions, invalidation behavior or an unsuitable workload.

### Task 5: Why is cache hit rate important?

A cache hit rate of 21 percent means most requests are missing the cache. If this workload is suitable for caching, improving cache effectiveness could reduce MongoDB pressure.

Before changing the cache, validate:

```text
What data is being cached?
Is stale data acceptable?
Why are misses occurring?
What is the TTL?
Are cache keys complete and normalized?
Are evictions or memory limits causing misses?
Could shared caching expose private data?
```

Do not optimize the hit rate by caching data that must remain fresh or private.

### Task 6: What does increasing queue depth mean?

If incoming work is greater than worker processing capacity:

```text
Incoming work > worker capacity
          |
          v
Queue depth increases
```

Potential responses include:

```text
More workers
Faster processing
Batching
Workload reduction
Backpressure
Dead-letter and retry review
```

However, do not confuse the queue signal with the primary database bottleneck. Worker throughput may be low because workers are also waiting on MongoDB.

### Task 7: What should you monitor after mitigation?

Continue monitoring:

```text
API p95 and p99
5xx and timeout rate
MongoDB CPU
Database connections
Query latency
Documents examined
Cache hit rate
Queue depth and job age
Checkout success
Readiness failures
```

Checkout success must remain in the dashboard. Infrastructure can look healthy while the business workflow remains broken.

### Recommended mitigation sequence

```text
Establish impact and freeze risky changes
          |
          v
Remove unready instance from traffic
          |
          v
Protect checkout and other critical paths
          |
          v
Inspect MongoDB queries, connections and capacity
          |
          v
Improve safe cache hits or reduce unnecessary reads
          |
          v
Control queue growth and worker retries
          |
          v
Load test or validate the smallest safe change
          |
          v
Monitor technical and business recovery
```

---

## 12. Final Interview Challenge

Answer this without looking back:

> Your MERN application suddenly becomes slow during a traffic spike. Walk me through your response.

A strong answer should follow this structure:

```text
1. Establish impact
          |
          v
2. Check metrics and alerts
          |
          v
3. Identify the bottleneck
          |
          v
4. Protect users
          |
          v
5. Scale or optimize the constrained layer
          |
          v
6. Check dependencies
          |
          v
7. Verify recovery
          |
          v
8. Document the incident
```

A polished interview response is:

> I would first establish user and business impact using error rate, p95 and p99 latency, checkout success and other key business metrics. I would inspect each layer - load balancer, API, cache, database, queue and external dependencies - to identify the actual bottleneck. I would not automatically add API instances because that could increase load on MongoDB. If an instance is unhealthy, I would remove it from traffic through readiness checks. I would then optimize or scale the constrained component, use caching or asynchronous processing where appropriate, and monitor the system after mitigation. Finally, I would verify recovery using both technical and business metrics and document preventive improvements.

The important pattern is evidence, mitigation, validation and learning.

---

## 13. Monthly Cumulative Real-World Project

This month's cumulative project is now a Production-Grade MERN DevOps Platform.

Suggested structure:

```text
mern-devops/
|
+-- application/
+-- docker/
+-- ci-cd/
+-- infrastructure/
|
+-- docs/
|   +-- HIGH-AVAILABILITY.md
|   +-- INCIDENT-RUNBOOK.md
|   +-- RELIABILITY-RUNBOOK.md
|   +-- PERFORMANCE-RUNBOOK.md
|   +-- CAPACITY-PLAN.md
|   +-- SCALING-ARCHITECTURE.md
|
+-- monitoring/
    +-- dashboards/
    +-- alerts/
    +-- logging/
```

### Monthly project objective

Demonstrate that the MERN application can:

```text
Build
  |
  v
Test
  |
  v
Scan
  |
  v
Deploy
  |
  v
Scale
  |
  v
Observe
  |
  v
Detect failures
  |
  v
Recover
  |
  v
Handle traffic spikes
  |
  v
Protect users
```

### Required operational evidence

The project should contain evidence of:

- Automated tests and repeatable builds.
- Versioned Docker artifacts.
- Security scanning and promotion gates.
- Health and readiness checks.
- Load balancing and stateless API behavior.
- Centralized structured logs.
- Actionable alerts with documented ownership.
- Dashboards for latency, errors, saturation and business outcomes.
- Capacity and scaling decisions based on measurements.
- Performance tests for important endpoints.
- Incident, reliability and performance runbooks.
- A rollback or feature-flag mitigation path.

### Final monthly scenario

Simulate this sequence:

```text
New deployment
      |
      v
Five-times traffic spike
      |
      v
One API instance fails
      |
      v
MongoDB becomes saturated
      |
      v
Latency increases
      |
      v
Checkout success falls
```

The system should have a documented response for every stage:

```text
Deployment marker and release owner
      |
      v
Traffic and SLO monitoring
      |
      v
Readiness-based failure isolation
      |
      v
Database bottleneck investigation
      |
      v
Performance and capacity mitigation
      |
      v
Business-metric verification
      |
      v
Incident record and corrective actions
```

That is the level of scenario-based thinking expected in a DevOps interview.

---

## 14. Week 9 Takeaways

The most important connections from this week are:

```text
Logging
    |
    v
Detection

Metrics
    |
    v
Diagnosis

Alerts
    |
    v
Action

Health checks
    |
    v
Failure isolation

Load balancing
    |
    v
High availability

Horizontal scaling
    |
    v
Capacity

Caching
    |
    v
Reduced repeated work

Indexes and query optimization
    |
    v
Reduced database work

Queues
    |
    v
Asynchronous processing and burst absorption

Incident response
    |
    v
Mitigation, recovery and learning
```

### The interview principle to remember

> Do not treat DevOps as a collection of tools. Treat it as a system for safely delivering, operating, scaling, observing and recovering production software.

When a production system slows down, do not guess from one metric. Establish impact, correlate signals across layers, identify the constrained component, protect users, validate recovery and record what should improve next time.

---

## 15. Next Topic

Today completes the Days 50-56 weekly revision. The syllabus now resumes with **SLOs, SLIs, SLAs and error budgets**, connecting MERN production metrics to measurable reliability targets.
