# Day 37 - Interview Questions: Docker Production Deployment Strategies & Rollback

## Beginner Level

### Q1. What is a Docker image tag?

A Docker image tag is a human-readable reference attached to an image, such as `v1.2.0`, `latest`, or a commit identifier like `abc1234`.

### Q2. Why is `latest` risky in production?

Because `latest` is mutable and does not uniquely identify a specific release. It changes over time, which makes troubleshooting and rollback harder.

### Q3. What is a rollback?

A rollback is the process of returning a system to a known-good previous version after a new deployment fails or causes issues.

## Intermediate Level

### Q4. What does “build once, deploy many” mean?

It means the application is built once as a tested artifact and then promoted through environments without being rebuilt differently for each environment.

### Q5. Why should production deployment use versioned images instead of a moving tag?

It improves traceability, enables safer rollback, and makes the deployed artifact reproducible and easier to reason about during incidents.

### Q6. What is an immutable image reference?

An immutable image reference is a release identifier that does not change over time, such as a version tag or image digest.

### Q7. What is the difference between “container started” and “application healthy”?

A container can be running but still not ready to serve traffic. Health checks confirm that the service meets expected readiness criteria.

### Q8. Why is health checking required before a release is accepted?

Because a new version may be started but still fail liveness or readiness checks. Without health validation, traffic can be sent to a failing service.

## Advanced Level

### Q9. Why is application rollback not always enough?

Because the application may depend on a database schema that is not compatible with the previous version. In that case, a rollback of the app alone may fail or create further issues.

### Q10. How do you make database migrations safer for rollback?

Use backward-compatible migrations, expand-and-contract patterns, and avoid removing old structures before the old application version is fully retired.

### Q11. What is an expand-and-contract migration?

It is a migration strategy where you add the new schema or field first, deploy compatible application versions, migrate data, and only then remove the deprecated structure.

### Q12. What are blue-green deployments?

Blue-green deployment uses two environments or versions, usually Blue and Green, and switches traffic between them to reduce deployment risk and support fast rollback.

### Q13. What are canary deployments?

Canary deployments send a small portion of traffic to the new version first. If it is healthy, the traffic is gradually increased.

### Q14. Why should deployment metadata be recorded?

Because teams need to know which version was deployed, when it was deployed, and what commit or digest corresponds to the release. This helps with diagnosis and rollback.

## Scenario Questions

### Q15. A new API version is running, but `/health` returns 500. Should you continue sending production traffic?

No. The service is not ready, and a health gate should block traffic until the service is healthy.

### Q16. A production deployment fails after a database schema change. What should the team do?

The team should confirm the failure, inspect logs and health metrics, verify whether the rollback target is valid, and then decide whether to restore the previous application version while also evaluating database compatibility.

## Strong Answer Template

### How would you explain your Docker deployment strategy in an interview?

> “We build a versioned image from a specific Git revision and tag it with a commit or release identifier. We test it, push it to a registry, and then promote the exact same artifact through environments. Before traffic is accepted, we validate health checks and operational readiness. We retain the previous known-good image so rollback is fast and deterministic. We also design database migrations to be backward-compatible so app rollback does not immediately break the schema.”

### Why is rollback important?

> Because production systems fail, and the fastest safe recovery path is to return to a known-good version instead of improvising during an incident.

### What is the difference between app rollback and database rollback?

> The application rollback is a process of restoring a previous build, but the database may need compatibility planning because schema changes might be destructive or incompatible with the older release.
