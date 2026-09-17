# Day 40 - Interview Questions: Docker Production Readiness & Final Container Assessment

## Beginner Level

### Q1. What does production-ready Docker mean to you?

It means the containers are reproducible, appropriately secured, observable, health-checked, correctly networked, persistent where required, and supported by documented deployment, recovery, and rollback procedures.

### Q2. Which service should normally be the public boundary of a MERN deployment?

Nginx or another reverse proxy/load balancer should normally be the public boundary. The API and MongoDB should remain on private networks unless there is a specific reason to expose them.

### Q3. Why should MongoDB not normally be publicly exposed?

Only trusted application components need database access. Keeping MongoDB internal reduces attack surface and prevents arbitrary internet clients from attempting to connect.

### Q4. Why is `docker ps` showing `Up` not enough?

It proves that the container process is running, not that the application is ready, its dependencies are reachable, or user requests succeed.

### Q5. What is a health check?

A health check is an automated test that reports whether a container or application is operating as expected. A useful health design distinguishes process liveness from traffic readiness.

### Q6. What is a Docker volume, and is it a backup?

A volume stores data outside a container's writable layer and can preserve it across selected container events. It is not automatically a backup because it may still be lost through deletion, corruption, host failure, or an operational mistake.

### Q7. Why use a registry in production?

A registry centrally stores and distributes versioned container artifacts so staging and production can pull the exact image that was built, tested, and approved.

## Intermediate Level

### Q8. How should an API connect to MongoDB in Compose?

It should normally use the MongoDB service name and internal port, for example `mongodb://mongodb:27017/mern`. Inside the API container, `localhost` refers to the API container itself.

### Q9. What is the difference between liveness and readiness?

Liveness asks whether the process is alive enough to continue running. Readiness asks whether the instance can safely receive traffic, often including required dependency checks.

### Q10. Should liveness depend on MongoDB?

Not automatically. If liveness fails whenever MongoDB has a temporary issue, the platform may repeatedly restart an otherwise useful process. The desired failure behavior should determine whether a dependency belongs in liveness, readiness, or application-level error handling.

### Q11. Why use environment-specific configuration with the same image?

It supports build-once-deploy-many. The application artifact stays consistent while staging and production supply different endpoints, feature settings, and secrets at runtime.

### Q12. How would you troubleshoot a 502 from Nginx?

I would inspect Nginx logs, verify the upstream host and port, confirm Docker DNS and network membership, test connectivity to the API service, check the API health endpoint, inspect API logs, and compare the current image and configuration with the previous release.

### Q13. How would you troubleshoot an API-to-MongoDB failure?

I would inspect API and MongoDB logs, review the resolved Compose configuration, verify that both services share a network, confirm the URI uses `mongodb` rather than `localhost`, test service-name resolution, check authentication, and verify MongoDB readiness.

### Q14. What should be recorded for deployment traceability?

Record the image repository and tag, image digest, Git revision, configuration or release identifier, deployment time, health result, operator or automation identity, and previous rollback target.

### Q15. What is the difference between persistence and backup?

Persistence keeps data available across selected container lifecycle events. Backup provides a recoverable copy for destructive events, corruption, or accidental deletion. A persistent volume does not replace a backup strategy.

### Q16. Why should production normally use `image:` rather than `build:`?

Production should consume a tested registry artifact. Building on the production host mixes build and runtime responsibilities and may produce a different result from the image tested in staging.

## Advanced and Scenario Questions

### Q17. A deployment reports all containers as running, but users receive 500 errors. What do you do?

I preserve evidence, confirm the affected route and scope, inspect proxy and API logs, check readiness and dependency connectivity, review resource metrics and restarts, identify the current release, compare it to the previous version, and choose a targeted fix or rollback based on evidence.

### Q18. Why should you avoid restarting everything as the first incident response?

A restart can destroy useful evidence, increase load, create a restart loop, and hide the root cause. First isolate the failing layer and collect logs, health, network, resource, and release information.

### Q19. What is a rollback?

A rollback returns the application to a known-good previous artifact after a new release causes unacceptable impact. It should include health validation and consideration of configuration, background jobs, persistent data, and database schema compatibility.

### Q20. Why can a database migration make application rollback unsafe?

A newer application may change the schema in a way the older application cannot read. Destructive migrations can make the previous application version fail even when its image is available.

### Q21. What is expand-and-contract migration?

It is a compatibility-oriented migration pattern: add the new schema while retaining the old form, deploy code that supports both, migrate data, switch usage, and remove the old form only after older versions no longer need rollback support.

### Q22. What is graceful shutdown important for?

It lets the application stop accepting new work, finish or cancel existing work predictably, close connections, and avoid corrupting or abandoning requests during deployment or failure.

### Q23. What is a good production logging approach for Node?

Write structured, useful logs to stdout/stderr so Docker can collect them. Include safe fields such as service, request ID, route, status, duration, and message, while excluding passwords, tokens, and connection strings.

### Q24. What would your production deployment checklist include?

Known source revision, tests, scan, approved image and digest, staging validation, configuration verification, health and readiness checks, migration review, rollback artifact, deployment procedure, and monitoring plan.

### Q25. How would you define production readiness in one answer?

> Production readiness means the system is reproducible, appropriately secured, observable, health-checked, correctly networked, recoverable, and supported by documented procedures for deployment, failure handling, and rollback.

## Project Pitch Template

> I built a production-style Dockerized MERN application. Nginx is the public reverse proxy, while the Node API and MongoDB communicate over a private Docker network. MongoDB uses persistent storage, and the API exposes health and readiness information. I use environment-specific configuration instead of baking secrets into images, build and scan versioned images, and push approved artifacts to a registry. Production deploys the exact tested image, and I retain a known-good previous release for rollback. I also documented troubleshooting for Nginx upstream errors, health failures, and API-to-MongoDB connectivity problems.
