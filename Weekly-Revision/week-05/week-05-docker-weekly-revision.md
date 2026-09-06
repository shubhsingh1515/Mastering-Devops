# DevOps Mentorship Program - Week 5 Sunday Revision

## Docker Phase: Days 25-29

### Sunday Weekly Revision, Quiz & Practical Assessment

**Focus:** Consolidate Docker Compose, security hardening, registries, deployment workflow, logging, monitoring, and production troubleshooting. No new topic today.

---

## Progress Tracker

### Phase 1 - Linux

Complete.

### Phase 2 - Docker

- Day 21 - Docker Fundamentals
- Day 22 - Images, Layers & Build Optimization
- Day 23 - Docker Networking & Persistent Storage
- Day 24 - Weekly Revision
- Day 25 - Docker Compose
- Day 26 - Docker Security & Production Hardening
- Day 27 - Registry, Image Tagging & Deployment Workflow
- Day 28 - Logging, Monitoring & Resource Management
- Day 29 - Weekly Revision & Assessment

**Progress:** 7 of approximately 10 Docker topics completed.

---

## How to Use This Revision

Answer the questions before reading the explanations. Complete the practical assessment by writing an investigation plan first, then run commands against your own MERN project.

The purpose is to connect the topics into one operational workflow:

```text
Build
  -> Version
  -> Publish
  -> Run
  -> Observe
  -> Diagnose
  -> Recover
```

---

## 1. Quick Review With Solutions

### Q1. Why should production Docker images avoid `latest`?

`latest` is a mutable tag. It may point to different image content at different times, so the same deployment configuration can run different application versions.

Prefer an explicit release tag, commit-based tag, or digest:

```text
mern-api:1.4.2
mern-api:8f31c2a
registry.example.com/team/mern-api@sha256:<digest>
```

Versioned references improve reproducibility, auditing, and rollback.

### Q2. Why does `mongodb://localhost:27017` usually fail between containers?

Inside the Node container, `localhost` refers to the Node container itself. MongoDB is a separate service. On a shared Docker or Compose network, use the service name:

```text
mongodb://mongodb:27017/mern
```

Docker's internal DNS resolves `mongodb` to the correct service. Container IP addresses should not be hardcoded because they can change when the stack is recreated.

### Q3. What is the difference between an image and a container?

An image is a packaged, reusable application artifact containing the filesystem, runtime, dependencies, and startup metadata. A container is a running or stopped instance created from that image.

```text
Image     -> packaged artifact
Container -> runtime instance
```

Many containers can be created from the same image.

### Q4. What problem does Docker Compose solve?

Docker Compose defines and runs multi-container applications from a declarative configuration file. It coordinates services, networks, volumes, environment variables, health checks, dependencies, ports, and build settings.

For a MERN stack, Compose can describe the frontend, API, MongoDB, and supporting services as one repeatable application topology.

### Q5. Why is a Docker volume not a backup?

A volume provides persistence across normal container replacement. A backup is an independent copy and recovery process for deletion, corruption, host loss, or disaster.

A volume can still be deleted or damaged, so MongoDB requires tested backups, retention, and restore procedures.

---

## 2. Weekly Concept Summary

The production workflow learned during this Docker phase is:

```text
Developer
    |
    v
Git repository
    |
    v
Docker build
    |
    v
Versioned image
    |
    v
Container registry
    |
    v
Docker Compose or orchestrator
    |
    v
Nginx
    |
    v
Node API
    |
    v
MongoDB
    |
    v
Persistent volume and backups
```

### Production principles

- Build once and promote the same artifact.
- Prefer immutable image versions over mutable tags.
- Keep images small, current, and focused.
- Run applications as non-root where practical.
- Drop unnecessary Linux capabilities.
- Use a read-only filesystem where the application permits it.
- Expose only intentional public ports.
- Use service-name DNS for internal communication.
- Keep important data outside disposable containers.
- Inject configuration and secrets at runtime.
- Combine logs, metrics, health checks, and resource limits.
- Retain known-good images for rollback.
- Investigate root causes instead of repeatedly restarting services.

---

## 3. Weekly Knowledge Quiz

### 1. Which image reference is generally preferred for predictable production deployments?

A. The newest local image  
B. `latest`  
C. The container name  
D. A specific version tag or immutable digest

**Answer: D.** A version tag communicates the release, while a digest identifies exact image content.

### 2. What does a container registry provide?

A. A replacement for Dockerfiles  
B. Storage and distribution for container images  
C. Automatic database backups  
D. A process namespace only

**Answer: B.** A registry lets CI publish an artifact and deployment environments pull that same artifact.

### 3. Which hostname should a Node API use for a Compose service named `mongodb`?

A. `localhost`  
B. `127.0.0.1`  
C. `mongodb`  
D. A temporary container IP

**Answer: C.** Compose provides service-name DNS on the shared network.

### 4. What does `docker compose up -d` do?

A. Deletes all images  
B. Starts the Compose stack in detached mode  
C. Creates only a volume  
D. Scans an image

**Answer: B.** It creates and starts the configured services without occupying the terminal.

### 5. Which command shows live CPU and memory usage?

A. `docker logs`  
B. `docker stats`  
C. `docker system df`  
D. `docker inspect`

**Answer: B.** `docker stats` provides live container resource information.

### 6. Does a container shown as `Up` automatically prove that the API is healthy?

A. Yes  
B. No

**Answer: B.** `Up` indicates process state. Health checks and real requests verify application behavior.

### 7. What does a named MongoDB volume provide?

A. Public database access  
B. Persistent storage outside the disposable container layer  
C. Automatic off-site backup  
D. Automatic encryption

**Answer: B.** Persistence is not the same as backup.

### 8. What does `--cap-drop=ALL` do?

A. Removes all containers  
B. Removes Linux capabilities from the container  
C. Disables all networking  
D. Deletes volumes

**Answer: B.** It reduces the container's privilege surface; required capabilities may need to be added back deliberately.

### 9. What does `docker push` do?

A. Uploads an image to a registry  
B. Starts a container  
C. Creates a network  
D. Runs a health check

**Answer: A.** The image must first be tagged with its registry-qualified name.

### 10. What is the safest general rollback strategy?

A. Rebuild arbitrary old source code  
B. Redeploy the previous known-good image  
C. Delete the registry  
D. Change the database port

**Answer: B.** The known-good image was already built and tested, avoiding new build differences.

### 11. What does `docker system df` summarize?

A. Docker disk usage  
B. Docker DNS records  
C. CPU scheduling  
D. Image vulnerabilities

**Answer: A.** It reports usage from images, containers, volumes, and build cache.

### 12. What is the difference between readiness and liveness?

A. Readiness concerns serving traffic; liveness concerns whether the process should continue running.  
B. They are always identical.  
C. Readiness is image scanning.  
D. Liveness is database backup status.

**Answer: A.** A service may be alive but temporarily unable to serve useful traffic.

---

## 4. Practical Assessment: MERN API Failure

### Scenario

Your production MERN stack contains:

- Nginx
- Node.js API
- MongoDB

Users report:

> The website loads, but every API request returns an error.

You observe:

```bash
docker compose ps
```

```text
frontend    Up
api         Up
mongodb     Up
```

API logs show:

```text
MongoServerSelectionError
```

### Task 1: Explain why `docker compose ps` is not enough

`docker compose ps` reports container state, but does not prove that:

- the API can reach MongoDB
- MongoDB is ready to accept connections
- the API has the correct environment variables
- credentials are valid
- the API is responding to requests
- the health endpoint is passing

A container can be `Up` while the application is functionally unhealthy.

### Task 2: List the first five commands

A reasonable first five-command investigation is:

```bash
docker compose ps -a
docker compose logs --tail 100 api
docker compose logs --tail 100 mongodb
docker compose config
docker compose exec api printenv MONGO_URI
```

Then continue as evidence requires:

```bash
docker network ls
docker network inspect <project>_default
docker stats --no-stream
curl http://localhost:3000/health
```

### Task 3: Explain why `localhost` is incorrect

Inside the API container:

```text
localhost -> API container
```

MongoDB is another service. The correct connection is normally:

```text
mongodb://mongodb:27017/mern
```

The API does not need MongoDB's port published to the host for this internal connection.

### Task 4: Verify the Docker network

Inspect the resolved Compose configuration:

```bash
docker compose config
docker compose config --services
```

List networks:

```bash
docker network ls
```

Inspect the project network:

```bash
docker network inspect <project>_default
```

Confirm that the API and MongoDB containers are both attached. The exact generated name depends on the Compose project name.

### Task 5: Separate possible causes

#### Configuration

```bash
docker compose config
docker compose exec api printenv MONGO_URI
```

Check for `localhost`, a wrong database name, missing credentials, or an incorrect port.

#### Networking

Confirm both services share a network and that the API uses `mongodb` rather than a hardcoded IP.

#### MongoDB availability

```bash
docker compose ps
docker compose logs mongodb
```

Check initialization, authentication, readiness, storage, and health status.

#### Application code

If configuration, DNS, and MongoDB are healthy, inspect connection retry logic, driver configuration, request handling, and recent source changes.

### Structured diagnosis

```text
State
  |
  v
Logs
  |
  v
Configuration
  |
  v
Network and DNS
  |
  v
MongoDB readiness
  |
  v
API behavior and code
  |
  v
Resource pressure
```

---

## 5. Hands-On Lab

Use the MERN project from the earlier lessons.

### Start the stack

```bash
docker compose up -d
```

### Check service status

```bash
docker compose ps
```

### View API logs

```bash
docker compose logs --tail 50 api
```

### Follow API logs

```bash
docker compose logs -f api
```

Stop following with `Ctrl+C` without necessarily stopping the stack.

### Watch resources

```bash
docker stats
```

### Inspect disk usage

```bash
docker system df
docker system df -v
```

### Inspect the API container

```bash
docker inspect <api-container>
docker top <api-container>
```

### Verify the health endpoint

```bash
curl http://localhost:3000/health
```

### Goal

Build a repeatable troubleshooting workflow:

```text
Status -> Logs -> Health -> Configuration -> Network -> Resources -> Recovery
```

Record what each command tells you and what evidence would change your diagnosis.

---

## 6. Mini Assignment

Create a one-page document titled:

> Production Docker Deployment Checklist for a MERN Application

Include:

### Image build strategy

- dependency-cache-friendly Dockerfile
- `.dockerignore`
- production dependencies only
- maintained base image
- image scan before publishing

### Tagging and versioning

- semantic version or commit SHA
- immutable digest usage
- no blind production use of `latest`
- image-to-commit traceability

### Registry workflow

- build once
- test and scan
- push to a protected registry
- promote the same artifact
- least-privilege CI permissions
- retain rollback images

### Compose architecture

- frontend, API, and MongoDB services
- service-name DNS
- runtime configuration
- dependencies and health checks
- intentional public ports

### Network and storage

- Nginx as public entry point where appropriate
- private API-to-database traffic
- no unnecessary MongoDB exposure
- named MongoDB volume
- independent backup and restore process

### Security

- non-root user
- minimal image
- dropped capabilities
- read-only filesystem where practical
- explicit writable paths
- no baked secrets
- CPU and memory limits

### Operations

- stdout and stderr log collection
- health endpoint
- readiness and liveness considerations
- `docker stats`
- disk monitoring
- previous known-good version
- rollback verification

This document should be suitable for an interview portfolio.

---

## 7. Interview Preparation

### Beginner

**Question:** What is Docker Compose?

**Expected answer:** Docker Compose is a tool for defining and running multi-container applications from a declarative configuration file.

### Intermediate

**Question:** Why should production avoid the `latest` image tag?

**Expected answer:** It is mutable and can point to different image contents over time, making deployments, audits, and rollbacks less predictable.

### Advanced scenario

> Your Node.js container is healthy, but requests still fail. What do you check?

Use this structured approach:

```text
Check logs
      |
      v
Verify health endpoint
      |
      v
Verify environment variables
      |
      v
Verify Docker network and service DNS
      |
      v
Test database connectivity
      |
      v
Check CPU, memory, and disk
      |
      v
Inspect recent deployment changes
```

A healthy process can still have incorrect configuration, unavailable dependencies, failing business logic, or an invalid request path. Structured troubleshooting is more valuable than memorizing isolated commands.

---

## 8. Recommended Documentation

Continue reading the official documentation for:

- Docker Engine
- Docker Compose
- Dockerfile best practices
- container networking
- Docker volumes
- image tagging and registries
- Docker health checks
- Docker resource constraints

Focus on understanding the concepts practiced this week rather than memorizing every option.

---

## 9. Weekly Summary

This week you learned how to:

- build optimized Docker images
- understand image layers and caching
- use `.dockerignore`
- connect containers with Docker networks
- persist MongoDB data using volumes
- define multi-container applications with Docker Compose
- harden containers for production
- run applications as non-root
- version and publish images
- promote the same artifact through environments
- monitor containers using logs and metrics
- distinguish running from healthy
- investigate CPU, memory, disk, and dependency failures
- troubleshoot production Docker incidents

These are foundational skills expected of junior DevOps engineers.

---

## 10. Week 5 Completion Checklist

- [ ] Answered the quick-review questions.
- [ ] Completed the weekly knowledge quiz.
- [ ] Started and inspected the MERN Compose stack.
- [ ] Verified the API health endpoint.
- [ ] Read and followed service logs.
- [ ] Monitored CPU and memory with `docker stats`.
- [ ] Checked Docker disk usage.
- [ ] Verified that the API uses `mongodb`, not `localhost`.
- [ ] Confirmed API and MongoDB share a network.
- [ ] Confirmed MongoDB uses a persistent volume.
- [ ] Reviewed non-root and least-privilege controls.
- [ ] Documented versioned image deployment and rollback.
- [ ] Created the production Docker deployment checklist.

---

## Tomorrow: Day 30

Day 30 begins the Month 2 cumulative project assessment. You will combine the Docker skills from Days 21-29 into a production-style MERN deployment:

- build a complete Dockerized MERN stack
- apply security hardening
- configure networking and volumes
- use Docker Compose
- publish and version images
- practice deployment and rollback
- monitor the running stack
- complete a mock DevOps interview

By the end of Day 30, you will have a portfolio-ready Docker project that resembles a real production deployment.
