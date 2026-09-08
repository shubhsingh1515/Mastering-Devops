# Day 31 - Interview Questions: Production-Ready Docker Containers

## Beginner Level

### Q1. What makes a Docker image production-ready?

It should be reproducible, appropriately small, secure, versioned, limited to required runtime dependencies, free of baked secrets, and easy to operate and troubleshoot.

### Q2. Why use `.dockerignore`?

It keeps unnecessary, local, and sensitive files out of the Docker build context and reduces accidental inclusion in the image.

### Q3. Why copy `package*.json` before application source?

Dependency manifests change less frequently than source code. Copying them first allows Docker to reuse the dependency installation layer when only source changes.

### Q4. Why use `npm ci` when a lockfile exists?

It performs a lockfile-based, reproducible installation and is better suited to automated builds than an unconstrained dependency installation.

---

## Intermediate Level

### Q5. Why externalize application configuration?

The same immutable image can be promoted across development, staging, and production while each environment supplies its own configuration at runtime.

### Q6. Why should Node run as a non-root user?

To follow least privilege and reduce the potential impact of a compromised application process.

### Q7. What is the difference between a running and healthy container?

A running container has an active main process. A healthy container passes a defined application health condition.

### Q8. Why are multi-stage builds useful?

They separate build tools and development dependencies from the final runtime image, reducing size and attack surface.

### Q9. Why should production containers normally log to stdout and stderr?

The container runtime and hosting platform can collect and centralize those streams. Unmanaged files can consume container storage and disappear during replacement.

### Q10. Why is manually modifying a production container bad practice?

The changes are not represented in source or the image, disappear on recreation, and cannot be reliably reproduced. Modify the Dockerfile or source, build a new image, test it, and redeploy.

---

## Advanced Level

### Q11. What happens when a container receives `SIGTERM`?

The runtime requests termination. A production application should stop accepting new work, finish in-flight requests where possible, close database and other resource connections, close the HTTP server, and exit cleanly.

### Q12. Why does PID 1 matter in a container?

The main process has special signal and child-process responsibilities. Using an exec-form command helps the actual application receive signals directly. Complex process trees may require an init process for child reaping.

### Q13. How do restart policies help and fail?

They can recover from transient process failures, but they do not fix application bugs. A persistent failure can create a restart loop, so logs, health checks, alerting, and root-cause analysis are still required.

### Q14. How do you choose resource limits?

Measure normal and peak workload, observe CPU and memory behavior, define limits that protect the host without starving the application, and validate the effective runtime configuration.

### Q15. How would you make a MERN Docker deployment reproducible?

Use Dockerfiles, lockfiles, cache-friendly layer ordering, versioned images, external runtime configuration, immutable artifacts, persistent storage, health checks, and documented Compose or deployment configuration.

### Q16. How do you prevent secrets from entering image history?

Do not place secrets in Dockerfile `ENV`, `ARG`, `RUN` commands, copied `.env` files, source control, or build logs. Inject them through protected runtime configuration or a secret-management system.

### Q17. How would you investigate a Node container that fails during deployment with `SIGTERM`?

Check whether the application handles the signal, whether readiness is removed before termination, whether active requests drain, whether database connections close, whether a timeout is configured, and whether the process receiving the signal is the actual Node process.

## Scenario Questions

### Scenario 1: The image is huge and builds slowly

Investigate:

```text
Dockerfile instruction order
        |
        v
Build context and .dockerignore
        |
        v
Dependency installation
        |
        v
Development dependencies
        |
        v
Multi-stage opportunities
        |
        v
Base image and cache behavior
```

Then measure image size, build duration, cache reuse, and runtime behavior after each change.

### Scenario 2: The container is running but requests fail during replacement

The likely missing control is graceful shutdown. Add `SIGTERM` handling, stop accepting new requests, drain active requests, close MongoDB, close the HTTP server, and verify behavior with `docker stop` and logs.

### Scenario 3: The security team asks why the API container is trusted

Explain the controls and their limits:

- maintained suitable base image
- production dependencies only
- non-root process
- no baked secrets
- dropped capabilities where tested
- read-only filesystem where practical
- resource limits
- image scan
- version or digest traceability
- health and runtime monitoring

No single control makes the container secure.

## Short Strong Answers

**Why not `latest`?**  
It is mutable. Use a specific version, commit tag, or digest.

**Why use `USER node`?**  
To reduce process privileges and limit compromise impact.

**Why handle `SIGTERM`?**  
To release resources and finish work cleanly during replacement or deployment.

**Why not modify a running container?**  
The change is not reproducible and disappears when the container is recreated.

**What is the production image goal?**  
A small, reproducible, versioned, secure, health-aware runtime artifact with no baked secrets.
