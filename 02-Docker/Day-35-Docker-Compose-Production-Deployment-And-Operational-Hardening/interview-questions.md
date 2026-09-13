# Day 35 - Interview Questions: Production Compose and Operational Hardening

## Beginner Level

### Q1. Which service should normally be the public entry point in a production MERN Compose architecture?

Nginx or another reverse proxy. The API and MongoDB should normally remain on private Docker networks.

### Q2. Why should MongoDB generally not be publicly exposed?

Only trusted application components normally need access to it. Keeping it internal reduces unnecessary attack surface and prevents direct public database access.

### Q3. Why use Docker service names instead of container IP addresses?

Service names provide stable logical discovery. Container IP addresses can change whenever services are recreated.

### Q4. What does a named volume provide for MongoDB?

It provides persistence so data can survive container replacement. It is not automatically a backup system.

## Intermediate Level

### Q5. What is the difference between persistence and backup?

Persistence allows data to survive container replacement. Backup creates recoverable copies that protect against deletion, corruption, and destructive events.

### Q6. What does a restart policy do?

It tells Docker whether and when to restart a container after it exits. It does not repair application bugs or invalid configuration.

### Q7. Why is container `Up` not enough to prove application health?

The process can be running while the application cannot serve requests or communicate with a required dependency. Health checks and real request tests provide stronger evidence.

### Q8. Why should the API not necessarily publish port `3000`?

If Nginx is the only client, it can reach `api:3000` through the private network. Publishing the API port creates unnecessary host exposure.

### Q9. Why should application logs normally go to stdout and stderr?

The container runtime can capture and forward them to centralized logging. Files inside ephemeral containers can disappear when containers are replaced unless persistence is designed explicitly.

## Advanced Level

### Q10. How would you make a Compose-based MERN stack more production-ready?

I would use a public reverse proxy, private internal services, externalized configuration, secret management, health checks, intentional restart policies, graceful shutdown, non-root users, minimal images, persistent storage, backups, resource controls, structured logs, and versioned images.

### Q11. What happens when a restart policy repeatedly restarts a broken application?

It creates a restart loop or restart storm. I would inspect logs, exit state, configuration, dependencies, health checks, network connectivity, and resource usage to identify the root cause.

### Q12. Can a Docker volume replace a backup strategy?

No. A volume provides persistence but does not protect against accidental deletion, corruption, or destructive database operations. Backups and restore testing are separate requirements.

### Q13. Can Docker Compose alone guarantee zero-downtime deployment?

No. Compose can define services and support health and restart behavior, but reliable rolling replacement needs a deliberate deployment strategy, traffic management, multiple instances, a service manager, or an orchestration platform.

### Q14. Why is `stop_grace_period` useful?

It gives an application time to handle `SIGTERM`, stop accepting new work, finish active requests, close database connections, and exit before stronger termination is used.

### Q15. How should you design health endpoints?

Separate liveness from readiness when useful. Liveness checks whether the process should remain alive; readiness checks whether it can receive traffic and use required dependencies. Do not expose secrets or perform expensive operations.

### Q16. Why are resource limits and monitoring important?

A memory leak or runaway process can affect unrelated services and the host. Resource boundaries and `docker stats` make pressure easier to detect and contain.

## Scenario Question

> Users can load the website, but all API requests fail. Compose shows Nginx, API, and MongoDB as `Up`. What do you do?

A strong answer:

```text
1. Check the actual user request path.
2. Read Nginx logs for a 502 or upstream error.
3. Read API logs.
4. Validate docker compose config.
5. Verify MONGO_URI uses mongodb, not localhost.
6. Confirm network membership and service-name DNS.
7. Test api:3000 from Nginx.
8. Check MongoDB health and resource state.
9. Identify the root cause before restarting.
10. Apply a targeted fix and retest.
```

`Up` only proves that the container process is running; it does not prove that the application path works.

## Strong Short Answers

**Why keep MongoDB internal?**  
Its trusted client is the API, so public exposure is unnecessary risk.

**Why not hard-code container IPs?**  
Container IPs can change; service names provide stable discovery.

**Is a volume a backup?**  
No. It provides persistence, while backups provide recoverable copies.

**What is an operationally hardened container?**  
A container with limited exposure, minimal privileges, external configuration, health signals, controlled recovery, clean shutdown, useful logs, and resource boundaries.

**What is the first troubleshooting sequence?**  
Logs, state, health, network, configuration, dependencies, resources, root cause.

**Can Compose alone promise zero downtime?**  
No. It needs a deliberate deployment and traffic-management strategy.
