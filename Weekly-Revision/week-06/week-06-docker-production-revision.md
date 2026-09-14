# DevOps Mentorship Program - Week 6 Sunday Revision

## Docker Phase: Days 31-35

### Day 36 - Weekly Docker Revision, Quiz, and Practical Assessment

**Type:** Sunday revision and practical assessment  
**Level:** Intermediate to Professional    
**Focus:** Consolidating Docker production architecture, networking, configuration, health, resilience, security, and troubleshooting.

> This revision connects the production Docker lessons from Days 31-35 into one realistic MERN incident and one operational workflow.

---

## Progress Tracker

### Phase 1 - Linux

Complete.

### Phase 2 - Docker

- Day 21 - Docker Fundamentals
- Day 22 - Images, Layers, and Build Optimization
- Day 23 - Docker Networking and Persistent Storage
- Day 24 - Weekly Revision
- Day 25 - Docker Compose Multi-Container MERN Application
- Day 26 - Docker Security and Production Hardening
- Day 27 - Docker Registry, Image Tagging, and Deployment Workflow
- Day 28 - Docker Logging, Monitoring, and Resource Management
- Day 29 - Weekly Revision and Assessment
- Day 30 - Monthly Cumulative MERN Docker Project
- Day 31 - Docker Best Practices and Production-Ready Containers
- Day 32 - Docker Production Deployment and Reverse Proxy
- Day 33 - Docker Production Configuration and Environment Management
- Day 34 - Docker Compose Resilience, Dependencies, and Deployment Patterns
- Day 35 - Docker Compose Production Deployment and Operational Hardening
- Day 36 - Weekly Revision and Practical Assessment

**Current focus:** Consolidate the production MERN deployment into an operational runbook and diagnose failures using evidence.

---

## How to Use This Revision

Complete the questions and practical tasks before reading the solutions. For the incident assessment, write your troubleshooting sequence first. Then run the commands against your own project and compare your reasoning with the model answer.

The weekly workflow is:

```text
Architecture
     |
     v
Network and service discovery
     |
     v
Configuration and secrets
     |
     v
Health and restart behavior
     |
     v
Logs and resource signals
     |
     v
Root-cause diagnosis
     |
     v
Targeted recovery
```

The most important operating rule is:

> When production fails, collect evidence and trace the request path before repeatedly restarting services.

---

## 1. Production MERN Mental Model

The production-oriented stack should look approximately like this:

```text
                         INTERNET
                             |
                             v
                    +----------------+
                    |     NGINX      |
                    |    :80 / :443  |
                    +-------+--------+
                            |
                      mern-network
                            |
                    +-------+--------+
                    |                |
                    v                v
              Frontend static       Node API
              assets                api:3000
                                      |
                                      v
                                  MongoDB
                                  :27017
                                      |
                                      v
                                Persistent volume
```

Around the architecture are the operational controls:

```text
Configuration and secrets
Health checks and readiness
Restart policies
Graceful shutdown
Structured logs
Resource monitoring
Image security
Persistent storage and backups
```

The architecture has clear boundaries:

```text
PUBLIC
  |
  +--> Nginx

INTERNAL
  |
  +--> API
  +--> MongoDB
```

Nginx is the normal public entry point. The API and MongoDB communicate through a private Docker network. MongoDB data is mounted on persistent storage rather than kept only inside the disposable container filesystem.

---

## 2. Revision: Docker Networking

If the API and MongoDB are separate Compose services, the API should use the MongoDB service name:

```text
mongodb://mongodb:27017/mern
```

This normally fails:

```text
mongodb://localhost:27017/mern
```

### Why?

Every container has its own network namespace. Inside the API container:

```text
localhost -> the API container itself
```

Docker's internal DNS resolves the Compose service name:

```text
mongodb -> the MongoDB container
```

The request path is:

```text
API container
      |
      | mongodb:27017
      v
MongoDB container
```

Do not hard-code a container IP. Container IPs may change when a service is recreated. Service names are the stable logical identity for Compose communication.

### Interview answer

> Each container has its own network namespace, so `localhost` refers to the current container. To reach another Compose service, use its service name and container port, such as `mongodb:27017`.

---

## 3. Revision: Host Ports and Container Ports

This mapping:

```yaml
ports:
  - "3000:3000"
```

means:

```text
HOST PORT 3000  ------>  CONTAINER PORT 3000
```

It makes the API reachable through the Docker host. It is not required for two services on the same Docker network to communicate.

Nginx can reach the API directly:

```text
nginx -> api:3000
```

The API does not need a public host mapping when Nginx is its only client.

### Production principle

> Do not publish a port merely because the process listens on that port. Publish it when an external host or network genuinely needs access.

A secure exposure model is:

```text
Public:
  Nginx :80/:443

Private:
  Nginx -> api:3000
  API   -> mongodb:27017
```

MongoDB should generally not publish `27017:27017` to the host or Internet.

---

## 4. Revision: Nginx Reverse Proxy and `502`

A normal request flow is:

```text
Browser
   |
   | GET /api/users
   v
Nginx
   |
   | proxy_pass http://api:3000
   v
Node/Express API
   |
   | mongodb:27017
   v
MongoDB
```

Nginx creates a controlled public boundary. It can serve frontend assets and proxy API requests without exposing the API or database directly.

When Nginx returns `502 Bad Gateway`, do not assume that Nginx itself is the root cause. Trace the entire upstream path:

```text
Nginx configuration
        |
        v
Docker network membership
        |
        v
DNS/service discovery
        |
        v
API container state
        |
        v
API listening port and interface
        |
        v
API health
        |
        v
API dependencies
```

Useful commands:

```bash
docker compose ps
docker compose logs --tail 100 nginx
docker compose logs --tail 100 api
docker network inspect <network>
docker compose exec nginx getent hosts api
docker compose exec nginx wget -qO- http://api:3000/health
```

The exact diagnostic command must exist in the selected image.

---

## 5. Revision: Configuration and Secrets

A production image should be independent of environment-specific values:

```text
Application image
       +
Environment configuration
       +
Secrets
       =
Running service
```

For example:

```text
mern-api:abc123
       |
       +--> Development configuration
       +--> Staging configuration
       +--> Production configuration
```

The image does not need to be rebuilt merely because `MONGO_URI`, `LOG_LEVEL`, or another runtime value changes.

### Backend runtime configuration

Node reads values during process execution:

```javascript
const mongoUri = process.env.MONGO_URI;
```

The same image can receive different values when started in different environments.

### Frontend build-time configuration

Typical React or Vite builds may embed variables during:

```bash
npm run build
```

The flow is:

```text
React environment value
       |
       v
Frontend build
       |
       v
Static JavaScript bundle
```

Changing the container environment afterward does not normally rewrite an already-built browser bundle. A runtime `config.js` pattern is required if one frontend artifact must be promoted across environments.

### Secret handling

Do not commit or bake production secrets into:

- `.env` files.
- Dockerfiles.
- Image layers.
- Committed Compose files.
- Logs.
- Health responses.
- Diagnostic screenshots.

Use `.env.example` for safe variable documentation and a protected configuration or secret-management system for production values.

---

## 6. Revision: Health, Readiness, and Restart Policies

A container being `Up` does not prove that the application is healthy:

```text
Container state != Application health
```

A health check may call:

```http
GET /health
```

and record:

```text
starting -> healthy -> unhealthy
```

### Liveness

Liveness asks:

> Is the process alive enough to continue?

### Readiness

Readiness asks:

> Should this service receive production traffic right now?

An API process may be alive while MongoDB is unavailable:

```text
Liveness:  PASS
Readiness: FAIL
```

This is more useful than restarting the process for every temporary dependency outage.

### Restart policies

Examples:

```yaml
restart: on-failure
restart: on-failure:5
restart: unless-stopped
```

Restart policies can provide recovery from certain process failures, but they are not root-cause fixes.

A restart loop looks like:

```text
API crashes
   |
   v
Docker restarts API
   |
   v
Same configuration fails
   |
   v
API crashes again
```

Investigate logs, exit state, configuration, dependencies, health checks, network, and resources instead of restarting repeatedly.

---

## 7. Revision: Graceful Shutdown

When Docker sends `SIGTERM`, a well-behaved Node application should:

```text
SIGTERM
   |
   v
Stop accepting new requests
   |
   v
Finish active requests where possible
   |
   v
Close HTTP server
   |
   v
Close database connections
   |
   v
Exit cleanly
```

Compose can provide a shutdown window:

```yaml
api:
  stop_grace_period: 30s
```

The application must actually handle the signal. A grace period only provides time; it does not implement cleanup automatically.

Without graceful shutdown, deployments may cause dropped requests, connection resets, incomplete operations, and unclosed resources.

---

## 8. Revision: Persistence, Backups, and Security

MongoDB data should be mounted at a persistent volume:

```text
MongoDB container
       |
       v
/data/db
       |
       v
mongo-data volume
```

Remember:

```text
Container lifecycle != Data lifecycle
Volume != Backup
```

A volume can survive container replacement, but it does not protect against accidental deletion, corruption, host failure, or disaster. Production requires backups, retention, and restore testing.

### Security checklist

```text
[ ] Non-root application user
[ ] Minimal runtime image
[ ] No secrets in image layers
[ ] No secrets committed to Git
[ ] Limited public ports
[ ] Private internal networks
[ ] Health checks
[ ] Intentional restart policies
[ ] Resource boundaries
[ ] Image scanning
[ ] Versioned image tags or digests
```

The goal is not to claim perfect security. The goal is to reduce unnecessary exposure and follow least privilege.

---

## 9. Revision: Logging and Resource Monitoring

Containers should generally write application logs to standard output and standard error:

```text
Node application
      |
      +--> stdout
      +--> stderr
             |
             v
       Container runtime
             |
             v
       Centralized logging
```

Use:

```bash
docker compose logs api
docker compose logs --tail 100 nginx
docker compose logs -f api
```

Avoid relying on files inside an ephemeral container unless persistent or centralized handling is deliberate.

Resource commands:

```bash
docker stats --no-stream
docker system df
```

A memory leak may cause host pressure or an out-of-memory termination. If application logs do not explain a restart, inspect resource usage and container state.

---

## 10. Practical Assessment: Production MERN Incident

You are the DevOps engineer on call.

The current state is:

```text
Nginx       Up
API         Restarting
MongoDB     Up
```

Users report:

```text
The website loads, but API requests fail.
```

Nginx logs show:

```text
502 Bad Gateway
```

API logs show:

```text
MongoServerSelectionError
```

### Your task

Before reading the model answer, write your troubleshooting sequence. Your reasoning should follow the request path:

```text
User
  |
  v
Nginx
  |
  v
API
  |
  v
MongoDB
```

### Step 1: Check service state

```bash
docker compose ps
docker compose ps -a
```

Determine whether the API is restarting, exited, unhealthy, or merely running.

### Step 2: Read API logs

```bash
docker compose logs --tail 100 api
```

Look for connection failures, missing configuration, authentication errors, port errors, and process termination messages.

### Step 3: Read MongoDB logs

```bash
docker compose logs --tail 100 mongodb
```

Check whether MongoDB completed startup, reports authentication problems, or is restarting itself.

### Step 4: Inspect the network

```bash
docker network inspect <network>
```

Confirm that API and MongoDB share the expected network.

### Step 5: Inspect resolved configuration

```bash
docker compose config
```

Redact secrets before sharing output.

### Step 6: Verify `MONGO_URI`

Check whether it contains:

```text
mongodb://localhost:27017/mern
```

or:

```text
mongodb://mongodb:27017/mern
```

### Root cause

Suppose the resolved configuration contains:

```text
MONGO_URI=mongodb://localhost:27017/mern
```

Inside the API container:

```text
API container
      |
      +--> localhost
              |
              v
          API container itself
```

The API is not reaching MongoDB. The correct service address is:

```text
mongodb://mongodb:27017/mern
```

After correcting the configuration:

```bash
docker compose up -d api
docker compose ps
docker compose logs --tail 100 api
curl -f http://localhost/api/health
```

Only restart or redeploy after the root cause is understood and corrected.

---

## 11. Practical Interview Assessment

### Question

> Your Node API and MongoDB run in Docker Compose, but the API cannot connect to MongoDB. Walk me through your troubleshooting process.

### Strong answer

> First I would inspect the API logs and container state. Then I would verify that the API and MongoDB share the expected Docker network. I would inspect the resolved Compose configuration and confirm that `MONGO_URI` uses the MongoDB service name rather than `localhost`. I would verify DNS resolution from inside the API container and test connectivity to MongoDB. I would also inspect MongoDB logs and health state. Only after identifying the root cause would I restart or redeploy the affected service.

This answer demonstrates:

```text
Evidence first
Service boundaries
Network awareness
Configuration awareness
Dependency testing
Targeted recovery
```

---

## 12. Weekly Knowledge Quiz

### Q1. Inside a Node container, MongoDB is a Compose service named `mongodb`. Which URI is normally correct?

A. `mongodb://localhost:27017/mern`  
B. `mongodb://mongodb:27017/mern`  
C. `mongodb://127.0.0.1:27017/mern`  
D. `mongodb://host:27017/mern`

### Q2. Nginx returns `502 Bad Gateway`. What should you investigate?

A. Only Nginx  
B. Only MongoDB  
C. The full Nginx-to-network-to-API request path  
D. Delete all containers

### Q3. What does this mean?

```yaml
ports:
  - "3000:3000"
```

A. Host port `3000` maps to container port `3000`  
B. MongoDB receives port `3000`  
C. Two containers are created  
D. Nginx automatically uses port `3000`

### Q4. Which service should generally not be publicly exposed in the basic MERN architecture?

A. Nginx  
B. MongoDB  
C. HTTPS endpoint  
D. Public web server

### Q5. What does `depends_on` not automatically guarantee?

A. A declared dependency relationship  
B. Startup ordering behavior  
C. Application readiness  
D. Compose awareness of the dependency

### Q6. Which command shows resolved Compose configuration?

A. `docker stats`  
B. `docker compose config`  
C. `docker history`  
D. `docker volume rm`

### Q7. What does a Docker volume provide?

A. Automatic disaster recovery  
B. Persistent storage  
C. Automatic encryption  
D. Automatic database backups

### Q8. Why should a Node application handle `SIGTERM`?

A. For graceful shutdown  
B. To increase CPU  
C. To create Docker images  
D. To expose MongoDB

### Q9. Which image reference is more production-friendly?

A. `mern-api:latest` only  
B. A specific version, commit tag, or immutable digest  
C. A random image name  
D. An untagged image

### Q10. A container is `Up`, but its `/health` endpoint fails. What does this demonstrate?

A. Docker is necessarily broken  
B. Container state and application health are different concepts  
C. MongoDB has been deleted  
D. The image cannot run

### Answer key

```text
1 -> B
2 -> C
3 -> A
4 -> B
5 -> C
6 -> B
7 -> B
8 -> A
9 -> B
10 -> B
```

### Score

| Score | Level |
|---|---|
| 9-10 | Excellent: ready for the practical assessment |
| 7-8 | Good: review the weaker production concepts |
| 5-6 | Targeted revision needed |
| 0-4 | Revisit Docker production fundamentals |

---

## 13. Coding Exercise: Docker Diagnostic Script

Create:

```text
docker-diagnose.sh
```

The script should:

1. Display Compose service status.
2. Show recent API logs.
3. Show recent Nginx logs.
4. Show Docker resource usage.
5. Show network information.
6. Test the public API health endpoint.
7. Return a non-zero exit code if the health request fails.

Useful commands:

```bash
docker compose ps
docker compose logs --tail 50 api
docker compose logs --tail 50 nginx
docker stats --no-stream
docker network inspect <network>
curl -f http://localhost/api/health
```

A PowerShell equivalent is acceptable on Windows. Keep the script safe: do not print all environment variables or secret values.

A successful result might look like:

```text
=== DOCKER STACK ===
nginx       OK
api         OK
mongodb     OK

=== PUBLIC HEALTH ===
API health request succeeded

STACK HEALTHY
```

A failed result should return a non-zero status and identify the evidence operators should inspect.

---

## 14. Hands-On Practical Assessment

Use your existing MERN project and perform this sequence without opening the previous daily lessons.

### Challenge A: Normal operation

```bash
docker compose up -d --build
docker compose ps
curl -f http://localhost/api/health
```

Record service state and health state separately.

### Challenge B: Network troubleshooting

Temporarily change:

```text
mongodb://mongodb:27017/mern
```

to:

```text
mongodb://localhost:27017/mern
```

Restart only the API, observe the logs, and diagnose the problem without immediately restarting the entire stack.

Restore the correct service name.

### Challenge C: Nginx troubleshooting

Change:

```nginx
proxy_pass http://api:3000;
```

to a wrong service name. Observe the resulting `502`, inspect the Nginx logs, test service discovery, and restore the correct upstream.

### Challenge D: Configuration troubleshooting

Run:

```bash
docker compose config
```

Verify that environment variables resolve as expected. Redact secrets in saved evidence.

### Challenge E: Health troubleshooting

Break the `/health` endpoint or health-check command. Observe the difference between:

```text
running
```

and:

```text
healthy
```

Restore the health check and confirm recovery.

---

## 15. Monthly Cumulative Project Checkpoint

### Project: Production-Ready MERN Docker Deployment

The target architecture is:

```text
Internet
   |
   v
 Nginx
   |
   +--> React static assets
   |
   +--> Node API
          |
          v
       MongoDB
          |
          v
       Persistent volume
```

### Required capabilities

```text
[ ] Dockerized frontend
[ ] Dockerized backend
[ ] MongoDB container or appropriate managed database
[ ] Compose configuration
[ ] Internal Docker network
[ ] Persistent MongoDB storage
[ ] Nginx reverse proxy
[ ] Health endpoint
[ ] Health check
[ ] Externalized configuration
[ ] No secrets committed
[ ] Non-root application
[ ] Optimized images
[ ] Versioned image tags
[ ] Logs available
[ ] Resource monitoring
[ ] Troubleshooting documentation
```

### New monthly requirement: production runbook

Create:

```text
RUNBOOK.md
```

It must answer:

1. How do I start the application?
2. How do I stop it?
3. How do I check service health?
4. Where do I find logs?
5. How do I check resources?
6. How do I diagnose an Nginx `502`?
7. How do I diagnose an API-to-MongoDB failure?
8. Where is persistent data stored?
9. How is configuration supplied?
10. How would I roll back an application image?

A good runbook is concise enough to use during an incident and detailed enough that another engineer can follow it without guessing.

---

## 16. Monthly Project Interview Challenge

### Question

> Show me a DevOps project you have built.

Do not show only:

```text
docker-compose.yml
```

Explain the complete operational design:

```text
Architecture
    |
    v
Networking
    |
    v
Persistence
    |
    v
Security
    |
    v
Configuration
    |
    v
Health
    |
    v
Logging
    |
    v
Troubleshooting
```

### Model answer

> I containerized a MERN application using separate application services and MongoDB. Nginx is the public boundary, while the API and database communicate over a private Docker network. MongoDB uses persistent storage. The backend receives environment-specific configuration at runtime, while the frontend is built as static assets. I added health checks, graceful shutdown handling, non-root execution, and versioned image references. I also documented troubleshooting procedures for Nginx `502` errors and API-to-MongoDB connectivity failures.

This explanation demonstrates architecture, security, operations, and troubleshooting rather than only Docker commands.

---

## 17. What You Should Know Without Notes

### Why does `localhost` not work between containers?

```text
localhost = the current container
```

### How does Nginx find the API?

```text
api:3000
```

through Docker service discovery.

### Why keep MongoDB private?

To reduce unnecessary attack surface and prevent direct public database access.

### Why use volumes?

To persist data across container replacement.

### Is a volume a backup?

No. Backups are separate recoverable copies with retention and restore testing.

### Why use health checks?

To verify a defined service or application condition instead of relying only on container state.

### Why use restart policies?

To provide automatic recovery from certain process failures.

### Do restart policies fix application bugs?

No. They can create a restart loop if the root cause remains.

### Why handle `SIGTERM`?

To stop accepting work, finish active requests where possible, close resources, and exit cleanly.

### Why use immutable or versioned image references?

For traceability, reproducibility, safer promotion, and rollback.

### Why externalize configuration?

So the same application artifact can run across environments with different values.

---

## 18. Self-Assessment

Rate yourself honestly:

| Skill | Strong | Review | Weak |
|---|---:|---:|---:|
| Docker networking | [ ] | [ ] | [ ] |
| Compose | [ ] | [ ] | [ ] |
| Volumes and persistence | [ ] | [ ] | [ ] |
| Image optimization | [ ] | [ ] | [ ] |
| Container security | [ ] | [ ] | [ ] |
| Registries and image tags | [ ] | [ ] | [ ] |
| Nginx and reverse proxy | [ ] | [ ] | [ ] |
| Environment configuration | [ ] | [ ] | [ ] |
| Health checks | [ ] | [ ] | [ ] |
| Troubleshooting | [ ] | [ ] | [ ] |

Several review or weak areas are expected. Use the practical lab to convert those areas into observable skills.

---

## 19. Day 36 Summary

This week's main lesson is that production Docker engineering is more than:

```bash
docker build
docker run
```

It is:

```text
Production Docker
       |
       +--> Networking and service discovery
       |
       +--> Security and least privilege
       |
       +--> Persistence and backups
       |
       +--> Nginx and request routing
       |
       +--> Configuration and secrets
       |
       +--> Health and readiness
       |
       +--> Logs and resource signals
       |
       +--> Restart behavior
       |
       +--> Graceful shutdown
       |
       +--> Troubleshooting and recovery
```

The most important interview principle is:

> Do not start by restarting things. Start by tracing the failure and collecting evidence.

The next lesson continues the Docker and Containers phase toward the final Docker assessment and the next curriculum phase.
