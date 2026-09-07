# Day 30 - Quiz: Monthly Cumulative MERN Docker Project

## Questions

1. Inside a Docker Compose network, what hostname should an API use to connect to MongoDB named `mongodb`?
   - A. `localhost`
   - B. `127.0.0.1`
   - C. `mongodb`
   - D. `host.docker.internal`

2. What is the primary purpose of a MongoDB volume?
   - A. Image optimization
   - B. Persistent data storage
   - C. DNS resolution
   - D. CPU management

3. What does this mapping mean?

   ```yaml
   ports:
     - "3000:3000"
   ```

   - A. Host port 3000 maps to container port 3000
   - B. Container port 3000 maps only internally
   - C. Two containers use port 3000
   - D. MongoDB uses port 3000

4. What is the main purpose of `.dockerignore`?
   - A. Prevent containers from starting
   - B. Reduce unnecessary build context
   - C. Encrypt Docker images
   - D. Create Docker networks

5. Why copy `package*.json` before the rest of the source code?
   - A. To expose ports
   - B. To improve Docker layer-cache reuse
   - C. To create a volume
   - D. To configure DNS

6. What does `depends_on` not guarantee by itself?
   - A. A declared service relationship
   - B. Compose dependency metadata
   - C. Application readiness
   - D. That the service exists in the configuration

7. Which is preferable for identifying a production image?
   - A. `latest`
   - B. `test`
   - C. A specific version, commit identifier, or immutable digest
   - D. `new`

8. What is one advantage of running a container as non-root?
   - A. It makes MongoDB faster
   - B. It reduces privileges available to the process
   - C. It eliminates all vulnerabilities
   - D. It automatically encrypts traffic

9. Which command provides live container CPU and memory information?
   - A. `docker images`
   - B. `docker stats`
   - C. `docker tag`
   - D. `docker history`

10. A container exits with code 137. What is a reasonable first investigation?
    - A. DNS records
    - B. Git branches
    - C. Memory pressure and supporting evidence
    - D. Docker image tags

11. Which service is usually the intended public entry point in the target architecture?
    - A. MongoDB
    - B. Nginx
    - C. The volume
    - D. The internal network

12. What does `docker push` do?
    - A. Uploads an image to a registry
    - B. Starts a container
    - C. Creates a volume
    - D. Performs a database backup

13. Why should the application image not contain production secrets?
    - A. Images are reusable and may be copied or inspected
    - B. Secrets stop Docker networking
    - C. Registries cannot store secure images
    - D. Environment variables cannot be used at runtime

14. What is the best general rollback approach?
    - A. Rebuild arbitrary old source code
    - B. Redeploy the previous known-good image or digest
    - C. Delete the current volume
    - D. Change the MongoDB port

15. What is the difference between a running container and a healthy application?
    - A. Running means the process is active; healthy means a defined application condition passes
    - B. They are always identical
    - C. Healthy only means the image is small
    - D. Running means the database is backed up

16. What should a production troubleshooting workflow combine?
    - A. Logs, health checks, configuration, network checks, and resource metrics
    - B. Only container restarts
    - C. Only image tags
    - D. Only frontend browser logs

## Answer Key

1. C  
2. B  
3. A  
4. B  
5. B  
6. C  
7. C  
8. B  
9. B  
10. C  
11. B  
12. A  
13. A  
14. B  
15. A  
16. A

## Score Guide

- 14-16: Excellent - ready to explain the project in an interview
- 11-13: Strong - review the missed operational areas
- 8-10: Needs review - revisit networking, persistence, and troubleshooting
- Below 8: Revisit the Docker lessons before submitting the project

## Bonus Questions

### 17. Why does a named volume not replace backups?

It protects data from normal container replacement but does not independently protect against deletion, corruption, host failure, or disaster.

### 18. Why should the API use `0.0.0.0` rather than only `127.0.0.1`?

`0.0.0.0` allows the process to accept traffic through the container network interface. `127.0.0.1` only binds to the container's loopback interface.

### 19. What evidence would support an OOM diagnosis for exit code 137?

Increasing memory usage, a memory limit, host memory pressure, runtime or kernel OOM evidence, logs, and timing that connects the termination to resource exhaustion.

### 20. What is the core project principle?

The artifact tested should be the artifact deployed, containers should be replaceable, and important data should have durable storage plus independent backups.
