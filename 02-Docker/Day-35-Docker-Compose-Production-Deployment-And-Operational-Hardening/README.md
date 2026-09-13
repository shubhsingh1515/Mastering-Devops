# DevOps Mentorship Program - Day 35

## Phase 2: Docker & Containers

### Docker Compose Production Deployment and Operational Hardening

**Level:** Intermediate to Professional  
**Focus:** Turning a Compose-based MERN application into an operationally reliable production stack.

> Day 35 builds on Day 32 reverse proxying, Day 33 configuration management, and Day 34 health checks, dependencies, restart policies, and graceful shutdown. CI/CD is intentionally not introduced yet.

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Design a production-oriented Docker Compose stack.
- Separate Compose deployment configuration from application code.
- Minimize unnecessary public exposure.
- Use service names and private Docker networks correctly.
- Explain persistence versus backup.
- Select restart policies intentionally.
- Combine health checks, graceful shutdown, and resource controls.
- Centralize container logs through standard output and error streams.
- Troubleshoot common Compose deployment failures systematically.
- Explain why basic Compose alone does not guarantee zero-downtime deployment.
- Describe operational hardening in a DevOps interview.

---

## 2. Why Production Compose Is Different

A development Compose file often exposes every service for convenience:

```yaml
services:
  frontend:
    ports:
      - "5173:5173"

  api:
    ports:
      - "3000:3000"

  mongodb:
    ports:
      - "27017:27017"
```

This is useful when a developer needs to connect directly to each service. It is usually too permissive for production.

Production asks a stricter question:

> Does this service really need to be reachable from outside the host?

A safer request path is:

```text
Internet
   |
   v
Nginx :80/:443
   |
   v
Node API :3000 on private network
   |
   v
MongoDB :27017 on private network
```

The usual exposure model is:

```text
Nginx     -> public entry point
API       -> internal
MongoDB   -> internal
```

This reduces attack surface, clarifies ownership, and makes the intended architecture easier to troubleshoot.

---

## 3. Target Production Architecture

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
              React static       Node API
              assets             api:3000
                                      |
                                      v
                                  MongoDB
                                  :27017
                                      |
                                      v
                                mongo-data
```

Depending on the application layout, Nginx can serve the React production build itself and proxy `/api` requests to Node. A separate frontend web-server container may be useful in some organizations, but it should not be added just because development used a separate frontend process.

A production Compose design should answer five questions:

```text
What runs?
How do services communicate?
What data persists?
Where does configuration come from?
What happens when a service fails?
```

For this MERN stack:

```text
Services       -> nginx, api, mongodb
Communication  -> private Docker network
Persistence    -> MongoDB named volume
Configuration  -> external environment/configuration mechanism
Recovery       -> health checks, restart policies, retries, monitoring
```

---

## 4. Production Compose Example

A simplified design might look like this:

```yaml
services:
  nginx:
    build: ./nginx
    ports:
      - "80:80"
    depends_on:
      api:
        condition: service_healthy
    networks:
      - mern-network
    restart: unless-stopped
    stop_grace_period: 15s

  api:
    build: ./backend
    environment:
      NODE_ENV: ${NODE_ENV:?NODE_ENV must be provided}
      MONGO_URI: ${MONGO_URI:?MONGO_URI must be provided}
      PORT: 3000
    networks:
      - mern-network
    depends_on:
      mongodb:
        condition: service_healthy
    restart: on-failure:5
    stop_grace_period: 30s
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s

  mongodb:
    image: mongo:8
    volumes:
      - mongo-data:/data/db
    networks:
      - mern-network
    restart: unless-stopped
    stop_grace_period: 30s
    healthcheck:
      test: ["CMD-SHELL", "mongosh --quiet --eval 'db.adminCommand({ ping: 1 }).ok' | grep 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

volumes:
  mongo-data:

networks:
  mern-network:
```

This is a design example, not a universal production file. Before using it, verify that `wget`, `mongosh`, and `grep` exist in the selected images and that the API listens on `0.0.0.0:3000`.

This example expresses useful intent:

- Nginx is the only public service.
- API and MongoDB use the internal network.
- MongoDB data survives container replacement.
- Health checks provide service signals.
- Nginx waits for a healthy API according to Compose dependency behavior.
- The API waits for a healthy MongoDB according to Compose dependency behavior.
- API restart attempts are bounded.
- Services receive time to shut down cleanly.

It does not replace backups, monitoring, secret management, host security, or a capable deployment controller.

---

## 5. Minimize Public Exposure

This mapping publishes a host port:

```yaml
api:
  ports:
    - "3000:3000"
```

If Nginx is the only service that needs the API, the mapping is usually unnecessary. Nginx can use the private address:

```text
http://api:3000
```

Similarly, avoid publishing MongoDB:

```yaml
mongodb:
  ports:
    - "27017:27017"
```

unless there is a documented, controlled operational requirement.

A strong production explanation is:

> I expose only the required public entry point and keep internal services on private Docker networks whenever possible.

Do not confuse a container's internal listening port with a public host port. The API can listen on `3000` internally without making `localhost:3000` accessible from outside the host.

---

## 6. Service Names and Docker Networking

If the Compose services are named `nginx`, `api`, and `mongodb`, Docker provides internal service discovery on the shared network:

```text
Nginx -> api:3000
API   -> mongodb:27017
```

Prefer these logical names over hard-coded addresses:

```text
api -> 172.18.0.4       wrong design
nginx -> 192.168.x.x    wrong design
```

Container IP addresses can change when a service is recreated. Service names remain the stable identity that Compose clients should use.

Inspect the network:

```bash
docker network ls
docker network inspect <project>_mern-network
```

Check DNS from a container when the diagnostic tool exists:

```bash
docker compose exec api getent hosts mongodb
docker compose exec nginx getent hosts api
```

If a service cannot resolve another service, check that both services are attached to the same network and that the Compose service name is correct.

---

## 7. Volumes and Production Data

MongoDB writes data under `/data/db`. A named volume maps that directory outside the replaceable container filesystem:

```text
MongoDB container
       |
       v
/data/db
       |
       v
mongo-data named volume
```

If the MongoDB container is replaced, the volume can be mounted into the new container:

```text
MongoDB container v1 -> removed
MongoDB container v2 -> starts with mongo-data
```

This is why stateful services need deliberate storage design.

A volume is persistence, but persistence is not the same as backup.

---

## 8. Persistence Is Not Backup

Suppose MongoDB uses a persistent volume and someone runs:

```javascript
db.dropDatabase()
```

The volume persists the new, deleted state. It does not know that the deletion was accidental.

The distinction is:

```text
Persistence -> data survives container replacement
Backup      -> recoverable copy protects against deletion or corruption
```

A production database strategy needs:

```text
Persistent storage
       +
Automated backups
       +
Restore testing
       +
Retention and recovery objectives
```

Do not claim that a Docker volume alone provides disaster recovery. A backup that has never been restored is an assumption, not verified protection.

---

## 9. Restart Policies and Restart Storms

Restart policies can improve recovery:

```yaml
restart: unless-stopped
```

is often considered for long-running infrastructure services.

```yaml
restart: on-failure:5
```

can be appropriate for an API when you want a bounded number of recovery attempts after failure.

A restart policy does not fix a broken configuration or application bug.

Example restart storm:

```text
MONGO_URI=wrong-value
       |
       v
API starts
       |
       v
MongoDB connection fails
       |
       v
API exits
       |
       v
Docker restarts API
       |
       v
Same failure repeats
```

The correct response is:

```text
Observe logs and state
       |
       v
Identify the root cause
       |
       v
Correct configuration or code
       |
       v
Restart or redeploy once
       |
       v
Verify health and request path
```

Repeatedly restarting a broken service only creates noise and may erase useful timing evidence.

---

## 10. Health Check Design

An API may expose:

```http
GET /health
```

with:

```json
{
  "status": "ok"
}
```

But first define what the endpoint means.

### Simple liveness check

```text
Is the Node process responsive?
```

This can be useful for deciding whether the process is alive enough to continue.

### Dependency-aware readiness check

```text
Is Node responsive?
Is MongoDB available?
Can required dependencies be used?
```

This is useful for deciding whether the service should receive traffic.

A health endpoint that always returns `200` even when MongoDB is unavailable is not a meaningful readiness signal. However, making liveness depend on every external system can cause unnecessary restarts during a temporary dependency outage.

Use the distinction deliberately:

```text
Liveness  -> Should this process remain alive?
Readiness -> Should this instance receive traffic?
```

A readiness endpoint can return `503 Service Unavailable` when the API is alive but not currently able to serve correctly. Never expose passwords, connection strings, stack traces, or other sensitive values through health responses.

---

## 11. Graceful Shutdown and `stop_grace_period`

During deployment or manual replacement, Docker sends a termination signal to the application. A Node API should:

```text
Receive SIGTERM
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
Close MongoDB connections
      |
      v
Exit cleanly
```

Compose can provide time for this process:

```yaml
api:
  stop_grace_period: 30s
```

The application must actually handle the signal; a grace period cannot make an application graceful by itself. If the application does not exit before the grace period ends, Docker may use stronger termination.

Graceful shutdown helps prevent:

- Dropped requests.
- Connection resets.
- Incomplete writes.
- Unclosed database connections.
- Misleading errors during replacement.

Test it rather than assuming it works:

```bash
docker stop <api-container>
docker logs <api-container>
```

Look for a shutdown message and evidence that resources closed in the expected order.

---

## 12. Deployment Lifecycle and Zero Downtime

Suppose the current version is:

```text
api:v1
```

and the new version is:

```text
api:v2
```

A simplistic replacement can create an outage:

```text
Stop v1
   |
   v
Gap with no API
   |
   v
Start v2
   |
   v
Requests fail during startup
```

A more production-oriented sequence is:

```text
Build v2
   |
   v
Start v2
   |
   v
Run health checks
   |
   v
Verify the real request path
   |
   v
Route traffic to v2
   |
   v
Gracefully stop v1
   |
   v
Monitor the result
```

Basic Compose on one host does not automatically provide Kubernetes-style rolling deployment guarantees. True zero-downtime deployment usually requires a deliberate strategy involving a reverse proxy, multiple instances, a service manager, an orchestrator, or a platform deployment feature.

A strong interview answer is:

> Compose is useful for defining and running multi-container applications and can support health and restart behavior. Compose alone on a single host does not automatically provide reliable zero-downtime rolling updates. That requires an appropriate deployment strategy or orchestration platform.

---

## 13. Operational Hardening Checklist

A production-oriented container stack should increasingly include:

```text
Minimal runtime images
Non-root application users
No secrets baked into images
Limited public ports
Private internal networks
Health checks
Intentional restart policies
Graceful shutdown
Persistent storage where required
Backups and restore testing
Structured logs
Resource boundaries
Versioned images
Configuration outside the image
```

These controls address different failure modes. A minimal image does not replace a health check. A health check does not replace a backup. A restart policy does not replace root-cause analysis.

### Non-root users

The application should not run as root unless there is a specific, justified requirement. Least privilege reduces the potential impact of a compromised process.

### Minimal images

Keep build tools and unnecessary packages out of the runtime image. Smaller images reduce attack surface and download time, but health-check tooling must still be supplied deliberately.

### Versioned images

Prefer explicit immutable tags or digests over an ambiguous moving tag:

```text
mern-api:7f81a2c
```

A versioned image makes rollback and incident investigation easier.

### Secrets

Use external secret/configuration mechanisms. Do not put production credentials in a Dockerfile, committed Compose file, image layer, or application log.

---

## 14. Resource Limits and Failure Containment

Suppose Node has a memory leak:

```text
Node memory increases
       |
       v
Host memory pressure
       |
       v
Other services are affected
```

Resource controls can contain the impact more effectively:

```text
Node process
       |
       v
Configured resource boundary
       |
       v
Failure is easier to detect and isolate
```

Direct Docker usage may apply limits such as:

```bash
docker run --memory=512m --cpus=1 mern-api:7f81a2c
```

Compose and deployment-platform resource settings vary by mode and version. Validate how the actual runtime enforces the setting rather than assuming that a configuration field has identical behavior everywhere.

Observe usage with:

```bash
docker stats --no-stream
```

Resource monitoring is part of troubleshooting. If logs do not explain a restart, check whether the process was killed because of memory or CPU pressure.

---

## 15. Logging Strategy

Containers should normally write application logs to standard output and standard error:

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

Then operators can use:

```bash
docker compose logs api
docker compose logs --tail 100 nginx
docker compose logs -f api
```

Avoid relying on a file such as `/app/logs/application.log` inside an ephemeral container unless persistent or centralized log handling has been designed deliberately. A file inside a replaceable container can disappear with the container.

Good production logs include:

- Timestamp.
- Service and operation.
- Severity.
- Error details.
- Request or correlation identifier where appropriate.

They should exclude:

- Passwords.
- Tokens.
- Full secret-bearing connection strings.
- Unnecessary personal data.

Later curriculum topics may connect this model to Loki, ELK, OpenTelemetry, and cloud logging.

---

## 16. Production Troubleshooting Workflow

When users report a failure, trace the request and the system layers in order:

```text
1. What is the user experiencing?
2. Which service handles the request?
3. Is the container running?
4. Is the application healthy?
5. Can services resolve each other?
6. Can services communicate?
7. Are configuration values correct?
8. Is a dependency healthy?
9. Are resources exhausted?
10. What is the root cause?
```

Core commands:

```bash
docker compose ps
docker compose logs --tail 100 api
docker compose logs --tail 100 nginx
docker compose config
docker network inspect <network>
docker inspect <container>
docker stats --no-stream
```

Do not change ports, delete volumes, or rebuild everything randomly. Make one targeted change after evidence identifies the likely cause.

---

## 17. Incident Scenario: Website Works, API Fails

Users report:

```text
The website loads, but API requests fail.
```

Compose shows:

```text
nginx      Up
api        Up
mongodb    Up
```

Do not conclude that everything is healthy. `Up` only describes container process state.

First inspect Nginx:

```bash
docker compose logs --tail 100 nginx
```

Suppose it reports:

```text
502 Bad Gateway
```

Then inspect the API:

```bash
docker compose logs --tail 100 api
```

Suppose it reports:

```text
MongoServerSelectionError
```

Trace the API-to-database path:

```bash
docker network inspect <network>
docker compose config
docker compose exec api getent hosts mongodb
docker compose logs --tail 100 mongodb
```

You discover:

```text
MONGO_URI=mongodb://localhost:27017/mern
```

Inside the API container, `localhost` points to the API container. The correct Compose service address is:

```text
MONGO_URI=mongodb://mongodb:27017/mern
```

This single incident combines reverse proxying, service discovery, configuration, health, logs, and dependency troubleshooting.

---

## 18. Practical Implementation Lab

Use the existing MERN project.

### Step 1: Validate Compose

```bash
docker compose config
```

Fix configuration errors before starting services.

### Step 2: Start the stack

```bash
docker compose up -d --build
```

### Step 3: Verify services

```bash
docker compose ps
```

Record container state and health state separately.

### Step 4: Verify the network

```bash
docker network ls
docker network inspect <your-network>
```

### Step 5: Verify the public API path

```bash
curl -f http://localhost/api/health
```

### Step 6: Verify API-to-MongoDB discovery

```bash
docker compose exec api getent hosts mongodb
```

Then test database connectivity using tools available in the image.

### Step 7: Check resource usage

```bash
docker stats --no-stream
```

### Step 8: Test graceful shutdown

```bash
docker stop <api-container>
docker logs <api-container>
```

Verify that the shutdown handler runs and resources close cleanly.

---

## 19. Failure Simulation Lab

These exercises are valuable because they test diagnosis rather than memorization.

### Failure 1: Wrong MongoDB hostname

Change:

```text
mongodb://mongodb:27017/mern
```

to:

```text
mongodb://wrong-host:27017/mern
```

Observe:

```bash
docker compose logs api
docker compose ps -a
```

Diagnose:

```bash
docker network inspect <network>
docker compose config
docker compose exec api getent hosts wrong-host
```

Restore the correct value and verify recovery.

### Failure 2: Wrong Nginx upstream

Temporarily change:

```nginx
proxy_pass http://api:3000;
```

to:

```nginx
proxy_pass http://wrong-api:3000;
```

Test:

```bash
curl http://localhost/api/health
```

Inspect Nginx logs and service DNS, then restore `api:3000`.

### Failure 3: Broken health check

Point the API health check at:

```text
/does-not-exist
```

Observe:

```bash
docker compose ps
docker inspect <api-container>
```

Restore `/health` and confirm that the health state recovers.

### Failure 4: Resource investigation

Run:

```bash
docker stats --no-stream
docker inspect <api-container>
```

Explain how resource pressure could cause a restart even when application logs are incomplete.

---

## 20. Production Readiness Checklist

```text
Docker Production Checklist
---------------------------
[ ] Dockerized services
[ ] Compose configuration
[ ] Internal Docker network
[ ] Persistent MongoDB volume
[ ] Nginx reverse proxy
[ ] Externalized configuration
[ ] Versioned images
[ ] Optimized Dockerfiles
[ ] Non-root application user
[ ] Health checks
[ ] Restart policies
[ ] Graceful shutdown
[ ] Standard output/error logs
[ ] Resource monitoring or limits
[ ] Limited public exposure
[ ] Automated backup strategy
[ ] Restore testing
[ ] Centralized monitoring
[ ] CI/CD
```

The final items are intentionally future work in this curriculum. A checklist is useful because production readiness is a collection of controls, not a single Docker command.

---

## 21. Interview Preparation

### Q1. Why should MongoDB generally not be publicly exposed?

Only trusted application components normally need access to it. Keeping it internal reduces attack surface and prevents direct database access from the public network.

### Q2. Why use Docker service names instead of container IP addresses?

Service names provide stable logical discovery. Container IP addresses can change whenever services are recreated.

### Q3. What is the difference between persistence and backup?

Persistence allows data to survive container replacement. Backup provides recoverable copies that protect against accidental deletion, corruption, and other destructive events.

### Q4. Why is `Up` not enough to determine application health?

A process can be running while the application cannot serve requests or communicate with required dependencies. Health checks and real request tests provide stronger evidence.

### Q5. How would you make a Compose MERN stack more production-ready?

I would use a public reverse proxy, private internal services, health checks, intentional restart policies, graceful shutdown, non-root users, minimal images, external configuration, secret management, persistent storage, backups, resource controls, structured logs, and versioned images.

### Q6. What happens if a restart policy repeatedly restarts a broken application?

It creates a restart loop or restart storm. I would inspect logs, exit state, configuration, dependencies, health checks, network connectivity, and resource usage to find the root cause.

### Q7. Can a Docker volume replace a backup strategy?

No. A volume provides persistence but does not protect against deletion, corruption, or destructive commands. Backups and restore testing are separate requirements.

### Q8. Can Docker Compose alone guarantee zero-downtime deployment?

No. Compose can define services and support health and restart behavior, but reliable rolling replacement generally requires a deliberate deployment strategy, multiple instances, traffic management, or an orchestration platform.

---

## 22. Review Questions From Earlier Lessons

### Networking

Why is this preferred inside the API container?

```text
mongodb://mongodb:27017
```

Because `mongodb` resolves to the MongoDB Compose service. `localhost` refers to the API container itself.

### Image optimization

Why does this order improve caching?

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

Dependency installation can remain cached when only application source changes.

### Security

Why should secrets not be placed in a Dockerfile?

They can become part of image layers and be exposed through inspection, storage, copying, or distribution.

### Reverse proxy

What does Nginx returning `502 Bad Gateway` generally mean?

Nginx could not obtain a valid response from its upstream. Investigate Nginx configuration, service discovery, network connectivity, backend state, and backend health.

### Logging

Which command follows API logs in real time?

```bash
docker compose logs -f api
```

---

## 23. Day 35 Summary

The central lesson is:

> Production Docker is about operating containers safely and predictably, not simply starting them.

The architecture is:

```text
Internet
   |
   v
Nginx
   |
   +--> Frontend static assets
   |
   +--> Private Node API
            |
            v
        Private MongoDB
            |
            v
        Persistent volume
```

Around that architecture, apply:

```text
Configuration
+ Security
+ Health
+ Restart behavior
+ Graceful shutdown
+ Logging
+ Resource controls
+ Persistence and backups
+ Troubleshooting
```

If asked how to make a Dockerized MERN application production-ready, explain both the architecture and the operational controls. That demonstrates production thinking rather than only Docker syntax.

Day 36 will continue the Docker production track with further practical hardening, followed by the final Docker assessment before the next curriculum phase.
