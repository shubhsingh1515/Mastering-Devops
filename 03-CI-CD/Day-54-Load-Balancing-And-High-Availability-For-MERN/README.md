# DevOps Mentorship Program - Day 54

## Phase 3: Production Operations

### Load Balancing and High Availability for MERN
 
**Focus:** Load balancing, stateless MERN APIs, health-aware routing, sessions, scaling and high-availability troubleshooting

Yesterday, Day 53, you learned how health checks and self-healing work:

```text
Process failure
    -> Restart

Readiness failure
    -> Remove from traffic

Dependency failure
    -> Isolate and investigate
```

Today we build the next layer:

> How do we keep a MERN application available when traffic increases or individual API instances fail?

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain why load balancing improves availability.
- Distinguish vertical and horizontal scaling.
- Understand health-aware traffic routing.
- Explain why production APIs should generally be stateless.
- Understand the problem with in-memory sessions.
- Explain sticky sessions and their tradeoffs.
- Design a highly available MERN API.
- Troubleshoot intermittent load-balancing failures.
- Answer high-availability questions in DevOps interviews.

---

## 2. Why One API Server Is Not Enough

With one API instance, a process failure can make the whole application unavailable:

```text
Users
  |
  v
Node/Express API
  |
  v
Failure = no API capacity
```

Multiple instances provide spare capacity:

```text
             Load Balancer
             /     |     \
            /      |      \
         API-1   API-2   API-3
```

If API-2 fails, the load balancer can continue routing to API-1 and API-3. High availability does not mean that every component never fails. It means that an expected component failure does not necessarily become a user-visible outage.

---

## 3. Vertical and Horizontal Scaling

### Vertical scaling

Vertical scaling makes one machine larger:

```text
2 CPU / 4 GB RAM
        |
        v
8 CPU / 32 GB RAM
```

It is often simple at the beginning, but the machine still has an upper limit and remains a single failure domain. Some vertical changes also require a restart or maintenance window.

### Horizontal scaling

Horizontal scaling adds more application instances:

```text
1 API instance
      |
      v
3 API instances
      |
      v
10 API instances
```

It is especially effective for stateless HTTP APIs because any healthy instance can handle the next request.

Important constraint:

> Scaling the API tier does not automatically scale MongoDB, Redis, payment providers or network capacity.

---

## 4. What a Load Balancer Does

A load balancer sits between users and backend instances. Depending on the platform, it can provide:

- Traffic distribution.
- Health checking.
- TLS termination.
- Connection management.
- Host or path routing.
- Failure isolation.
- Weighted traffic for canary releases.

Common distribution strategies include:

### Round robin

```text
Request 1 -> API-1
Request 2 -> API-2
Request 3 -> API-3
Request 4 -> API-1
```

### Least connections

New work is sent toward an instance with fewer active connections.

### Weighted routing

```text
API-v1 -> 95%
API-v2 -> 5%
```

Weighted routing is useful for canaries, controlled releases and instances with unequal capacity.

A load balancer without health checks can repeatedly send requests to a broken backend. That creates intermittent failures that look random to users but are predictable when correlated with instance identity.

---

## 5. Health-Aware Routing

The load balancer should use readiness, not only process or container state:

```text
API-1 -> /ready 200 -> receives traffic
API-2 -> /ready 503 -> removed from new traffic
API-3 -> /ready 200 -> receives traffic
```

When API-2 fails readiness, existing requests may drain if the platform supports it. The load balancer continues probing API-2, and it can return to service only after readiness is healthy for the agreed stability window.

Readiness answers:

> Can this instance safely receive new traffic right now?

Liveness answers:

> Is this process alive enough to continue running?

Keep those decisions separate. A MongoDB outage may make an instance unready without making a restart useful.

---

## 6. MERN High-Availability Architecture

A basic production topology is:

```text
                         Internet
                            |
                            v
                     Load Balancer
                            |
               +------------+------------+
               |            |            |
               v            v            v
             API-1        API-2        API-3
               |            |            |
               +------------+------------+
                            |
                            v
                         MongoDB
```

React static assets may be served separately while the browser sends API requests through the load balancer:

```text
Browser
  +-> React static assets
  +-> API requests -> Load Balancer -> Node/Express replicas
```

The API instances should use the same compatible image, configuration contract and database schema expectations. They should expose instance identity and version in logs so failures can be correlated with a specific backend.

---

## 7. Stateless APIs

An API is stateless when it does not depend on private in-memory state from an earlier request in order to handle the next request.

A fragile design looks like this:

```text
Request 1 -> API-1
              |
              +-> session stored only in memory

Request 2 -> API-2
              |
              +-> session missing
```

A scalable design keeps shared state in an external system:

```text
API-1 +--+
API-2 +--+-> Shared state store
API-3 +--+
```

MongoDB can store durable application data. A dedicated session or cache store can hold ephemeral shared state when the application needs it. JWTs can reduce server-side session state, but they do not remove every stateful concern. Rate limits, caches, WebSocket connections, temporary workflows and background jobs still need deliberate designs.

The key question is:

> Can any healthy API instance safely handle the next request?

---

## 8. Sessions and Sticky Sessions

A sticky session routes a client repeatedly to the same backend:

```text
User A -> Load Balancer -> API-1
User A -> Load Balancer -> API-1
```

This can make in-memory sessions appear to work, but it has costs:

- If API-1 dies, its in-memory session is lost.
- Traffic can become unevenly distributed.
- Scaling and failover become less flexible.
- Rolling deployments must preserve client affinity.

For many production APIs, a better design is:

```text
Browser
   |
   v
Load Balancer
   |
   v
Any healthy API instance
   |
   v
Shared session or state system
```

Sticky sessions are a deliberate tradeoff, not a substitute for designing shared state correctly.

---

## 9. Failure, Recovery and Deployment

When an instance crashes:

```text
API-1 OK   API-2 FAILED   API-3 OK
                 |
                 v
       Readiness or health failure
                 |
                 v
      Traffic goes to API-1 and API-3
                 |
                 v
       Auto-recovery replaces API-2
                 |
                 v
       New API-2 passes readiness
```

During a rolling deployment, a new version must pass liveness, readiness and representative smoke checks before receiving production traffic:

```text
v1 -> v1 -> v1
       |
       v
v2 ready -> v1 -> v1
       |
       v
v2 -> v2 -> v1
       |
       v
v2 -> v2 -> v2
```

For a canary, weighted routing can move traffic gradually:

```text
v1 -> 95%
v2 -> 5%
```

Monitor 5xx rate, latency, dependency errors and business signals. Increase the v2 weight only when the evidence supports it. Roll back or set v2 to 0% when it is unhealthy.

---

## 10. Database Bottlenecks

Adding API replicas can increase pressure on MongoDB:

```text
More API instances
        |
        v
More database connections
        |
        v
MongoDB saturation
        |
        v
Higher latency and timeouts
```

Monitor:

- Connection pool usage.
- Query latency and slow queries.
- CPU and memory.
- Disk and I/O.
- Index effectiveness.
- Replication health.
- Error and timeout rates.

If three API instances become ten while MongoDB is already at its connection limit, the application can become slower rather than faster. Scaling must follow the bottleneck, not just the visible tier.

---

## 11. Troubleshooting Intermittent 5xx Errors

When users report that retries often work, ask:

1. Are requests distributed across multiple instances?
2. Is one instance unhealthy or running a different version?
3. Are health checks configured and honored?
4. Do logs differ by instance and version?
5. Is shared state available to every replica?
6. Is MongoDB or another dependency saturated?

Example evidence:

```text
api-1 -> v11 -> normal
api-2 -> v10 -> 18% errors
api-3 -> v11 -> normal
```

Likely diagnosis:

```text
Load balancer still routes to api-2
        |
        v
Outdated or unhealthy instance serves requests
        |
        v
Users see intermittent failures
```

Immediate mitigation is to remove API-2 from traffic, preserve evidence, stop the inconsistent rollout and investigate why readiness or deployment controls failed.

Structured logs should include at least:

```json
{
  "service": "mern-api",
  "instance": "api-2",
  "version": "11.0.7",
  "requestId": "83ad91",
  "status": 500
}
```

---

## 12. Interview Answers

### How would you make a Node.js API highly available?

I would run multiple stateless API instances behind a load balancer with health-aware routing. I would use liveness and readiness checks, graceful shutdown and a rolling or canary deployment strategy. Shared state would live outside individual processes where needed. I would monitor both the API and its dependencies, especially MongoDB, and verify that the database layer can handle the expected load.

### Why should a production API be stateless?

Statelessness allows any healthy instance to handle the next request. That makes horizontal scaling, load balancing, failover and rolling deployments easier. State that must be shared belongs in an appropriate external system rather than only in one process's memory.

### Why can performance worsen after scaling from 3 to 20 instances?

A downstream dependency may be saturated. More API instances can create more MongoDB connections, increase query concurrency, exhaust external API limits or consume network capacity. I would inspect the complete request path instead of assuming that adding application replicas solves every bottleneck.

---

## 13. Practical Exercise

Create a high-availability design for your MERN project and document:

```text
Load balancing:
    How is traffic distributed?

Health:
    Which endpoint determines readiness?

Failure:
    What happens when API-2 crashes?

Scaling:
    How do you add API-4 safely?

Sessions:
    Where does shared session or state live?

Deployment:
    How do you introduce a new version?

Observability:
    How do you identify which instance failed?
```

Use the deliverable in `HIGH-AVAILABILITY.md` as the design record.

---

## 14. Review Questions

1. What is the difference between liveness and readiness?
2. Why does a stateless API scale more easily?
3. What should happen when one backend returns readiness `503`?
4. Why can sticky sessions hide a design problem?
5. Why can more API replicas overload MongoDB?
6. What evidence identifies one bad backend during intermittent failures?
7. Why should a canary receive weighted traffic gradually?
8. What does graceful shutdown protect during replacement?

---

## Day 54 Summary

```text
Horizontal scaling
        +
Stateless application
        +
Health-aware load balancing
        +
Shared state strategy
        +
Observability
        +
Self-healing
        =
A stronger high-availability foundation
```

High availability is not simply running more servers. It is designing the system so failures are isolated, unhealthy instances stop receiving traffic, healthy capacity continues serving users and the shared database is monitored as carefully as the API tier.

### Next Lesson

Day 55: scaling MERN workloads and capacity planning, including API capacity, database capacity, caching, queues and background workers.
