# Day 54 Interview Questions: Load Balancing and High Availability

## 1. How would you make a Node.js API highly available?

I would run multiple stateless API instances behind a load balancer with readiness-aware routing. I would use liveness and readiness checks, graceful shutdown and a rolling or canary deployment strategy. Shared state would live in an external system where necessary. I would monitor both API and dependency health, especially MongoDB capacity and connection pressure.

## 2. What is the difference between liveness and readiness?

Liveness determines whether the process is alive enough to continue running. Readiness determines whether the instance should receive new traffic. A readiness failure usually removes an instance from service but does not automatically justify restarting it.

## 3. Why should a production API be stateless?

Statelessness lets any healthy instance handle the next request. This simplifies horizontal scaling, load balancing, failover and rolling deployments. State that must be shared should be stored in a suitable external system instead of only in one process's memory.

## 4. What is a sticky session and why is it not always ideal?

A sticky session repeatedly routes a client to the same backend. It can support in-memory sessions, but it reduces routing flexibility and can lose session state when that instance fails. A shared session store is usually more resilient for a horizontally scaled service.

## 5. What should happen when one replica becomes unhealthy?

The load balancer should stop sending new requests to it while continuing to probe it. Healthy replicas should serve traffic. The team should inspect the failed instance, version, dependency access and logs. It is not automatically necessary to restart every replica.

## 6. Why can a load balancer cause intermittent 5xx errors?

If it lacks effective health checks, it may continue routing requests to one broken backend. Errors appear intermittent because requests routed to healthy instances succeed. Instance ID, version and request ID in structured logs make this pattern visible.

## 7. Why can scaling from three to twenty API instances make performance worse?

The API tier may put more pressure on a shared bottleneck such as MongoDB connections, query capacity, an external provider, CPU, memory or network limits. I would compare request load with downstream saturation before adding more replicas.

## 8. How would you design a canary release?

I would start a compatible new version, wait for liveness and readiness, send a small weighted percentage of traffic to it and monitor error rate, latency, dependency health and business outcomes. I would increase the weight only after a stability window. If signals degrade, I would set the canary weight to zero and investigate.

## 9. What should happen when MongoDB is unavailable to one API instance?

If MongoDB is critical for the instance's routes, it should fail readiness and leave new traffic. I would investigate DNS, network policy, TLS, credentials, configuration and database health. Restarting all API replicas would not restore a shared database dependency and could amplify the outage.

## 10. Why is graceful shutdown important during deployment?

The instance should leave traffic, stop accepting new work, finish active requests, close dependency connections and exit within its deadline. This reduces dropped requests, partial work and duplicate business operations during replacement.

## 11. How do you safely add API-4?

Start it with the same compatible image and configuration contract, verify liveness, initialize dependencies, wait for readiness, run smoke checks and then allow the load balancer to include it. Monitor capacity, latency, errors and MongoDB connections after the change.

## 12. What logs help diagnose an instance-specific failure?

At minimum: service name, environment, instance ID, version or image digest, request ID, route, status code, latency and a safe error classification. These fields allow failures to be grouped by backend and release.

## 13. How would you answer an outage scenario with one crashed instance and one outdated instance?

I would preserve evidence, remove the outdated or unhealthy instance from traffic, confirm remaining capacity, pause the rollout and compare image versions and readiness results. I would replace or repair only the affected instance, then reintroduce it after a stability window and verify user and business metrics.

## 14. What does high availability not guarantee?

It does not guarantee zero bugs, zero downtime for every failure, unlimited capacity or database availability. It reduces the impact of defined failure modes and requires redundancy, health-aware routing, dependency planning, observability and tested recovery procedures.
