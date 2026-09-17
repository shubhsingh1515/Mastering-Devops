# DevOps Mentorship Program - Day 40

## Phase 2: Docker & Containers

### Docker Production Readiness & Final Container Assessment

**Level:** Intermediate -> Professional  
**Focus:** Combining Docker, Compose, networking, storage, Nginx, configuration, health checks, registries, security, deployment, troubleshooting, and rollback

Day 40 is the Docker phase integration lesson. The goal is no longer to memorize one more command. The goal is to review and operate a production-style Dockerized MERN system as a complete delivery and operations workflow.

The phase progression is:

```text
Docker fundamentals
      |
      v
Dockerfiles and image optimization
      |
      v
Compose, networking, and volumes
      |
      v
Nginx and configuration
      |
      v
Health checks, logging, and monitoring
      |
      v
Registries, deployment, and rollback
      |
      v
Image security
      |
      v
Production readiness assessment
```

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Design a production-style Docker architecture for a MERN application.
- Review a Dockerfile for reproducibility, security, and runtime problems.
- Review a Compose configuration for exposure, persistence, and reliability problems.
- Explain the image, registry, staging, and production delivery flow.
- Distinguish liveness from readiness.
- Troubleshoot a multi-layer incident without restarting everything blindly.
- Explain how logs, health checks, image metadata, and network state provide evidence.
- Identify how database schema changes affect application rollback.
- Define a practical deployment and rollback procedure.
- Identify remaining gaps before calling a Docker deployment production-ready.
- Answer common professional-level Docker interview questions.

---

## 2. Target Production MERN Architecture

The target runtime architecture is:

```text
                         INTERNET
                             |
                             v
                    +-----------------+
                    |      NGINX      |
                    |     :80/:443    |
                    +--------+--------+
                             |
                      Private network
                             |
                    +--------+--------+
                    |                 |
                    v                 v
                Frontend            API
                                        |
                                        v
                                    MongoDB
                                        |
                                        v
                                Persistent volume
```

The delivery system is separate from the runtime:

```text
Developer
   |
   v
Git
   |
   v
Docker build
   |
   v
Tests and security scan
   |
   v
Container registry
   |
   v
Staging
   |
   v
Health and smoke checks
   |
   v
Production
   |
   v
Monitoring and rollback
```

The public boundary should normally be Nginx or another reverse proxy. The API and MongoDB should communicate over private Docker networks. MongoDB should not be publicly published merely because the API needs it.

Typical service responsibilities are:

```text
Nginx
+ public HTTP/HTTPS entry point
+ TLS termination where appropriate
+ routing and request limits

Frontend
+ static assets or frontend server
+ internal access behind Nginx where appropriate

API
+ business logic
+ internal port such as 3000
+ health and readiness endpoints

MongoDB
+ internal database service
+ authentication
+ persistent data volume
```

---

## 3. Production Readiness Is a System

`docker compose up -d` only proves that Docker accepted the requested state. It does not prove that users can safely use the application.

Production readiness combines:

```text
Security
+ Reliability
+ Reproducibility
+ Observability
+ Recovery
+ Documentation
```

A useful review model is:

```text
Production readiness
       |
       +-- Security
       |   +-- non-root user
       |   +-- secret handling
       |   +-- private services
       |   +-- scanned images
       |
       +-- Reliability
       |   +-- health checks
       |   +-- restart policy
       |   +-- graceful shutdown
       |   +-- persistent data
       |
       +-- Operations
       |   +-- logs
       |   +-- metrics
       |   +-- runbooks
       |   +-- deployment and rollback
       |
       +-- Reproducibility
           +-- versioned image
           +-- registry artifact
           +-- recorded digest
           +-- environment configuration
```

A production review should ask not only whether the application starts, but also:

```text
Can we identify what is running?
Can we detect when it is unhealthy?
Can we recover its data?
Can we limit the impact of a compromise?
Can we explain what changed?
Can we roll back safely?
```

---

## 4. Dockerfile Production Review

Consider this simplified Dockerfile:

```dockerfile
FROM node:latest

WORKDIR /app

COPY . .

RUN npm install

ENV MONGO_PASSWORD=secret

EXPOSE 3000

CMD ["node", "server.js"]
```

A production review should identify several concerns.

### Mutable base image

`node:latest` can resolve to different content over time. Prefer a maintained, controlled version and pin by digest when the build process requires exact reproducibility.

### Uncontrolled build context

`COPY . .` may copy `.env`, `.git`, local `node_modules`, logs, test output, or other unnecessary files if `.dockerignore` is missing or incomplete.

### Non-reproducible dependency installation

When a suitable lock file exists, `npm ci` is normally preferable to `npm install` for clean, repeatable installation. The runtime stage should install only dependencies required at runtime.

### Secret in image configuration

`ENV MONGO_PASSWORD=secret` can expose the value through image metadata, inspection, logs, or downstream use. Secrets belong in an appropriate runtime secret mechanism.

### No explicit runtime user

If no user is selected, the process may run as root. Use a non-root user where the application permits it.

### No separation between build and runtime

If the application requires a build step, development dependencies and tools should remain in a builder stage rather than being shipped to production.

A better simplified direction is:

```dockerfile
FROM node:22.14.0-slim

WORKDIR /app
ENV NODE_ENV=production

COPY package*.json ./
RUN npm ci --omit=dev

COPY --chown=node:node . .
USER node

EXPOSE 3000
CMD ["node", "server.js"]
```

This is still only a starting point. The complete review must also cover `.dockerignore`, health checks, shutdown behavior, image scanning, configuration, logging, and the deployment platform.

---

## 5. Compose Production Review

This configuration creates unnecessary exposure:

```yaml
services:
  api:
    build: ./backend
    ports:
      - "3000:3000"

  mongodb:
    image: mongo
    ports:
      - "27017:27017"
```

Questions to ask:

```text
Does the public internet need direct access to port 3000?
Does the public internet need direct access to MongoDB?
Should production build source code on the host?
Where is MongoDB data persisted?
Are service names used for internal DNS?
Are health checks and restart behavior defined?
Are secrets injected without baking them into images?
```

A production-oriented design normally exposes only the reverse proxy to the host:

```yaml
services:
  nginx:
    ports:
      - "80:80"
      - "443:443"

  api:
    expose:
      - "3000"

  mongodb:
    expose:
      - "27017"
```

The exact Compose syntax and network layout depend on the project. The principle is that internal communication uses service names and private networks:

```text
nginx -> api:3000
api   -> mongodb:27017
```

not:

```text
api -> localhost:27017
```

In production, prefer a tested registry image reference over rebuilding source on the production host:

```yaml
services:
  api:
    image: registry.example.com/mern-api:v2.3.0
```

This supports build-once-deploy-many and makes the deployed artifact traceable.

---

## 6. Configuration and Secrets

Separate the application artifact from environment-specific configuration:

```text
Same image
   |
   +-- Staging configuration
   |
   +-- Production configuration
```

Do not build separate application images merely to change a database URL or environment name. The image should be portable; configuration should be supplied at deployment time.

Keep these outside the image:

```text
Database passwords
JWT signing keys
API tokens
Private certificates
Production connection strings
```

Use the appropriate mechanism for the environment:

```text
Orchestrator secrets
Cloud secret manager
Protected deployment variables
Mounted secret files with controlled permissions
```

Configuration review should verify:

```text
Required variables exist
Values target the intended environment
No production secrets are committed
No secret is in image history
The API uses the database service name
Defaults do not silently point to production
```

A configuration error can look like an application bug. Record the resolved configuration safely, but never print secret values in logs.

---

## 7. Health Checks: Liveness and Readiness

A useful API may expose:

```text
GET /health
GET /ready
```

### Liveness

Liveness answers:

> Is the process alive enough that restarting it might help?

A liveness check should usually be lightweight. It should not necessarily fail every time an external dependency has a temporary problem.

### Readiness

Readiness answers:

> Can this instance safely receive traffic now?

A readiness check may verify important dependencies such as MongoDB, depending on the desired operational behavior.

The distinction is:

```text
Liveness  -> process survival
Readiness -> traffic eligibility
```

Do not make liveness depend on every external system unless that restart behavior is intentional. Otherwise, a temporary database outage can cause a restart loop instead of allowing the API to remain alive, report controlled errors, and recover its dependency connection.

A health endpoint should be:

```text
Fast
Authenticated appropriately for the environment
Observable in logs and monitoring
Specific about status without leaking secrets
```

A running container is not enough evidence of application health:

```text
docker ps shows Up
        !=
API accepts valid requests and dependencies are ready
```

---

## 8. Failure Isolation and Restart Behavior

Suppose MongoDB becomes unavailable. A fragile design may behave like this:

```text
MongoDB failure
      |
      v
API crashes
      |
      v
Nginx has no usable upstream
      |
      v
Entire website is unavailable
```

A more resilient design aims for:

```text
MongoDB failure
      |
      v
API remains alive where possible
      |
      v
Database-dependent requests return controlled errors
      |
      v
Readiness reports the dependency problem
      |
      v
Operators investigate and recover MongoDB
```

Restart policies are useful, but they are not a root-cause fix. Repeatedly restarting an unhealthy service can hide evidence, amplify load, and create a restart loop.

Review:

```text
Restart policy
Startup order
Dependency readiness
Graceful shutdown
Connection retry behavior
Timeouts and circuit behavior
```

Resilience means the system fails in a controlled and diagnosable way, not merely that every container restarts.

---

## 9. Observability: Logs, Metrics, and Traces

During an incident, operators should be able to answer:

```text
What failed?
When did it fail?
Which version was running?
Which service is responsible?
How many users are affected?
What changed recently?
```

The three major observability categories are:

```text
Logs    -> detailed events
Metrics -> numerical health and resource signals
Traces  -> request path across services
```

At this stage, focus on logs, Docker state, health results, and resource metrics.

For Node applications, write logs to stdout and stderr so Docker can collect them:

```javascript
console.log(JSON.stringify({
  level: "info",
  service: "api",
  message: "API started"
}));
```

Structured logs are easier to search and correlate than arbitrary text. Include safe fields such as:

```text
level
service
requestId
route
statusCode
durationMs
message
```

Never include passwords, tokens, full connection strings, or other secrets.

---

## 10. Deployment Traceability

When production reports errors, the team should identify the exact release quickly:

```text
Current image:  mern-api:8f31c2a
Git commit:     8f31c2a
Deployment:     2026-09-17
Previous image: mern-api:71de992
Image digest:   sha256:...
```

Image versioning is not merely a naming convention. It enables:

```text
Incident correlation
Change investigation
Audit evidence
Deterministic rollback
Environment comparison
```

Record at deployment time:

```text
Image repository and tag
Image digest
Git revision
Configuration version or release ID
Deployment timestamp
Operator or automation identity
Health-check result
Rollback target
```

Avoid relying only on `latest`. A semantic version, commit identifier, or immutable digest makes the intended release clearer.

---

## 11. Rollback Readiness

Rollback should be designed before an incident:

```text
Current release:  v2.0.0
Previous release: v1.9.0
```

When the new release causes unacceptable impact:

```text
v2.0.0
   |
   v
Confirm impact and release correlation
   |
   v
Select known-good artifact
   |
   v
Deploy previous image
   |
   v
Run health and smoke checks
   |
   v
Monitor and document
```

A rollback is more than changing an image tag. Review:

```text
Application version
Database schema
Configuration shape
External API compatibility
Persistent data
Background jobs
Cache behavior
```

Keep the previous image and its deployment metadata long enough to support recovery and investigation.

---

## 12. Database Compatibility and Expand-and-Contract

Application rollback can be unsafe when a newer release changes the database schema incompatibly.

Example:

```text
v1 expects: users.name
v2 changes: users.name -> users.display_name
```

If v2 migrates the database destructively and you immediately run v1 again, v1 may fail because the field it expects no longer exists.

A safer migration pattern is expand-and-contract:

```text
1. Expand: add the new field or structure while retaining the old one.
2. Deploy: run an application version that can use both forms.
3. Migrate: backfill or transform data gradually.
4. Switch: move reads and writes to the new structure.
5. Contract: remove the old structure only after rollback is no longer required.
```

Before rollback, ask:

```text
Is the database schema backward-compatible?
Was a destructive migration executed?
Can the old application read current data?
Do background jobs understand both versions?
Is a database restore required instead?
```

Persistence keeps data available across selected container events. It is not the same as backup. A backup provides a recoverable copy for destructive failure, corruption, or accidental deletion.

---

## 13. Final Security Checklist

Before calling the deployment production-ready, verify:

```text
[ ] Maintained base image
[ ] Controlled version or digest
[ ] Minimal runtime image
[ ] Production dependencies only
[ ] Non-root user where practical
[ ] No secrets in image or history
[ ] Reviewed .dockerignore
[ ] Image scan completed
[ ] Only required ports exposed
[ ] MongoDB private
[ ] Secure registry access
[ ] Deployment identity uses least privilege
```

Security is one part of the complete readiness decision. A secure image can still fail because of bad configuration, missing health checks, lost data, or an undocumented rollback procedure.

---

## 14. Final Deployment Checklist

Before production:

```text
[ ] Image built from a known revision
[ ] Tests passed
[ ] Security scan completed
[ ] Approved image pushed to registry
[ ] Exact tag and digest recorded
[ ] Staging deployment validated
[ ] Production configuration verified
[ ] Health and readiness checks passing
[ ] Database migration reviewed
[ ] Previous version available
[ ] Rollback procedure documented
[ ] Runbook available
```

During deployment:

```text
Deploy
  |
  v
Health and readiness
  |
  v
Logs
  |
  v
Resource metrics
  |
  v
User impact
```

After deployment:

```text
Monitor
  |
  v
Confirm stability
  |
  v
Record release evidence
```

---

## 15. Production Incident Scenario

Deployment:

```text
mern-api:v2.3.0
Nginx: healthy
API: running
MongoDB: running
```

Users report:

```text
POST /api/orders -> 500
```

Do not begin with `docker compose restart`. First preserve evidence and isolate the failing layer:

```text
1. Confirm the affected route and scope.
2. Check the API logs.
3. Check Nginx logs and upstream errors.
4. Check API health and readiness.
5. Check database connectivity and database logs.
6. Check network and service-name resolution.
7. Check CPU, memory, disk, and container restarts.
8. Identify the recent image, commit, and configuration change.
9. Compare the release with the previous known-good version.
10. Decide whether remediation or rollback is appropriate.
11. Preserve evidence and document the timeline.
```

The goal is to identify the failed layer, not to create additional changes before the cause is understood.

---

## 16. Incident Drill: MongoDB Connectivity

Suppose API logs show:

```text
MongoServerSelectionError
```

Investigate:

```bash
docker compose logs api
docker compose logs mongodb
docker network ls
docker network inspect <network-name>
docker compose config
```

You discover:

```text
MONGO_URI=mongodb://localhost:27017/mern
```

Inside the API container, `localhost` refers to the API container itself. It does not refer to the MongoDB container.

Use the Compose service name on the shared Docker network:

```text
MONGO_URI=mongodb://mongodb:27017/mern
```

The root cause crosses several topics:

```text
Container isolation
+ Docker DNS
+ Compose configuration
+ Network membership
+ Application dependency handling
```

After changing configuration, validate the resolved Compose file, restart only the affected service where safe, run readiness checks, and verify a real application request.

---

## 17. Final Practical Assessment

Design a production deployment for:

```text
frontend/
backend/
nginx/
mongodb
```

Requirements:

```text
1. Nginx is publicly accessible.
2. The API is internal.
3. MongoDB is internal.
4. MongoDB data persists.
5. The API has health information.
6. Containers do not run unnecessarily as root.
7. Secrets are not baked into images.
8. Images are versioned.
9. Images are scanned.
10. Previous versions can be rolled back.
```

Create:

```text
docker-compose.prod.yml
Dockerfile
.dockerignore
DEPLOYMENT.md
RUNBOOK.md
SECURITY.md
```

The documentation should explain:

```text
Architecture
Networking
Configuration
Persistence
Health and readiness
Security
Deployment
Rollback
Troubleshooting
```

Use the assessment to demonstrate decisions, not just file creation. Explain why each public port, network, volume, health check, user, image reference, and recovery step exists.

---

## 18. Monthly Cumulative Docker Milestone

The project should now demonstrate:

```text
[ ] Dockerfiles
[ ] Compose
[ ] Networking
[ ] Volumes
[ ] Nginx
[ ] Environment configuration
[ ] Health checks
[ ] Logging
[ ] Image versioning
[ ] Registry workflow
[ ] Security practices
[ ] Deployment procedure
[ ] Rollback procedure
[ ] Troubleshooting documentation
```

The complete operating model is:

```text
Git
  |
  v
Build
  |
  v
Test
  |
  v
Scan
  |
  v
Registry
  |
  v
Staging
  |
  v
Health validation
  |
  v
Production
  |
  v
Monitor
  |
  v
Rollback if required
```

---

## 19. Interview Project Pitch

Practice this explanation:

> I built a production-style Dockerized MERN application. Nginx acts as the public reverse proxy, while the Node API and MongoDB communicate over a private Docker network. MongoDB uses persistent storage, and the API exposes health information for operational checks. I use environment-specific configuration instead of baking secrets into the image, build versioned images, scan them for vulnerabilities, and push them to a registry. Production deploys the exact tested artifact, and I retain a known-good previous image for rollback. I also documented troubleshooting procedures for Nginx 502 errors, health failures, and API-to-MongoDB connectivity problems.

This demonstrates understanding of delivery, security, runtime operations, and recovery rather than only Docker syntax.

---

## 20. Day 40 Takeaways

The Docker phase is now a software delivery and production operations system:

```text
                 SOFTWARE DELIVERY
                        |
                        v
                     Docker
                        |
        +---------------+---------------+
        v               v               v
     Security       Reliability      Operations
        |               |               |
        v               v               v
     Scanning        Health           Logging
     Non-root        Restart          Monitoring
     Secrets         Shutdown         Runbooks
     Network         Persistence      Rollback
        |               |               |
        +---------------+---------------+
                        v
                   Production
```

The most important principle is:

> Production readiness means the system is reproducible, appropriately secured, observable, health-checked, recoverable, and supported by documented operational procedures.

Do not only say, "I know Docker." Explain how Docker fits into a reliable software delivery and production operations system.

### Next phase

The next lesson begins the transition from core Docker into CI/CD. The purpose of CI/CD is to automate the build, test, scan, artifact, deployment, and verification flow you have developed across this Docker phase.
