# Day 54 Assignment: Design a Highly Available MERN API

## Objective

Design a production MERN API that can continue serving users when one API instance fails. Your design must cover traffic routing, health checks, statelessness, shared state, scaling, deployment and database bottlenecks.

## Scenario

Your system contains a load balancer, three Node/Express API replicas and MongoDB:

```text
Load Balancer
   |       |       |
 API-1   API-2   API-3
   \       |       /
        MongoDB
```

Production reports intermittent HTTP 500 responses:

```text
api-1 -> v11 -> normal
api-2 -> v10 -> 18% errors
api-3 -> v11 -> normal
```

## Required Deliverables

1. Create `HIGH-AVAILABILITY.md` with the ten required architecture sections.
2. Define how the load balancer distributes traffic and excludes unhealthy replicas.
3. Define `/health` and `/ready` behavior, including status codes.
4. Explain what happens when API-2 crashes or fails readiness.
5. Explain how API-4 can be added without sending traffic before it is ready.
6. Document the session and shared-state strategy.
7. Define rolling or canary deployment behavior.
8. Identify MongoDB capacity risks and the metrics you will monitor.
9. Define instance-level structured logging fields.
10. Write a recovery procedure and evidence checklist.

## Design Questions

1. Why does horizontal scaling improve availability?
2. What is the difference between liveness and readiness?
3. Why should a production API avoid private in-memory session state?
4. When might sticky sessions appear useful, and what do they cost?
5. What should happen when API-2 returns readiness `503`?
6. Why can adding API replicas make MongoDB slower?
7. How should a canary receive traffic?
8. What evidence proves a recovered instance is safe to reintroduce?
9. Which fields identify the backend responsible for a failed request?
10. Why does graceful shutdown matter during rolling deployment?

## Failure Drill

Write your response in this order:

1. Confirm `/health` and `/ready` for every replica.
2. Determine whether errors are concentrated by instance or version.
3. Remove API-2 from new traffic.
4. Confirm API-1 and API-3 have enough healthy capacity.
5. Pause the inconsistent rollout.
6. Investigate API-2 configuration, image, DNS, network and MongoDB access.
7. Avoid restarting every API replica.
8. Replace or repair API-2 through the approved deployment process.
9. Wait for a readiness stability window.
10. Reintroduce the instance gradually and verify technical and business metrics.

## Completion Criteria

- Multiple API instances sit behind a load balancer.
- Readiness, not only process status, controls traffic eligibility.
- Any healthy instance can handle the next request.
- Shared state is externalized or deliberately managed.
- Unhealthy instances receive no new traffic.
- New versions pass readiness before promotion.
- MongoDB capacity and connection pressure are monitored.
- Logs identify service, instance, version and request ID.
- Recovery includes user-facing and business validation.
- The design explains why adding API instances can worsen a downstream bottleneck.
