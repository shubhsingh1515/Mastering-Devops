# Day 28 - Quiz: Docker Logging, Monitoring & Resource Management

## Questions

1. Which command follows Docker container logs?
   - A. `docker logs -f`
   - B. `docker logs --watch-only`
   - C. `docker tail`
   - D. `docker stream`

2. Which command shows live CPU and memory usage?
   - A. `docker stats`
   - B. `docker monitor`
   - C. `docker top-stats`
   - D. `docker metrics`

3. Does `docker ps` showing `Up` prove that an API is healthy?
   - A. Yes
   - B. No

4. What is readiness concerned with?
   - A. Whether an application is ready to serve traffic
   - B. Whether Docker is installed
   - C. Whether an image is small
   - D. Whether the host has SSH

5. What does a restart policy do?
   - A. Determines when Docker should restart a container
   - B. Fixes application bugs
   - C. Creates a volume
   - D. Scans an image

6. What command summarizes Docker disk usage?
   - A. `docker system df`
   - B. `docker disk`
   - C. `docker storage`
   - D. `docker df-host`

7. Why can automatic restarts be dangerous?
   - A. They can hide the underlying problem and create restart loops
   - B. They delete images
   - C. They disable networking
   - D. They always corrupt volumes

8. What should you check when a container repeatedly exits?
   - A. Logs, exit status, configuration, dependencies, and resources
   - B. Only DNS
   - C. Only the image name
   - D. Only Nginx

9. Why should you not blindly run `docker system prune`?
   - A. It can remove resources you still need
   - B. It permanently disables Docker
   - C. It changes Linux users
   - D. It deletes the kernel

10. What is the key observability combination?
    - A. Logs, metrics, and health signals
    - B. Dockerfile and SSH only
    - C. DNS and Git only
    - D. CPU only

11. What does exit code 137 commonly indicate?
    - A. A process received `SIGKILL`, possibly because of memory pressure
    - B. The image was successfully built
    - C. The container created a volume
    - D. The DNS record is missing

12. What is the best description of readiness?
    - A. Whether the application can currently serve useful traffic
    - B. Whether the image has no vulnerabilities
    - C. Whether the host has a shell
    - D. Whether the container has a name

## Answer Key

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
11. A  
12. A

## Bonus Questions

### 13. Why can a container be running but unhealthy?

Its main process may still exist while the application is failing requests, stuck, unable to reach a dependency, or otherwise unable to provide useful service.

### 14. Why should exit code 137 not be treated as a complete diagnosis?

It indicates a `SIGKILL`, but several situations can produce that signal. Memory metrics, host state, logs, and limits are needed to identify the actual cause.

### 15. What should you verify before adding a Compose health check?

Verify that the endpoint exists, returns the expected status, has no harmful side effects, and that the command used by the check exists in the image.

### 16. Why can a restart policy make an incident harder to diagnose?

Repeated restarts can hide the original failure, rotate useful logs, and create a loop that keeps the service unavailable.

### 17. What should a safe Docker cleanup process include?

Measure disk usage, identify ownership and retention needs, protect volumes and rollback images, remove only confirmed-unused resources, and monitor afterward.
