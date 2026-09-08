# Day 31 - Quiz: Docker Best Practices & Production-Ready Containers

## Questions

1. Which Dockerfile order generally provides better cache reuse?
   - A. `COPY . .` then `RUN npm ci`
   - B. `COPY package*.json ./`, `RUN npm ci`, then `COPY . .`
   - C. `RUN npm ci` before setting a working directory
   - D. The order never matters

2. What is the main security benefit of `USER node`?
   - A. Faster networking
   - B. Less memory usage
   - C. Reduced privileges
   - D. Automatic encryption

3. Where should production secrets ideally come from?
   - A. Dockerfile
   - B. Git repository
   - C. A protected secret-management mechanism
   - D. Image tag

4. What does a health check provide beyond seeing `Container: Up`?
   - A. It verifies a defined application health condition
   - B. It increases CPU
   - C. It creates a volume
   - D. It pushes an image

5. Why are multi-stage Docker builds useful?
   - A. They eliminate networking
   - B. They separate build dependencies from the final runtime image
   - C. They automatically deploy to a cloud provider
   - D. They replace Git

6. What is the purpose of handling `SIGTERM` in Node.js?
   - A. Increase MongoDB storage
   - B. Graceful shutdown
   - C. Build Docker images
   - D. Change the image tag

7. Why should you not manually install packages inside a production container?
   - A. Containers do not support packages
   - B. The change is not reproducible and disappears when the container is recreated
   - C. Docker automatically deletes packages
   - D. It changes the DNS server

8. Which is more appropriate for a production artifact?
   - A. `latest` only
   - B. `random`
   - C. A specific version or immutable digest
   - D. `test`

9. What does build, test, and deploy without rebuilding between environments mean?
   - A. Mutable infrastructure
   - B. Build once and promote the tested artifact
   - C. Manual deployment
   - D. Container debugging

10. Your Node container is running, but `/health` returns failure. What does this demonstrate?
    - A. Container state and application health are different concepts
    - B. Docker networking is always broken
    - C. The image must be deleted
    - D. Volumes are unavailable

11. Why does PID 1 matter in a container?
    - A. It determines the image tag
    - B. It affects signal handling and child-process management
    - C. It creates MongoDB volumes
    - D. It automatically scans dependencies

12. What is the main risk of putting a secret in a Dockerfile `ENV` instruction?
    - A. The value can become visible in image metadata or history
    - B. It makes the image immutable
    - C. It prevents networking
    - D. It disables health checks

13. What does `restart: unless-stopped` provide?
    - A. A possible automatic restart after container failure
    - B. A fix for application bugs
    - C. An image vulnerability scan
    - D. A database backup

14. What should resource limits be based on?
    - A. Arbitrary guesses only
    - B. Observed workload and validated runtime behavior
    - C. The image tag
    - D. The number of Dockerfiles

15. What is the best logging destination for a typical containerized application?
    - A. Unmanaged files only inside the container
    - B. stdout and stderr for runtime collection
    - C. A password-protected Dockerfile
    - D. The image layer

## Answer Key

1. B  
2. C  
3. C  
4. A  
5. B  
6. B  
7. B  
8. C  
9. B  
10. A  
11. B  
12. A  
13. A  
14. B  
15. B

## Score Guide

- 13-15: Production-ready understanding
- 10-12: Good; review the missed operational areas
- 7-9: Needs revision
- Below 7: Revisit Dockerfiles, security, lifecycle, and observability

## Bonus Questions

### 16. Why is `.dockerignore` not a complete secret-management solution?

It helps exclude files from the build context, but secrets can still enter through Dockerfile instructions, build arguments, logs, dependencies, or runtime configuration. Use a proper protected secret mechanism.

### 17. What should happen after `docker stop`?

The application should receive a termination signal, stop accepting new work, finish or cancel active work safely, close dependencies, close the server, and exit within a controlled timeout.

### 18. Why is a smaller image not automatically a better image?

It may lack required libraries, use an incompatible runtime, or miss security updates. Compatibility, maintainability, patching, and reproducibility matter too.

### 19. What is immutable infrastructure?

It is the practice of replacing instances with newly built, tested artifacts rather than modifying running instances in place.

### 20. What is the main Day 31 principle?

A production container should be replaceable, reproducible, secure, observable, and easy to shut down and recover.
