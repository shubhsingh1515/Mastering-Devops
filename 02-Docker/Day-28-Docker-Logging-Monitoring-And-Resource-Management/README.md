# DevOps Mentorship Program - Day 28

## Phase 2: Docker & Containers

### Topic: Docker Logging, Monitoring & Resource Management


## Review From Earlier Lessons

### Q1. Why should production Node containers normally run as non-root?

To apply least privilege and reduce the potential impact of a compromised application.

### Q2. What is the difference between a volume and a backup?

A volume provides persistent storage. A backup is an independent recovery mechanism that can restore data after deletion, corruption, or host loss.

### Q3. Why should Node not use `localhost` for MongoDB in another container?

Inside the Node container, `localhost` means the Node container itself. The application should use the MongoDB service name on the shared Docker network.

```text
mongodb://mongodb:27017/mern
```

### Q4. Why should production use explicit image versions rather than blindly using `latest`?

A mutable tag can point to different image content over time, making deployments less deterministic and rollbacks harder to reason about.

### Q5. What does `-p 8080:3000` mean?

```text
Host port 8080
       |
       v
Container port 3000
```

---

## 1. Why Container Monitoring Matters

A container being `Up` does not necessarily mean that the application is healthy.

A MERN production system can have this state:

```text
Container       -> running
Application     -> failing requests
CPU             -> 100 percent
Memory          -> exhausted
Disk            -> full
MongoDB         -> unavailable
Requests        -> timing out
```

Observability combines multiple signals:

```text
Logs
  +
Metrics
  +
Health checks
  +
Configuration
```

One signal rarely explains an incident by itself. For example, `docker ps` tells you that a process exists, but logs and health checks tell you whether it is useful.

---

## 2. Learning Objectives

By the end of this lesson, you should be able to:

- Inspect container logs.
- Follow logs in real time.
- Monitor CPU, memory, network, and process usage.
- Distinguish container state from application health.
- Diagnose CPU, memory, disk, and dependency pressure.
- Understand common exit codes.
- Configure and evaluate restart policies.
- Distinguish liveness from readiness.
- Troubleshoot a failing MERN API container.
- Explain container observability in an interview.

---

## 3. Container Logs

View all logs for a container:

```bash
docker logs mern-api
```

Follow new log entries:

```bash
docker logs -f mern-api
```

Show only recent entries:

```bash
docker logs --tail 100 mern-api
```

Include timestamps:

```bash
docker logs -t mern-api
```

Combine useful options:

```bash
docker logs -f --tail 50 -t mern-api
```

For Compose services:

```bash
docker compose logs api
docker compose logs -f api
docker compose logs --tail 100 mongodb
```

Logs should help answer:

- When did the failure begin?
- Which request or dependency failed?
- Did configuration change?
- Did the process shut down cleanly?
- Is the container repeatedly restarting?

Avoid logging secrets, tokens, passwords, or complete sensitive request bodies.

---

## 4. Container Logs Versus Application Logs

A Node application may write:

```javascript
console.log('API started');
```

Docker captures the process standard output and standard error streams:

```text
Node.js process
      |
      +--> stdout
      +--> stderr
              |
              v
      Docker logging mechanism
              |
              v
          docker logs
```

Container logs and host filesystem log files are not automatically the same thing. A production logging design should decide:

- where logs are collected
- how long logs are retained
- how logs are searched
- how sensitive values are removed
- how log volume is controlled
- how logs are forwarded to a central system

A container should normally write application logs to stdout and stderr when the runtime platform is responsible for collection. Writing unmanaged log files inside a container can fill its writable layer.

---

## 5. Production Incident: API Is Up but Requests Fail

Suppose users report that API requests are failing. Start with:

```bash
docker ps
```

You see:

```text
mern-api   Up
```

That proves only that the container and its main process are still present. Continue:

```bash
docker logs --tail 100 mern-api
```

You may find:

```text
MongoServerSelectionError
```

Now investigate the dependency path:

```text
Node API
   |
   v
MongoDB
   |
   v
Docker network
   |
   v
Configuration and credentials
```

The lesson is simple:

> Container state is only one signal. Verify the application and its dependencies.

---

## 6. Docker Stats

Run live resource monitoring:

```bash
docker stats
```

For one container:

```bash
docker stats mern-api
```

Important columns include:

```text
CONTAINER
CPU %
MEM USAGE / LIMIT
MEM %
NET I/O
BLOCK I/O
PIDS
```

Use `docker stats` during an incident to form a hypothesis. It is not a complete application monitoring system because it does not explain which endpoint, query, or function is consuming the resource.

For a snapshot rather than an interactive stream:

```bash
docker stats --no-stream
```

---

## 7. CPU Troubleshooting

Suppose the API shows sustained high CPU usage. Do not immediately restart it, because a restart may remove evidence.

Investigate:

```text
What changed?
      |
      v
Traffic increased?
      |
      v
Infinite loop or expensive endpoint?
      |
      v
Database query problem?
      |
      v
Excessive logging?
      |
      v
Garbage collection or memory pressure?
```

Useful commands include:

```bash
docker stats mern-api
docker top mern-api
docker inspect mern-api
```

If the image contains the required tools:

```bash
docker exec -it mern-api sh
top
```

Host-level tools can also help:

```bash
top
ps aux --sort=-%cpu | head
```

Then compare the timing of the CPU increase with deployments, traffic, database changes, and application logs.

---

## 8. Memory Troubleshooting

Suppose memory usage keeps increasing:

```text
mern-api
Memory: 950 MB and rising
```

Possible causes include:

- a memory leak
- unbounded caching
- large request payloads
- unexpectedly large database results
- increased traffic
- a queue or worker backlog
- inefficient application behavior
- runtime garbage-collection pressure

Do not immediately conclude that Docker is leaking memory. First compare:

```text
Application workload
        +
Runtime behavior
        +
Container memory limit
        +
Host memory availability
```

Use:

```bash
docker stats --no-stream mern-api
docker inspect mern-api
docker logs --tail 100 mern-api
```

If the application has a memory limit, determine whether it is a sizing problem or whether the process is growing abnormally.

---

## 9. Out of Memory and Exit Code 137

If a container or host reaches a memory limit, a process may be terminated. Check stopped containers:

```bash
docker ps -a
```

Inspect the container:

```bash
docker inspect mern-api
```

Read its logs:

```bash
docker logs mern-api
```

Check host memory on Linux:

```bash
free -h
```

Exit code `137` commonly indicates that a process received `SIGKILL`. Memory pressure and an OOM kill are possible causes, but the number alone is not a complete diagnosis.

Use this investigation chain:

```text
Container stopped
      |
      v
Exit status and logs
      |
      v
Resource metrics
      |
      v
Container memory limit
      |
      v
Host memory
      |
      v
Application behavior
```

Do not conclude `137` means a Docker defect without checking the surrounding evidence.

---

## 10. Other Container Exit Codes

Check status with:

```bash
docker ps -a
```

You may see:

```text
Exited (1)
Exited (137)
Exited (0)
```

These are clues:

- `0` usually means the main process exited successfully.
- `1` commonly indicates an application or command failure.
- `137` commonly indicates `SIGKILL`, possibly from memory pressure.

The exact interpretation depends on the command, runtime, signal, and host. Always combine the exit code with logs, configuration, dependencies, and resource data.

---

## 11. Health Checks

Remember:

```text
Running is not the same as healthy
```

A Node process may still exist while MongoDB connectivity is broken or the API cannot serve useful requests.

A basic endpoint might be:

```text
GET /health
```

with a response such as:

```json
{
  "status": "ok"
}
```

The endpoint and container health check should match the application's purpose. A check that only verifies that the Node process responds may not prove that critical dependencies are available.

Health checks should be:

- fast
- deterministic
- safe to call repeatedly
- free of side effects
- based on a meaningful condition

---

## 12. Liveness Versus Readiness

These concepts are especially important in orchestrated environments.

### Liveness

Liveness asks:

> Is the application process alive enough to continue running?

A failed liveness check may cause the platform to restart the application.

### Readiness

Readiness asks:

> Is the application ready to receive useful traffic right now?

A failed readiness check should normally remove the instance from traffic without necessarily restarting it.

Example:

```text
Node process alive
        |
        v
MongoDB unavailable
        |
        +--> Liveness: possibly passing
        +--> Readiness: failing
```

The distinction prevents routing requests to an application that cannot complete its work. Avoid putting expensive or fragile dependency checks into liveness if a temporary dependency outage should not restart every application instance.

---

## 13. Restart Policies

Docker restart policies include:

```text
no
always
on-failure
unless-stopped
```

Example:

```bash
docker run \
  --restart=unless-stopped \
  myapp
```

A Compose service may use:

```yaml
services:
  api:
    restart: unless-stopped
```

Restart policies can improve availability after transient failures. They do not fix application bugs.

A broken application may produce:

```text
Crash
  |
  v
Restart
  |
  v
Crash
  |
  v
Restart loop
```

When a container restarts repeatedly, preserve evidence by checking logs, exit status, health behavior, configuration, dependencies, and resource pressure.

---

## 14. MERN Restart-Loop Example

Suppose:

```text
Node API
   |
   v
MongoDB unavailable
```

The API crashes and Docker restarts it:

```text
Node crash
   |
   v
Docker restart
   |
   v
Node starts
   |
   v
MongoDB unavailable
   |
   v
Node crashes again
```

The correct question is:

> Why is MongoDB unavailable?

Investigate:

- MongoDB container status
- MongoDB logs
- the API's `MONGO_URI`
- network and service-name configuration
- database readiness
- credentials and authentication
- application retry behavior

Restarting faster is not a substitute for fixing the dependency failure.

---

## 15. Docker Disk Usage

Docker consumes disk through:

- image layers
- stopped containers
- named volumes
- build cache
- container writable layers
- log files

Get a summary:

```bash
docker system df
```

Inspect detailed usage:

```bash
docker system df -v
```

List related resources:

```bash
docker images
docker ps -a
docker volume ls
docker builder du
```

A full disk can cause image builds, database writes, logging, and container startup to fail. Disk monitoring is therefore both an operational and availability concern.

---

## 16. Dangerous Cleanup Commands

This command can remove unused Docker resources:

```bash
docker system prune
```

A more aggressive command may include volumes:

```bash
docker system prune --volumes
```

Do not run cleanup blindly. Before removing anything, ask:

```text
What is unused?
What is still needed?
What data is persistent?
What image is required for rollback?
What retention policy applies?
```

Use a controlled process:

```text
Measure
   |
   v
Identify
   |
   v
Confirm
   |
   v
Clean safely
   |
   v
Monitor
```

A named volume may contain important database data. An old image may be the fastest rollback artifact. Cleanup requires operational context.

---

## 17. Hands-On Lab

Start the Compose stack:

```bash
docker compose up -d
```

Check service state:

```bash
docker compose ps
```

View recent API logs:

```bash
docker compose logs --tail 50 api
```

Follow API logs:

```bash
docker compose logs -f api
```

In another terminal, monitor resources:

```bash
docker stats
```

Check Docker disk usage:

```bash
docker system df
```

Inspect the API container:

```bash
docker inspect <api-container>
```

Test the health endpoint:

```bash
curl http://localhost:3000/health
```

You are combining:

```text
Logs
  +
Metrics
  +
Health
  +
Configuration
```

Record what each command tells you and which command you would use first during an incident.

---

## 18. Compose Observability Example

A production-oriented service might include:

```yaml
services:
  api:
    build: ./backend
    restart: unless-stopped
    environment:
      NODE_ENV: production
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s
```

The exact health-check command depends on the binaries installed in the image. Minimal images may not contain `wget` or `curl`.

Do not blindly copy a check that requires a missing command. Options include:

- install the required tool deliberately
- use a runtime already available in the image
- add a small health-check executable
- validate the command during image testing

Restart policy, health check, and application retry behavior solve different problems. They should be designed together.

---

## 19. Interview Preparation

### Beginner

#### Q1. How do you view container logs?

```bash
docker logs <container>
```

#### Q2. How do you monitor live container resource usage?

```bash
docker stats
```

#### Q3. What is the difference between a running and a healthy container?

Running indicates that the container's main process is active. Healthy means that the container passes a defined health condition.

---

### Intermediate

#### Q4. How would you investigate a container that repeatedly exits?

Start with:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
```

Then investigate the exit status, configuration, environment variables, dependencies, health behavior, and CPU, memory, and disk pressure.

#### Q5. What could cause exit code 137?

A process received `SIGKILL`. Memory pressure and an OOM kill are common possibilities, but logs, container limits, host memory, and other evidence must confirm the cause.

#### Q6. What does `docker system df` show?

It summarizes Docker disk usage from images, containers, local volumes, and build cache.

---

### Advanced Interview Scenario

> Your MERN API container restarts every few minutes. How do you investigate?

A strong investigation is:

```text
docker ps -a
      |
      v
Check restart behavior
      |
      v
docker logs
      |
      v
Check exit code
      |
      v
docker inspect
      |
      v
docker stats
      |
      v
Host CPU, memory, and disk
      |
      v
Application dependencies
      |
      v
MongoDB and configuration
      |
      v
Health and restart behavior
```

Then distinguish:

```text
Application crash
       vs
OOM kill
       vs
Dependency failure
       vs
Configuration error
       vs
Health or restart behavior
```

This is stronger than simply restarting the container.

---

## 20. Quiz

### 1. Which command follows Docker container logs?

A. `docker logs -f`  
B. `docker logs --watch-only`  
C. `docker tail`  
D. `docker stream`

### 2. Which command shows live CPU and memory usage?

A. `docker stats`  
B. `docker monitor`  
C. `docker top-stats`  
D. `docker metrics`

### 3. Does `docker ps` showing `Up` prove that an API is healthy?

A. Yes  
B. No

### 4. What is readiness concerned with?

A. Whether an application is ready to serve traffic  
B. Whether Docker is installed  
C. Whether an image is small  
D. Whether the host has SSH

### 5. What does a restart policy do?

A. Determines when Docker should restart a container  
B. Fixes application bugs  
C. Creates a volume  
D. Scans an image

### 6. What command summarizes Docker disk usage?

A. `docker system df`  
B. `docker disk`  
C. `docker storage`  
D. `docker df-host`

### 7. Why can automatic restarts be dangerous?

A. They can hide the underlying problem and create restart loops  
B. They delete images  
C. They disable networking  
D. They always corrupt volumes

### 8. What should you check when a container repeatedly exits?

A. Logs, exit status, configuration, dependencies, and resources  
B. Only DNS  
C. Only the image name  
D. Only Nginx

### 9. Why should you not blindly run `docker system prune`?

A. It can remove resources you still need  
B. It permanently disables Docker  
C. It changes Linux users  
D. It deletes the kernel

### 10. What is the key observability combination?

A. Logs, metrics, and health signals  
B. Dockerfile and SSH only  
C. DNS and Git only  
D. CPU only

### Answer Key

1. A  
2. A  
3. B  
4. A  
5. A  
6. A  
7. A  
8. A  
9. A  
10. A

---

## 21. Practical Assessment: Production Incident

Your MERN API is repeatedly restarting. `docker ps -a` shows:

```text
mern-api   Exited (137)
```

### Investigation

Start with recent logs:

```bash
docker logs --tail 100 mern-api
```

Inspect the container:

```bash
docker inspect mern-api
```

Check resource behavior:

```bash
docker stats --no-stream
```

Check the host:

```bash
free -h
```

Check Docker disk usage:

```bash
docker system df
```

### Your job

Determine whether the evidence points toward:

```text
Application crash
       or
Memory pressure or OOM
       or
Configuration problem
       or
Dependency failure
```

Then explain what you would change to prevent recurrence. Consider:

- an application fix
- memory or resource sizing
- explicit resource limits
- monitoring and alerting
- health and readiness behavior
- dependency resilience
- capacity planning
- a tested recovery procedure

Do not claim that exit code `137` proves OOM without supporting evidence.

---

## 22. Month 2 Cumulative Project Update

Your containerized MERN project should now include observability:

```text
                    Internet
                       |
                       v
                     Nginx
                       |
                Private network
                       |
               +-------+-------+
               v               v
          Frontend           Node API
                                |
                                v
                             MongoDB
                                |
                                v
                          Persistent volume
```

Operational layer:

```text
Containers
   |
   +-- Logs
   +-- CPU
   +-- Memory
   +-- Disk
   +-- Health
   +-- Restart behavior
```

Your project should demonstrate:

```bash
docker compose ps
docker compose logs
docker stats
docker system df
docker inspect
```

Document:

- the API health endpoint
- the container restart policy
- resource expectations
- MongoDB storage
- the backup strategy
- failure scenarios
- the recovery procedure
- log retention and sensitive-data handling

---

## 23. Final Interview Challenge

Answer aloud:

> A production Node.js container is restarting repeatedly. How do you determine whether the problem is the application, Docker, or the host?

A strong answer follows this path:

```text
Container state
     |
     v
Logs
     |
     v
Exit code
     |
     v
Container configuration
     |
     v
CPU and memory metrics
     |
     v
Host resources
     |
     v
Application dependencies
     |
     v
Health and restart behavior
```

Then explain:

> I would not treat the restart itself as the root cause. I would use logs, exit status, resource metrics, and dependency health to determine why the process is terminating.

---

## 24. Today's Key Takeaways

You learned:

- `docker logs`
- real-time log following
- `docker stats`
- CPU and memory monitoring
- exit codes
- OOM and resource-pressure troubleshooting
- liveness versus readiness
- restart policies
- restart loops
- Docker disk usage
- safe cleanup principles
- MERN production observability
- incident investigation

The mental model is:

```text
                 Production Container
                         |
        +----------------+----------------+
        v                v                v
      Logs            Metrics           Health
        |                |                |
        +----------------+----------------+
                         v
                    Diagnosis
                         |
              +----------+----------+
              v          v          v
          App issue   Resource   Dependency
                         issue       issue
```

### Key production principle

> Running is a state. Healthy is an observed condition. Always verify the application itself.

---

## Next Lesson - Day 29

Docker Production Deployment & Reverse Proxy:

- Nginx with Docker
- reverse proxying to containers
- HTTPS and TLS architecture
- internal versus public ports
- production Compose structure
- zero or minimal downtime deployment concepts
- MERN production routing
- common 502 and 504 Docker failures
- system design interview questions
