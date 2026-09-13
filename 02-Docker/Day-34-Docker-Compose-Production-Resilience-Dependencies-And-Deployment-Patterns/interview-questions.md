# Day 34 - Interview Questions: Compose Resilience and Deployment Patterns

## Beginner Level

### Q1. What is a Docker health check?

A command Docker runs periodically to test whether a defined service condition passes. Docker records the result as `starting`, `healthy`, or `unhealthy`.

### Q2. What is a restart policy?

A Docker setting that controls whether and under what conditions a container should automatically restart after it exits.

### Q3. Does `depends_on` guarantee that a service is ready?

No. It expresses a dependency or startup relationship. A service may have started while its application is still initializing or unable to accept connections.

### Q4. What does `docker compose ps` showing `Up` prove?

It proves that the container's main process is running. It does not prove that the application is healthy or that all dependencies are available.

## Intermediate Level

### Q5. Why is a health check useful?

It turns an operational assumption into a repeatable signal. Other tooling can distinguish a running container from one that can perform a defined health function.

### Q6. What is the difference between liveness and readiness?

Liveness asks whether the process is alive enough to continue. Readiness asks whether the service can currently receive production traffic.

### Q7. Why can a health check fail even when the application works?

The command may not exist in the runtime image, the URL may be wrong, the port may be wrong, or the check may be too strict or too shallow.

### Q8. What does `restart: on-failure` do?

It asks Docker to restart the container after a failure exit. It does not diagnose or fix the failure. A retry limit such as `on-failure:5` can bound a persistent restart loop.

### Q9. Why does MongoDB need an application-compatible health check?

Different MongoDB image versions may provide different client commands or output. The check must use tools and authentication settings supported by the selected image.

## Advanced Level

### Q10. How would you prevent an API from crashing because MongoDB starts slowly?

Use a MongoDB health check, a health-aware Compose dependency condition where supported, application connection retries, exponential backoff, timeouts, and a readiness state that stays false until the dependency is usable.

### Q11. Why is graceful shutdown important during deployment?

It allows the process to stop accepting new traffic, finish in-flight work where possible, close HTTP and database resources, and exit cleanly instead of abruptly dropping operations.

### Q12. What happens during a restart loop?

The process exits, Docker restarts it, it encounters the same failure, and exits again. The loop may keep the container technically active while the service remains unusable. Logs and exit state must be inspected to find the root cause.

### Q13. An API is healthy but users still receive errors. What does that tell you?

The health check may not cover the complete user request path or every dependency. Test the actual Nginx-to-API route and inspect application logs.

### Q14. How would you design a readiness endpoint?

Return success only when the service can perform the work required for traffic, such as reaching MongoDB. Return HTTP `503` when the process is alive but temporarily unable to serve correctly. Do not expose secrets or expensive operations.

### Q15. How would you troubleshoot an API that keeps restarting while MongoDB is `Up`?

I would inspect API logs, exit code, restart count, health state, resolved Compose configuration, `MONGO_URI`, network membership, service-name resolution, MongoDB readiness, health-check executables, listening ports, and resource usage. `Up` does not prove MongoDB is ready.

### Q16. What is a safe replacement deployment sequence?

Build the new version, start it, run health checks, verify the real request path, route traffic to it, gracefully terminate the old version, and monitor the result.

### Q17. Why should restart policies not be used as the only resilience mechanism?

They only react to process exits. They do not provide application retry logic, dependency readiness, traffic control, diagnosis, data protection, or observability.

## Scenario Question

> Your API container is `Restarting`, MongoDB is `Up`, and Nginx returns `502`. What do you do?

A strong answer:

```text
1. Read API logs and inspect exit state.
2. Check API restart count and health state.
3. Validate docker compose config.
4. Verify MONGO_URI uses mongodb, not localhost.
5. Confirm API and MongoDB share the expected network.
6. Resolve mongodb from inside the API container.
7. Check MongoDB health and logs.
8. Verify the API listening port and health endpoint.
9. Test api:3000 from Nginx.
10. Check memory and other resource failures.
```

I would not repeatedly restart the stack before collecting evidence.

## Strong Short Answers

**Why not trust `depends_on` alone?**  
Because startup order is not readiness.

**Why can `curl` break a health check?**  
The runtime image may not contain `curl`.

**What does readiness mean?**  
The service can currently receive and successfully process its required traffic.

**Why handle `SIGTERM`?**  
To stop accepting work, finish active requests where possible, close resources, and exit cleanly.

**What does a restart loop tell you?**  
The process is repeatedly failing; inspect the first useful error instead of treating restarts as a fix.

**What is a good first diagnostic sequence?**  
Logs, container state, health state, network, configuration, dependencies, and resource usage.
