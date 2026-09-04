# Day 28 - Interview Questions: Docker Logging, Monitoring & Resource Management

## Beginner Level

### Q1. How do you view container logs?

```bash
docker logs <container>
```

### Q2. How do you follow logs in real time?

```bash
docker logs -f <container>
```

### Q3. How do you monitor live container resource usage?

```bash
docker stats
```

### Q4. What is the difference between a running and a healthy container?

Running indicates that the container's main process is active. Healthy means that the container passes a defined health condition.

---

## Intermediate Level

### Q5. How would you investigate a container that repeatedly exits?

Start with:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
```

Then investigate the exit status, environment variables, configuration, dependencies, health behavior, CPU, memory, disk, and restart policy.

### Q6. What could cause exit code 137?

A process received `SIGKILL`. Memory pressure and an OOM kill are common possibilities, but the exit code alone is not proof. Confirm using logs, limits, resource metrics, and host evidence.

### Q7. What does `docker system df` show?

It summarizes Docker disk usage from images, containers, local volumes, and build cache.

### Q8. Why can automatic restarts be dangerous?

They can hide the underlying failure and create a restart loop. Restarting improves availability only when the failure is transient or already understood.

### Q9. What is the difference between liveness and readiness?

Liveness asks whether the process should continue running. Readiness asks whether the application can currently serve useful traffic. A dependency failure may make an app unready without requiring a restart.

---

## Advanced Level

### Q10. A MERN API container restarts every few minutes. How do you investigate?

1. Check `docker ps -a` and restart behavior.
2. Read recent and historical logs.
3. Check the exit code.
4. Inspect configuration, environment, mounts, networks, and health status.
5. Monitor CPU and memory with `docker stats`.
6. Check host CPU, memory, disk, and Docker storage.
7. Check MongoDB status, logs, readiness, and connectivity.
8. Distinguish application crashes, OOM kills, dependency failures, and configuration errors.
9. Apply a targeted fix and verify with health checks and logs.

### Q11. How can a container be `Up` while the API is unavailable?

The main process may still be alive while the application is stuck, unable to connect to MongoDB, failing requests, or listening on the wrong interface. Process state is not the same as functional health.

### Q12. Why should applications normally log to stdout and stderr in containers?

The container runtime and platform can collect, route, retain, and centralize those streams. Unmanaged files inside a container can consume the writable layer and disappear when the container is replaced.

### Q13. Why should you avoid blindly running `docker system prune --volumes`?

It can remove unused-looking volumes that contain important persistent data, including database files. Cleanup must be based on ownership, retention, and backup knowledge.

### Q14. What is a good health-check design?

It should be fast, repeatable, safe, and meaningful. It should test the condition the platform needs to know, and its command must exist in the image.

### Q15. How do restart policies, health checks, and application retries differ?

Restart policies control container recreation after process termination. Health checks report a defined condition. Application retries handle transient dependency failures inside the running application. They complement one another but do not replace one another.

## Short Strong Answers

**How do you start a container incident investigation?**  
Check state, logs, exit status, configuration, resource metrics, host resources, and dependencies.

**Does `Up` mean healthy?**  
No. It only says the container process is active.

**What does exit 137 suggest?**  
A `SIGKILL`, possibly memory pressure; verify with evidence.

**Why not just restart it?**  
A restart may hide the root cause and create a restart loop.

**What is observability?**  
Using logs, metrics, health signals, and context to understand system behavior and diagnose failures.
