# Day 28 - Assignment: Docker Logging, Monitoring & Resource Management

## Objective

Use Docker observability commands to inspect a MERN stack, identify a failing service, understand resource pressure, and design a safe recovery procedure.

---

## Part 1: Start the Stack

From the directory containing `compose.yaml`:

```bash
docker compose up -d
docker compose ps
```

Record:

- service names
- container state
- published ports
- health status, if configured

### Question

Does a service shown as `Up` automatically mean that the API is healthy?

### Expected answer

No. `Up` indicates that the container process is active. A health check or real request is needed to verify application behavior.

---

## Part 2: Inspect Logs

Run:

```bash
docker compose logs --tail 50 api
docker compose logs --tail 50 mongodb
```

Follow API logs:

```bash
docker compose logs -f api
```

In another terminal, call the API:

```bash
curl http://localhost:3000/health
```

### Questions

1. Which process writes the log message?
2. Is the message on stdout or stderr?
3. Would the log still be available if the container restarted?
4. Does the log contain any secret or sensitive value?

---

## Part 3: Monitor Resources

Run:

```bash
docker stats
docker stats --no-stream
```

Record the API values for:

- CPU percentage
- memory usage and limit
- network I/O
- block I/O
- process count

### Question

Why is `docker stats` useful but not sufficient by itself?

### Expected answer

It shows resource behavior, but it does not identify the application endpoint, query, or code path causing the usage. Logs, application metrics, and host evidence are also required.

---

## Part 4: Inspect the Container

Find the API container and run:

```bash
docker inspect <api-container>
docker top <api-container>
```

Look for:

- restart policy
- health status
- environment configuration
- resource limits
- mounted volumes
- connected networks
- exit status

Do not share inspection output publicly if it contains credentials or tokens.

---

## Part 5: Test Health and Readiness

Test the endpoint:

```bash
curl http://localhost:3000/health
```

Explain the difference:

```text
Liveness  -> Is the process alive?
Readiness -> Can it serve useful traffic now?
```

### Scenario

The Node process responds, but MongoDB is unavailable.

Answer:

- Should liveness necessarily fail?
- Should readiness pass?
- Should traffic be sent to the API?

### Expected answer

Liveness may still pass because the process is alive. Readiness should normally fail if the API cannot perform required work, and traffic should be withheld until the dependency is usable.

---

## Part 6: Investigate a Restarting Container

Use:

```bash
docker ps -a
docker logs --tail 100 <api-container>
docker inspect <api-container>
docker stats --no-stream
```

Determine whether the evidence suggests:

- application crash
- configuration error
- MongoDB dependency failure
- health-check failure
- memory pressure
- CPU pressure
- disk exhaustion

### Important

Do not solve every restart incident by restarting the container again. Restarting can hide evidence and may create a restart loop.

---

## Part 7: Exit Code 137 Assessment

Assume the API shows:

```text
Exited (137)
```

Check:

```bash
docker logs --tail 100 <api-container>
docker inspect <api-container>
docker stats --no-stream
free -h
docker system df
```

### Question

Can you state with certainty that Docker caused an OOM kill from the exit code alone?

### Expected answer

No. Exit code `137` commonly indicates `SIGKILL`, and memory pressure is a possible cause. Confirm it with resource metrics, container limits, host memory, and logs.

---

## Part 8: Disk Usage and Safe Cleanup

Run:

```bash
docker system df
docker system df -v
docker images
docker ps -a
docker volume ls
```

Identify which resources are:

- required for the current stack
- needed for rollback
- persistent database data
- genuinely unused

Do not run this blindly:

```bash
docker system prune --volumes
```

Explain why deleting volumes can be dangerous.

### Expected answer

Volumes may contain persistent database data. Removing them can cause irreversible data loss if no separate backup is available.

---

## Part 9: Add Operational Configuration

Update the API service with a restart policy and health check:

```yaml
services:
  api:
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s
```

Confirm that the health-check command exists in the image. Rebuild and start:

```bash
docker compose up -d --build
docker compose ps
```

### Question

Why is a restart policy not a replacement for fixing the application?

### Expected answer

It may recover from a transient failure, but a persistent application or dependency problem will create a restart loop and delay diagnosis.

---

## Part 10: Production Incident Report

Write a short incident report containing:

1. What users observed
2. When the incident began
3. Container and health status
4. Relevant log messages
5. Exit status
6. CPU, memory, and disk observations
7. Dependency status
8. Root cause hypothesis
9. Immediate recovery action
10. Long-term prevention

Use this investigation path:

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
Configuration
     |
     v
Resource metrics
     |
     v
Host resources
     |
     v
Dependency health
     |
     v
Recovery and prevention
```

---

## Part 11: Submission Checklist

- [ ] Started the Compose stack
- [ ] Inspected service status
- [ ] Viewed and followed logs
- [ ] Monitored CPU and memory
- [ ] Inspected the container configuration
- [ ] Tested the health endpoint
- [ ] Explained liveness and readiness
- [ ] Investigated a simulated restart loop
- [ ] Explained exit code 137 carefully
- [ ] Checked Docker disk usage
- [ ] Documented safe cleanup boundaries
- [ ] Configured or evaluated a restart policy
- [ ] Documented a recovery procedure
- [ ] Wrote a root-cause hypothesis rather than only restarting the container
