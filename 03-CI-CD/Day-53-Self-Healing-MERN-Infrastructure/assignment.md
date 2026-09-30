# Day 53 Assignment: Design a Self-Healing MERN API

## Objective

Design health checks and recovery behavior for a production MERN application. Your design must distinguish process liveness, traffic readiness, dependency failure and restartable process failure.

## Scenario

Your production system contains a load balancer, three Node/Express API replicas and MongoDB:

```text
Load Balancer
   |       |       |
 API-1   API-2   API-3
   \       |       /
        MongoDB
```

A deployment is replacing API-2. At the same time, MongoDB becomes temporarily unavailable to API-2 only.

```text
API-1: /health 200, /ready 200
API-2: /health 200, /ready 503
API-3: /health 200, /ready 200
```

## Required Deliverables

1. Create `RELIABILITY-RUNBOOK.md` with liveness, readiness, dependency, restart, load-balancer, shutdown and recovery sections.
2. Define `/health` and `/ready` response behavior, including status codes.
3. Classify MongoDB, payment, Redis and email as critical or optional for selected workflows.
4. Define a Docker restart policy and explain its limits.
5. Describe load-balancer behavior when one replica is unready.
6. Describe graceful shutdown during a rolling deployment.
7. Document at least five failure scenarios and the correct first response.
8. Define the evidence required before declaring recovery.

## Design Questions

Answer these in your notes:

1. What is the difference between liveness and readiness?
2. Why should liveness usually avoid checking MongoDB?
3. What HTTP status should `/ready` return when a critical dependency is unavailable?
4. Should a readiness failure always restart the process? Why or why not?
5. Which dependency failures should remove the complete API from traffic?
6. What is the danger of making optional email service a global readiness dependency?
7. Why can a restart policy fail to solve a process that remains alive but returns HTTP 500?
8. How do you recognize a restart loop?
9. What should happen before a new version receives production traffic?
10. Why does graceful shutdown reduce dropped requests?
11. What signals prove that a recovered replica is safe to return to service?
12. How can self-healing amplify an external dependency outage?

## Failure Drill

Write the response in this order:

1. Confirm `/health` and `/ready` for all replicas.
2. Verify that API-2 is removed from new traffic.
3. Confirm API-1 and API-3 have sufficient healthy capacity.
4. Investigate MongoDB reachability, DNS, configuration, network policy and database health.
5. Avoid restarting all API replicas.
6. Stop the rollout if the new version is involved.
7. Restore the dependency or correct the instance configuration.
8. Confirm API-2 readiness remains healthy for a stability window.
9. Reintroduce API-2 gradually.
10. Verify technical and business metrics.

## Completion Criteria

- Liveness and readiness have separate purposes and endpoints.
- `/health` is lightweight and does not expose secrets.
- `/ready` returns `503` when a required dependency prevents safe traffic.
- Optional dependencies do not unnecessarily remove the entire service.
- Restart policy is paired with logs, metrics, alerting and backoff.
- Unready instances receive no new load-balancer traffic.
- Rolling deployment waits for readiness before promotion.
- Graceful shutdown drains active work and closes resources.
- Recovery verification includes user-facing and business signals.
- The design avoids restarting every replica for a dependency outage.
