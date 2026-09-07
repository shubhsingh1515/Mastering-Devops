# Day 30 - Interview Questions: Cumulative MERN Docker Project

## Beginner Level

### Q1. What is the purpose of Docker Compose in this project?

Compose defines and runs the frontend, API, and MongoDB as one repeatable multi-container application. It also describes networks, volumes, environment configuration, dependencies, and health checks.

### Q2. How does the API connect to MongoDB?

It uses MongoDB's Compose service name on the shared network:

```text
mongodb://mongodb:27017/mern
```

It does not use `localhost` because `localhost` points to the API container itself.

### Q3. Why does MongoDB need a volume?

The volume keeps database files outside the disposable MongoDB container filesystem so data survives normal container recreation.

### Q4. What does `-p 3000:3000` mean?

It maps host port `3000` to container port `3000`. It does not mean that every service needs to publish its port to the host.

---

## Intermediate Level

### Q5. How would you optimize the backend image?

Use a suitable maintained base image, `.dockerignore`, copy dependency manifests before source, use `npm ci --omit=dev`, use multi-stage builds where appropriate, avoid unnecessary files, and run the final process as non-root.

### Q6. Why should MongoDB not normally be published to the host?

The API can reach it through the private Docker network. Publishing the database port unnecessarily increases exposure and attack surface.

### Q7. How would you test MongoDB persistence?

Create a test document, remove the MongoDB container, recreate the service, and confirm the document remains. Then explain that this proves persistence, not independent backup recovery.

### Q8. How do you secure secrets?

Do not put credentials in Dockerfiles, image layers, Git, or logs. Inject them at runtime through protected configuration or an appropriate secret-management system.

### Q9. What does a health check prove?

It reports whether a defined test passes. It does not prove that every business operation works or that a dependency will remain available indefinitely.

### Q10. Why should the API listen on `0.0.0.0` in a container?

Because the application must accept connections through the container network interface. Binding only to `127.0.0.1` can make it unreachable from outside the container.

---

## Advanced Level

### Q11. The frontend loads, but every API request fails. All containers show `Up`. What do you do?

I would inspect API and MongoDB logs, test the API health endpoint, inspect the resolved Compose configuration, verify `MONGO_URI`, confirm that both services share a network, resolve `mongodb` from the API container, check MongoDB readiness and credentials, inspect resource usage, and review recent deployment changes.

### Q12. What would you investigate for exit code 137?

Exit code 137 commonly indicates `SIGKILL`, possibly due to memory pressure. I would check logs, container memory limits, `docker stats`, host memory, kernel evidence where available, and application behavior before concluding that OOM caused the termination.

### Q13. How do you apply defense in depth to this project?

Use non-root users, minimal suitable images, `.dockerignore`, production dependencies only, dropped capabilities, read-only filesystems where practical, explicit writable paths, protected secrets, private database networking, resource limits, image scanning, health checks, and runtime monitoring.

### Q14. Explain build once and promote.

CI builds, tests, scans, and pushes one versioned image. Staging runs that exact image, and production promotes the same image or digest rather than rebuilding from source. This prevents environment-specific build differences.

### Q15. How would you roll back the deployment?

Deploy the previous known-good image tag or digest from the registry, verify health and traffic, and check database schema compatibility. Do not rebuild old source code unless there is a specific reason.

### Q16. What should the project README contain?

Architecture, startup instructions, ports, networks, volumes, environment variables, health checks, logs, monitoring, security choices, image and registry workflow, backup strategy, rollback, and troubleshooting steps.

### Q17. How would you improve a script that only checks whether containers are running?

Add health endpoint validation and a test of the API-to-MongoDB path. A running process is not proof that the application can serve useful traffic.

## System Design Scenario

> Design a production-style Dockerized MERN deployment for a small team.

A strong answer:

```text
Internet
    |
    v
Nginx or managed ingress
    |
    v
Private application network
    |
    +--> Frontend
    +--> Node API
             |
             v
          MongoDB
             |
             v
       Persistent storage
```

Then explain:

- only Nginx or the intended ingress is public
- the API reaches MongoDB with `mongodb:27017`
- MongoDB is not unnecessarily published
- data uses persistent storage and independent backups
- images are versioned and scanned
- services run with least privilege
- logs, health signals, resource metrics, and rollback are documented

## Short Strong Answers

**What is the most important Docker networking rule?**  
Inside a container, `localhost` means that container. Use service-name DNS for peer services.

**What is the most important deployment rule?**  
The artifact tested should be the artifact deployed.

**What is the most important data rule?**  
Containers are replaceable; important data needs durable storage and independent backups.

**What is the most important troubleshooting rule?**  
Do not treat a restart as a diagnosis. Collect logs, configuration, network, health, and resource evidence.
