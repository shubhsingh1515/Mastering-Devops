# DevOps Mentorship Program - Day 34

## Phase 2: Docker & Containers

### Production Docker Compose: Resilience, Dependencies, and Deployment Patterns

**Level:** Intermediate to Professional  
**Focus:** Making a Compose-based MERN stack reliable during startup, failure, recovery, and deployment.

> Day 33 separated configuration from the application image. Day 34 adds operational resilience: the stack must not only start, but must also expose useful health signals, handle temporary failures, and shut down cleanly.

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain why container startup order is not the same as application readiness.
- Describe exactly what `depends_on` does and does not guarantee.
- Configure and interpret Docker health checks.
- Choose restart policies intentionally.
- Design resilient startup for a MERN application and MongoDB.
- Explain liveness and readiness checks.
- Recognize and troubleshoot restart loops.
- Connect `SIGTERM` and graceful shutdown to deployment reliability.
- Diagnose unhealthy services with Docker Compose commands.
- Explain production Compose design in an interview.

---

## 2. The Problem With "Container Is Running"

When you run:

```bash
docker compose up -d
```

Compose may show:

```text
mongodb   Up
api       Up
nginx     Up
```

That is useful information, but it does not prove that the complete application is ready.

MongoDB may still be initializing its database files while the API immediately attempts a connection:

```text
MongoDB process starts
        |
        v
MongoDB is still initializing
        |
        v
API starts and connects immediately
        |
        v
Connection refused or timeout
```

The result may be an API crash, a restart loop, failed health checks, or Nginx returning `502 Bad Gateway`.

The fundamental distinction is:

```text
Container started != Application ready
Application ready != Every dependency healthy
```

A container is a process boundary. Readiness is an application behavior. Dependency availability is a separate operational condition.

---

## 3. Startup, Readiness, and Health Are Different

A useful model is:

```text
Container state
      |
      v
Process state
      |
      v
Application state
      |
      v
Dependency state
```

### Container state

Docker knows whether the container process is running, stopped, or restarting.

### Process state

The main process may still be alive, but it might be blocked, overloaded, or unable to serve requests.

### Application state

The API may be listening on port `3000`, but its routes may not work correctly.

### Dependency state

The API may be healthy by itself but unable to use MongoDB, an external API, a queue, or another required service.

A reliable deployment observes the relevant layers instead of treating `Up` as proof that all layers work.

---

## 4. What `depends_on` Actually Does

A basic Compose dependency looks like this:

```yaml
services:
  api:
    depends_on:
      - mongodb

  mongodb:
    image: mongo:8
```

At a high level, this expresses a service startup relationship:

```text
depends_on
    |
    v
Start mongodb before api
```

It does not automatically mean:

```text
MongoDB accepts connections
MongoDB has finished initialization
MongoDB is healthy
The API can complete a real database operation
```

This is why the following assumption is unsafe:

> `depends_on` means the dependency is ready.

A service can be started before the dependency is ready. Even a dependency marked healthy only proves that the configured health test passed at that moment; the API still needs timeout and retry behavior for later failures.

### Compose health-aware dependency

Compose implementations can use a health condition:

```yaml
services:
  mongodb:
    image: mongo:8
    healthcheck:
      test: ["CMD-SHELL", "mongosh --quiet --eval 'db.adminCommand({ ping: 1 }).ok' | grep 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

  api:
    build: ./backend
    depends_on:
      mongodb:
        condition: service_healthy
```

This can improve startup ordering, but it is not a complete resilience strategy. The API should still retry temporary database failures and expose its own health state.

Always confirm that the health-check command exists in the selected MongoDB image and works with that image version.

---

## 5. What Is a Docker Health Check?

A health check is a command that Docker runs inside a container to test a defined condition:

```text
Container
    |
    v
Health-check command
    |
    +--> Exit 0: check passed
    |
    +--> Non-zero exit: check failed
```

Docker records health states such as:

```text
starting
healthy
unhealthy
```

A health check should test something meaningful for the service. It should be quick, deterministic, and safe to run repeatedly.

### API health endpoint

A Node API might expose:

```http
GET /health
```

with a response such as:

```json
{
  "status": "ok"
}
```

A simple Express route could be:

```javascript
app.get("/health", (_request, response) => {
  response.status(200).json({ status: "ok" });
});
```

This is a basic process and HTTP health check. It does not necessarily prove MongoDB is usable.

### API health check in Compose

```yaml
services:
  api:
    build: ./backend
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
```

The exact test must match the tools installed in the image. Minimal Alpine or distroless images may not include `curl`, `wget`, a shell, or a package manager.

### Why a health check can fail even when the API works

If the image does not contain `curl`, this check fails because Docker cannot execute the command:

```yaml
test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
```

That is a health-check configuration error, not necessarily an application error. Choose a tool that exists in the image, install the intended diagnostic tool deliberately, or use a purpose-built health-check approach.

---

## 6. Health-Check Parameters

Example:

```yaml
healthcheck:
  test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
  interval: 30s
  timeout: 5s
  retries: 3
  start_period: 10s
```

### `test`

The command Docker runs. With `CMD`, arguments are executed directly. With `CMD-SHELL`, the command is interpreted by a shell inside the image.

### `interval`

How often Docker runs the check after the service starts checking.

```text
interval: 30s -> run approximately every 30 seconds
```

### `timeout`

The maximum time allowed for one check. A slow or blocked check should fail rather than hang forever.

### `retries`

How many consecutive failures are tolerated before Docker marks the container `unhealthy` after it has entered normal checking.

### `start_period`

A startup grace period. This gives the application time to initialize before ordinary failures count against its health state.

The parameters form this flow:

```text
Container starts
      |
      v
Startup grace period
      |
      v
Run checks at interval
      |
  +---+---+
  |       |
 PASS    FAIL
  |       |
  v       v
Healthy  Retry
```

A health check should not be so strict that normal startup becomes failure, and it should not be so shallow that a broken service is reported healthy.

---

## 7. Liveness Versus Readiness

These concepts become especially important in Kubernetes, but they are useful in Compose-based systems too.

### Liveness

Liveness asks:

> Is the application process alive enough to continue?

A liveness check may verify that the process can respond to a simple endpoint. It should avoid declaring a temporary dependency outage as a reason to restart a process unnecessarily.

### Readiness

Readiness asks:

> Can this application currently receive production traffic?

A readiness check may verify that required dependencies are available and that the application can perform the work it promises to perform.

Example state:

```text
API process: running
MongoDB: unavailable

Liveness:  PASS
Readiness: FAIL
```

This is more useful than a single `Up` value. A live but not ready API should be removed from traffic while it recovers, rather than receiving requests that are guaranteed to fail.

### Choosing endpoint behavior

You may use separate endpoints:

```text
/live  -> process is responsive
/ready -> required dependencies are usable
```

A simple `/health` endpoint is acceptable for a small lab, but production systems should decide explicitly whether it is checking process liveness, dependency readiness, or both.

Do not put expensive operations, destructive writes, or secret values into a health endpoint.

---

## 8. MongoDB Health Checks

A MongoDB health check should use a command supported by the image and authentication setup. A common pattern is:

```yaml
healthcheck:
  test: ["CMD-SHELL", "mongosh --quiet --eval 'db.adminCommand({ ping: 1 }).ok' | grep 1"]
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 20s
```

The exact command may need adjustment when:

- The image uses `mongo` instead of `mongosh`.
- Authentication is enabled.
- The selected image version changes available tools.
- The container runs under a restricted user.
- The command output differs from the expected value.

A health check should be verified directly:

```bash
docker compose exec mongodb mongosh --quiet --eval "db.adminCommand({ ping: 1 })"
```

If the command is unavailable, inspect the image documentation and test an image-appropriate command. Do not copy a health check blindly from an unrelated MongoDB version.

---

## 9. Restart Policies

Restart policies control what Docker should do after a container exits. They are useful for resilience, but they do not fix the cause of a crash.

### `on-failure`

```yaml
restart: on-failure
```

Restart when the container exits with a non-zero status. This is useful for unexpected application failures, but behavior around manually stopped containers and retry limits should be understood for the Compose and Docker versions in use.

A maximum retry count can be used:

```yaml
restart: on-failure:5
```

This limits repeated restarts and can make a persistent configuration failure easier to notice.

### `unless-stopped`

```yaml
restart: unless-stopped
```

Restart the service after failures or Docker daemon restarts unless an operator explicitly stopped it. This is commonly considered for long-running infrastructure services, but it should still be paired with monitoring and useful logs.

### Other policies

```yaml
restart: "no"
```

Do not automatically restart. This is often the default and can be useful for one-shot jobs or deliberate debugging.

```yaml
restart: always
```

Always restart according to Docker's restart behavior, including after daemon restarts. Use intentionally and understand how manual stopping is handled.

### The key warning

```text
Crash -> restart -> crash -> restart
```

A restart policy provides recovery from some process failures. It does not diagnose invalid configuration, bad code, missing dependencies, or an impossible database connection.

---

## 10. Understanding Restart Loops

A typical restart loop looks like this:

```text
API starts
   |
   v
MongoDB unavailable
   |
   v
API exits with an error
   |
   v
Docker restarts API
   |
   v
API fails again
   |
   +---- repeat ----+
```

Symptoms may include:

```text
Restarting
Restarting
Restarting
```

or a continuously increasing restart count.

Do not respond by repeatedly running:

```bash
docker compose down
docker compose up -d
```

That may hide evidence without fixing the problem.

Investigate:

```bash
docker compose ps -a
docker compose logs --tail 100 api
docker inspect <api-container>
docker compose config
docker network inspect <network>
```

Look for:

- Missing required environment variables.
- A wrong `MONGO_URI` hostname or port.
- MongoDB authentication failures.
- A health-check command that is not installed.
- A process listening on the wrong port.
- A memory limit or out-of-memory termination.
- A syntax or application startup error.
- A dependency that is running but not ready.

A restart loop is a signal to find the root cause, not proof that the restart policy is working well.

---

## 11. API Startup Resilience

Even with Compose health checks, the API should handle temporary dependency failures reasonably. MongoDB may restart, a network connection may drop, or a managed dependency may briefly be unavailable.

A robust startup strategy can include:

- Connection retry logic.
- Exponential backoff.
- Connection and operation timeouts.
- Clear structured error logging.
- A readiness state that remains false until dependencies are usable.
- Graceful shutdown of open connections.

Conceptually:

```text
API starts
   |
   v
Connect to MongoDB
   |
 +-+----------------+
 |                  |
Success            Failure
 |                  |
 v                  v
Ready            Wait and retry
                       |
                       v
                 Backoff increases
```

Exponential backoff avoids hammering a dependency during an outage:

```text
Attempt 1 -> wait 1 second
Attempt 2 -> wait 2 seconds
Attempt 3 -> wait 4 seconds
Attempt 4 -> wait 8 seconds
```

Use an upper bound so the wait does not become unbounded. Log the dependency and attempt number, but never log passwords or full secret-bearing connection strings.

### When should an API exit?

There is no universal answer. A service may fail fast when a required configuration value is missing or invalid. It may retry when a dependency is temporarily unavailable. The design should distinguish permanent configuration errors from transient operational failures.

```text
Missing MONGO_URI       -> fail clearly
Temporary Mongo outage  -> retry with backoff
Invalid credentials     -> alert and stop or use controlled retry
```

---

## 12. Graceful Shutdown and `SIGTERM`

Docker normally asks a containerized process to stop by sending `SIGTERM`. A production-friendly Node process should handle that signal:

```text
Receive SIGTERM
      |
      v
Stop accepting new traffic
      |
      v
Finish in-flight requests where possible
      |
      v
Close HTTP server
      |
      v
Close MongoDB connections
      |
      v
Exit with a controlled status
```

A minimal Node pattern is:

```javascript
const shutdown = async (signal) => {
  console.log(`${signal} received; shutting down`);

  server.close(async () => {
    await mongoose.connection.close();
    process.exit(0);
  });

  setTimeout(() => process.exit(1), 10000).unref();
};

process.on("SIGTERM", () => shutdown("SIGTERM"));
process.on("SIGINT", () => shutdown("SIGINT"));
```

The exact implementation depends on the framework and database client. The important behavior is to stop accepting new work, allow active work to finish within a deadline, close resources, and exit.

Without graceful shutdown, replacements can cause:

- Dropped requests.
- Incomplete writes.
- Connection cleanup problems.
- Long-running requests being terminated abruptly.
- Misleading errors during deployments.

---

## 13. Safe Deployment Replacement

Suppose the current API is `api:v1` and the new version is `api:v2`.

A safe replacement conceptually follows:

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
Verify the request path
   |
   v
Route traffic to v2
   |
   v
Gracefully terminate v1
   |
   v
Monitor the deployment
```

Another view:

```text
                 api:v1
                   |
                   | serving traffic
                   v
             Start api:v2
                   |
                   v
             v2 becomes healthy
                   |
                   v
          Route traffic to v2
                   |
                   v
             SIGTERM -> v1
                   |
                   v
                v1 exits
```

Plain Docker Compose does not automatically provide a complete zero-downtime rolling deployment controller. The exact traffic-switching method depends on the deployment platform, reverse proxy, service manager, and orchestration strategy.

The principle still matters:

> Never assume a newly started container is ready for production traffic.

This becomes the foundation for Kubernetes rolling updates, blue-green deployments, and canary releases.

---

## 14. Production-Style MERN Compose Example

Start with a clear dependency model:

```yaml
services:
  mongodb:
    image: mongo:8
    volumes:
      - mongo-data:/data/db
    networks:
      - mern-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "mongosh --quiet --eval 'db.adminCommand({ ping: 1 }).ok' | grep 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

  api:
    build: ./backend
    environment:
      NODE_ENV: ${NODE_ENV:?NODE_ENV must be provided}
      MONGO_URI: ${MONGO_URI:?MONGO_URI must be provided}
      PORT: 3000
    networks:
      - mern-network
    restart: on-failure:5
    depends_on:
      mongodb:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s

  nginx:
    build: ./nginx
    ports:
      - "80:80"
    networks:
      - mern-network
    depends_on:
      api:
        condition: service_healthy
    restart: unless-stopped

volumes:
  mongo-data:

networks:
  mern-network:
```

This configuration expresses useful intent:

- MongoDB has persistent storage.
- MongoDB has a health signal.
- The API waits for a healthy MongoDB according to Compose's dependency condition.
- The API has a bounded restart policy.
- Nginx waits for the API health condition.
- Only Nginx publishes a public port.

It still does not replace application retry logic, monitoring, backups, security controls, or a real deployment controller.

### Validate before starting

```bash
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs --tail 100 api
```

### Important image compatibility check

Before using this example, verify:

- `mongosh` exists in the MongoDB image.
- `grep` exists in the MongoDB image.
- `wget` exists in the API image.
- The API listens on `0.0.0.0:3000`, not only `127.0.0.1:3000`.
- `/health` returns a successful response.

A health check that references a missing executable will report a false failure.

---

## 15. Health Endpoint Design

A simple endpoint:

```javascript
app.get("/health", (_request, response) => {
  response.status(200).json({ status: "ok" });
});
```

A dependency-aware readiness endpoint might track database state:

```javascript
app.get("/ready", (_request, response) => {
  const ready = mongoose.connection.readyState === 1;

  if (!ready) {
    return response.status(503).json({ status: "not_ready" });
  }

  return response.status(200).json({ status: "ready" });
});
```

Use status `503 Service Unavailable` when the service is alive but not ready to receive traffic. This lets a proxy, load balancer, or deployment system distinguish temporary unavailability from a successful response.

Avoid exposing:

- Database passwords.
- Full connection strings.
- Internal stack traces.
- Sensitive infrastructure details.

A health endpoint should be observable without becoming an information leak.

---

## 16. Troubleshooting Toolkit

### See service state

```bash
docker compose ps
docker compose ps -a
```

### Read recent logs

```bash
docker compose logs --tail 100 api
docker compose logs --tail 100 mongodb
docker compose logs --tail 100 nginx
```

### Follow logs

```bash
docker compose logs -f api
```

### Inspect a container

```bash
docker inspect <container>
```

Look for exit code, restart count, health status, mounts, environment configuration, and network attachments.

### Print health state only

PowerShell-friendly format output:

```powershell
docker inspect --format='{{json .State.Health}}' <container>
```

### Validate Compose interpolation

```bash
docker compose config
```

### Inspect networks

```bash
docker network ls
docker network inspect <network>
```

### Test service DNS

```bash
docker compose exec api getent hosts mongodb
docker compose exec nginx getent hosts api
```

### Test the API from inside the network

```bash
docker compose exec nginx wget -qO- http://api:3000/health
```

### Test the public path

```bash
curl -f http://localhost/api/health
```

### Check resource usage

```bash
docker stats --no-stream
```

A restart caused by memory pressure may not be explained by application logs alone. Check container state and host resource metrics.

---

## 17. Incident Scenario: API Is Restarting

Dashboard:

```text
API unavailable
```

Command:

```bash
docker compose ps
```

Output:

```text
mongodb   Up
api       Restarting
nginx     Up
```

### Do not begin with blind restarts

Avoid immediately doing:

```bash
docker compose down
docker compose up -d
```

This can destroy useful timing and log evidence.

### Investigation sequence

```text
1. Check API logs
2. Check API exit and restart state
3. Check MongoDB logs and health
4. Check MONGO_URI and other configuration
5. Check Docker network membership
6. Test MongoDB DNS from the API container
7. Test MongoDB connectivity
8. Check resource usage
9. Identify the root cause
10. Apply a targeted fix and retest
```

Commands:

```bash
docker compose logs --tail 100 api
docker inspect <api-container>
docker compose logs --tail 100 mongodb
docker compose config
docker network inspect <network>
docker compose exec api getent hosts mongodb
docker stats --no-stream
```

Possible root causes include:

- `MONGO_URI` points to `localhost`.
- MongoDB is not ready or has failed its health check.
- The API uses credentials that MongoDB rejects.
- The API health check calls a missing executable.
- The API listens on the wrong port.
- The application exits on every transient connection failure.
- The container is being killed because of memory pressure.

The goal is to distinguish:

```text
Container state
      |
      v
Application state
      |
      v
Dependency state
```

---

## 18. `502 Bad Gateway` and Unhealthy Services

If Nginx returns `502`, check the request path in order:

```text
Client
  |
  v
Nginx configuration
  |
  v
api service DNS
  |
  v
API listening port
  |
  v
API health and process state
  |
  v
MongoDB readiness
```

Commands:

```bash
docker compose logs --tail 100 nginx
docker compose logs --tail 100 api
docker compose exec nginx getent hosts api
docker compose exec nginx wget -qO- http://api:3000/health
docker compose ps
```

A container marked healthy does not guarantee that every route works. The health test may only call `/health`; authentication middleware, application data, or another dependency may still fail on the real request path.

---

## 19. Best Practices

### 1. Define meaningful health checks

Do not rely solely on `Up`. Test a real, safe condition.

### 2. Match checks to the image

Confirm that every health-check executable exists in the runtime image.

### 3. Use application retry logic

Health checks and `depends_on` do not replace retries, timeouts, and backoff in the application.

### 4. Use restart policies intentionally

Restart policies should recover from suitable failures without hiding persistent defects.

### 5. Implement graceful shutdown

Handle `SIGTERM`, finish in-flight work where possible, close resources, and exit.

### 6. Keep internal services private

Expose Nginx publicly and keep API and MongoDB on the private network unless a controlled operational requirement says otherwise.

### 7. Log useful context without secrets

Include service, operation, error, timestamp, and correlation context. Do not include passwords or full secret-bearing URIs.

### 8. Test failure paths deliberately

Break a health endpoint in a lab, observe `unhealthy`, restore it, and document the recovery.

### 9. Monitor restart counts

A service that is technically running but restarting repeatedly is not reliable.

### 10. Treat readiness as traffic control

A service should receive traffic only when it can perform its required work.

---

## 20. Common Mistakes

### Mistake 1: Assuming `depends_on` means ready

It primarily describes dependency ordering. Use health-aware conditions where supported and implement application retries.

### Mistake 2: Using `curl` in an image without `curl`

The health check fails because the command is missing, even if the application is fine.

### Mistake 3: Using restart policies to hide bugs

A crash-restart loop is not resilience. Inspect the first failure and fix the cause.

### Mistake 4: Checking only container state

Always reason through container, process, application, and dependency state.

### Mistake 5: Health check is too shallow

A check that only tests an open port may pass while the database-dependent API is unusable.

### Mistake 6: Health check is too expensive

A check that performs heavy queries or writes data can harm the service it is measuring.

### Mistake 7: No graceful shutdown

Abrupt termination can drop active requests and leave resources in an uncertain state.

### Mistake 8: Wrong listening interface

An API listening only on `127.0.0.1` may not be reachable from Nginx. Container services normally need to listen on `0.0.0.0` inside the container.

### Mistake 9: No timeout or retry limit

A blocked health command or unlimited retry loop can create a different operational failure.

---

## 21. Interview Preparation

### Q1. What is a Docker health check?

A command Docker runs periodically to test whether a defined service condition passes. Docker records the result as starting, healthy, or unhealthy.

### Q2. What is a restart policy?

A Docker setting that controls whether a container should automatically restart after it exits and under what conditions.

### Q3. Why is `depends_on` not sufficient for production readiness?

Because starting a dependency first does not prove that it has finished initializing or can accept the required requests.

### Q4. What is the difference between liveness and readiness?

Liveness asks whether the process is alive enough to continue. Readiness asks whether the service can currently receive production traffic.

### Q5. How do you prevent an API from failing when MongoDB starts slowly?

Use an appropriate MongoDB health check, a health-aware Compose dependency where supported, application connection retries, exponential backoff, timeouts, and readiness handling.

### Q6. Why is graceful shutdown important?

It allows the service to stop accepting new work, finish in-flight requests where possible, close resources, and exit cleanly during deployment or restart.

### Q7. An API container keeps restarting. What do you check?

I inspect API logs and exit state first, then verify configuration, network membership, service-name resolution, MongoDB readiness, health-check commands, listening ports, and resource usage. I would not assume that restarting the container fixes the root cause.

### Q8. A container is healthy but users still receive errors. What does that indicate?

The health check may be too shallow and may not test the complete user request path or every dependency. I would test the actual Nginx-to-API path and inspect application logs.

### Q9. Why should an API return `503` when it is not ready?

`503 Service Unavailable` tells a proxy or client that the service is temporarily unable to handle traffic, rather than falsely reporting success.

### Q10. What does a safe replacement deployment look like?

Build the new version, start it, verify its health and request path, route traffic to it, gracefully terminate the old version, and monitor the result.

---

## 22. Review Questions From Earlier Lessons

### Networking

Why is this usually correct inside the API container?

```text
mongodb://mongodb:27017
```

Because `mongodb` resolves to the Compose service. `localhost` refers to the API container itself.

### Reverse proxy

Nginx returns `502 Bad Gateway`. Investigate Nginx upstream configuration, API availability, Docker networking, DNS, ports, API health, and the MongoDB dependency.

### Image caching

Why copy dependency files before the rest of the source?

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

Because dependency layers can remain cached when only application source changes.

### Security

Why should production secrets not be baked into an image?

Image layers can be stored, copied, inspected, and distributed. A secret in an image can outlive the container and spread to systems that should never receive it.

### Persistence

Why does MongoDB need a volume?

The container is replaceable, but the database data should survive container recreation and service replacement.

---

## 23. Day 34 Summary

The most important distinction is:

```text
Container started
       !=
Application ready
       !=
Application healthy
```

### Dependencies

```text
depends_on
```

expresses a relationship, but it does not magically make a dependency ready.

### Health

```text
healthcheck
```

provides a defined signal that Docker can record and other deployment logic can use.

### Resilience

```text
restart policy
+ application retries
+ timeouts and backoff
+ health checks
```

Together, these improve recovery. None of them removes the need to understand the failure.

### Deployment

```text
Start new container
       |
       v
Health check
       |
       v
Verify request path
       |
       v
Route traffic
       |
       v
Gracefully stop old version
```

### Troubleshooting

```text
Logs
  |
Container state
  |
Health state
  |
Network
  |
Configuration
  |
Dependencies
  |
Root cause
```

Day 35 will bring together Compose, networks, volumes, health checks, restart policies, configuration, Nginx, security, and production operations.
