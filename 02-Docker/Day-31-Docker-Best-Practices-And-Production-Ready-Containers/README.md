# DevOps Mentorship Program - Day 31

## Phase 2: Docker & Containers

### Docker Best Practices & Production-Ready Container Design

**Level:** Intermediate to Professional  
**Focus:** Turning a working Docker setup into a reproducible, secure, observable, and operable production container.

> Day 30 was the monthly cumulative Docker/MERN project. Today continues the Docker phase. CI/CD is not introduced yet.

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Design production-ready Docker images.
- Apply Dockerfile best practices.
- Understand container lifecycle and immutability.
- Separate configuration from the image.
- Reduce image attack surface and size.
- Handle signals and graceful shutdown in Node.js.
- Use health checks and restart policies correctly.
- Apply resource limits and operational controls.
- Explain Docker design decisions in a DevOps interview.

---

## 2. Why Production Docker Design Matters

A developer can run:

```bash
docker build .
docker run myapp
```

and say that the application is containerized. A production review asks more questions:

- Is the image reproducible?
- Is it larger than necessary?
- Does the process run as root?
- Are secrets inside the image?
- What happens when Docker sends `SIGTERM`?
- Can Docker determine whether the application is healthy?
- Can the container restart safely?
- Can we identify the exact version running?
- Does important data incorrectly live inside the container?
- Can the deployment be rolled back?

Production Docker is about **operability**, not just packaging.

---

## 3. Production Container Mental Model

A production container should be replaceable, reproducible, and disposable:

```text
                 Container
        +-------------------------+
        | Application             |
        | Configuration           |
        | Runtime dependencies    |
        +------------+------------+
                     |
              External resources
                     |
        +------------+-------------+
        v            v             v
    Database       Secrets       Storage
```

The image should contain the application and runtime dependencies. Environment-specific configuration, credentials, and persistent data should be supplied separately.

Do not design a production system around manually changing a running container. Make changes in source or the Dockerfile, build a new image, test it, and replace the container.

---

## 4. Dockerfile Best Practices

A basic Dockerfile may work but still be difficult to operate:

```dockerfile
FROM node:22

WORKDIR /app

COPY . .

RUN npm install

CMD ["node", "server.js"]
```

Problems include:

1. `node:22` may contain more packages than the application needs.
2. Copying all source before installing dependencies weakens cache reuse.
3. `npm install` is less reproducible than `npm ci` when a lockfile exists.
4. The process may run as root.
5. Development files, local dependencies, or secrets may enter the image.
6. The image does not clearly document health, resource, or shutdown behavior.

A better Node.js starting point is:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY . .

USER node

EXPOSE 3000

CMD ["node", "server.js"]
```

### Why the order matters

```text
Package manifests
        |
        v
Install dependencies
        |
        v
Copy application source
```

Dependency manifests change less frequently than application source. Keeping them in an earlier layer allows Docker to reuse dependency installation when only source files change.

`EXPOSE` documents the container port; it does not publish that port to the host. Publishing still requires a runtime or Compose port mapping.

---

## 5. Layer Caching

Suppose the project contains:

```text
package.json
package-lock.json
server.js
routes/
controllers/
models/
```

If only `server.js` changes, this structure is cache-friendly:

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

The dependency layer can usually remain cached:

```text
Source change
     |
     v
Dependency input unchanged
     |
     v
Dependency layer reused
     |
     v
Only later source layer rebuilt
```

With this order, source changes can invalidate the layer before dependency installation:

```dockerfile
COPY . .
RUN npm ci
```

If `package.json` or `package-lock.json` changes, reinstalling dependencies is expected because the dependency graph may have changed.

Cache behavior depends on the instruction, build context, inputs, and builder. Check build output instead of assuming a layer was reused.

---

## 6. Keep Runtime Images Small

A production runtime image should not contain unnecessary:

- Git history
- development dependencies
- test tools
- documentation
- temporary files
- local caches
- compilers and build tools
- source maps that are not needed at runtime

For Node.js applications:

```bash
npm ci --omit=dev
```

can omit development dependencies from a runtime installation.

Do not optimize for size alone. The real goal is:

> Use the smallest practical image without sacrificing compatibility, security updates, reliability, or maintainability.

A smaller image generally means faster pulls and a smaller attack surface, but a broken or unpatched image is not an improvement.

---

## 7. Multi-Stage Builds

A React frontend needs Node and build tools to create static assets, but the production server may only need Nginx.

```dockerfile
FROM node:22-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
```

The stages have different purposes:

```text
Build stage
  - Node
  - npm
  - source code
  - development dependencies
  - build tools
        |
        v
   static assets
        |
        v
Runtime stage
  - Nginx
  - production assets only
```

The final image does not need the complete Node build environment. This reduces size and removes tools that are unnecessary at runtime.

For a Node API, use a separate dependency or build stage when it helps keep the runtime image focused. Copy every file required at runtime, not only `server.js`.

---

## 8. Run as Non-Root

A common weak model is:

```text
Container
    |
    v
root
    |
    v
Application
```

Prefer:

```dockerfile
USER node
```

when the image and application support it.

Verify the runtime identity:

```bash
docker exec -it <container> whoami
docker exec -it <container> id
```

Expected output should identify a restricted user such as `node`, not `root`.

Running as non-root follows least privilege and reduces the potential impact of an application compromise. It does not make the application completely secure. Image, dependency, network, filesystem, secret, and application security still matter.

---

## 9. Configuration Versus Image

Do not bake environment-specific configuration into an image:

```text
Production configuration
        |
        v
Docker image
```

Instead, build one artifact and inject configuration at runtime:

```text
Same image
   |
   +--> Development configuration
   +--> Staging configuration
   +--> Production configuration
```

Example image:

```text
registry.example.com/team/mern-api:abc123
```

Staging might provide:

```text
NODE_ENV=staging
MONGO_URI=<staging-uri>
```

Production might provide:

```text
NODE_ENV=production
MONGO_URI=<production-uri>
```

The image remains identical. This prevents environment-specific rebuilds and makes promotion more reliable.

---

## 10. Never Bake Secrets Into Images

Never do this:

```dockerfile
ENV MONGO_PASSWORD="supersecret"
```

Never do this:

```dockerfile
COPY .env .
```

Never commit a real `.env` file to Git.

Images can be inspected, copied, cached, pushed to registries, and retained in old versions. A secret in an image can remain exposed even after the source line is removed.

Use this model:

```text
Secret store or protected configuration
                |
                v
            Deployment
                |
                v
        Container environment
```

`.dockerignore` helps prevent accidental inclusion, but it is not a complete secrets-management solution.

---

## 11. Health Checks

A container can be running while the application is unhealthy:

```text
Container running
       !=
Application healthy
```

Expose a safe endpoint:

```text
GET /health
```

Example response:

```json
{
  "status": "ok"
}
```

A useful health check should be:

- fast
- deterministic
- safe to call repeatedly
- free of side effects
- based on a meaningful condition

A process-only check may pass while MongoDB is unavailable. Decide whether the endpoint should test only process liveness or whether a separate readiness check should verify critical dependencies.

---

## 12. Graceful Shutdown in Node.js

When Docker stops a container, the application can receive `SIGTERM`. A production Node.js process should stop safely rather than abruptly terminating active work.

Example:

```javascript
process.on("SIGTERM", async () => {
  console.log("SIGTERM received");

  await mongoose.connection.close();

  server.close(() => {
    console.log("HTTP server closed");
    process.exit(0);
  });
});
```

A robust implementation should also prevent new work, handle shutdown errors, and avoid hanging forever. A simplified lifecycle is:

```text
SIGTERM
   |
   v
Stop accepting new requests
   |
   v
Finish in-flight work
   |
   v
Close database connection
   |
   v
Close HTTP server
   |
   v
Exit cleanly
```

Graceful shutdown matters during deployments, scaling, container replacement, and rolling updates. It reduces dropped requests and incomplete database operations.

---

## 13. Why PID 1 Matters

Inside a container, the main process has a special role as PID 1. Prefer an exec-form command:

```dockerfile
CMD ["node", "server.js"]
```

over unnecessarily wrapping the application in a shell command.

Correct signal delivery matters because Docker sends termination signals to the container process. If a shell wrapper absorbs or mishandles signals, the Node process may not receive `SIGTERM` as expected.

Use an init process when the application creates child processes and needs proper child reaping. The important principle is to understand which process receives signals and how the process tree shuts down.

---

## 14. Restart Policies

Examples include:

```yaml
restart: unless-stopped
```

and:

```yaml
restart: on-failure
```

A direct Docker example is:

```bash
docker run \
  --restart=unless-stopped \
  myapp
```

Restart policies can improve availability after transient failures, but:

```text
Restarting != Fixing
```

A persistent bug can create:

```text
Application bug
      |
      v
Container crashes
      |
      v
Docker restarts it
      |
      v
Application crashes again
      |
      v
Restart loop
```

Always pair restart behavior with logs, health checks, alerting, and root-cause investigation.

---

## 15. Resource Limits

A container should not necessarily consume unlimited host resources.

Compose configuration may use runtime-specific resource settings such as:

```yaml
services:
  api:
    deploy:
      resources:
        limits:
          cpus: "1"
          memory: 512M
```

The exact behavior depends on the Compose implementation and runtime mode. Validate the effective limits in the environment where the stack runs.

Direct Docker usage can apply limits with:

```bash
docker run \
  --memory=512m \
  --cpus=1 \
  myapp
```

The purpose is to create a boundary:

```text
Application
     |
     v
Resource limit
     |
     v
Host and neighboring services remain protected
```

Choose limits from observed workload, not arbitrary numbers. Monitor for both starvation and uncontrolled growth.

---

## 16. Logging Best Practices

Applications should generally write logs to:

```text
stdout
stderr
```

Docker and the hosting platform can then collect and forward those streams:

```text
Node.js
   |
   +--> stdout
   +--> stderr
          |
          v
   Docker logging mechanism
          |
          v
   Central logging system
```

Use:

```bash
docker logs <container>
docker logs -f <container>
```

Avoid relying on unmanaged log files inside the container. They can fill the writable layer and disappear when the container is replaced.

Never log passwords, tokens, connection strings with credentials, or sensitive request bodies.

---

## 17. Container Immutability

Avoid changing a running production container manually:

```bash
docker exec -it api sh
```

then:

```bash
apt install something
vim config.js
npm install package
```

Those changes are not represented in the Dockerfile or source configuration. They disappear when the container is recreated and cannot be reliably reproduced on another machine.

Use this workflow instead:

```text
Change Dockerfile or source
        |
        v
Build a new image
        |
        v
Test the image
        |
        v
Deploy a replacement container
```

This is the immutable infrastructure mindset.

---

## 18. Image Versioning

Avoid controlled production deployments that rely only on:

```text
mern-api:latest
```

Prefer:

```text
mern-api:1.4.2
```

or a commit-based tag:

```text
mern-api:7f81a2c
```

A digest provides an even stronger exact-content reference:

```text
registry.example.com/team/mern-api@sha256:<digest>
```

Traceability should connect:

```text
Git commit
    |
    v
7f81a2c
    |
    v
mern-api:7f81a2c
    |
    v
Deployment record
```

Production should be able to answer which source revision and image content are running.

---

## 19. Production MERN Architecture

Putting the ideas together:

```text
                         Internet
                            |
                            v
                    +---------------+
                    |     Nginx     |
                    |    Frontend   |
                    +-------+-------+
                            |
                            v
                    +---------------+
                    |  Node/Express |
                    |      API      |
                    |  non-root     |
                    |  health check |
                    | graceful stop |
                    +-------+-------+
                            |
                    Private Docker network
                            |
                            v
                    +---------------+
                    |    MongoDB    |
                    +-------+-------+
                            |
                            v
                         Volume
```

The application image should be:

```text
Small
  +
Reproducible
  +
Versioned
  +
Non-root
  +
No secrets
  +
Health-aware
  +
Observable
```

---

## 20. Hands-On Lab

Use the Day 30 MERN project.

### Task 1: Optimize the backend Dockerfile

Confirm that:

- package files are copied first
- `npm ci` is used with a lockfile
- production dependencies are selected
- source is copied afterward
- the application runs as non-root
- the `CMD` is explicit

### Task 2: Add `.dockerignore`

Use appropriate entries:

```text
node_modules
.git
.env
coverage
npm-debug.log
```

### Task 3: Add the health endpoint

Implement:

```text
GET /health
```

Return HTTP 200 and:

```json
{
  "status": "ok"
}
```

Test it:

```bash
curl http://localhost:3000/health
```

### Task 4: Verify the user

```bash
docker exec -it <api-container> whoami
```

### Task 5: Inspect the image

```bash
docker images
docker history <image>
```

Look for unnecessarily large layers and accidental build artifacts.

### Task 6: Monitor the container

```bash
docker stats
```

Observe:

```text
CPU %
MEM USAGE / LIMIT
MEM %
```

---

## 21. Graceful Shutdown Challenge

Your production Node container is `Up`, but requests randomly fail during deployments. You discover that Docker sends `SIGTERM` and the Node process exits while requests are still being processed.

The missing behavior is graceful shutdown handling.

The desired sequence is:

```text
SIGTERM
   |
   v
Stop accepting new work
   |
   v
Finish current requests
   |
   v
Close MongoDB connection
   |
   v
Close HTTP server
   |
   v
Exit cleanly
```

Test the behavior with:

```bash
docker stop <api-container>
docker logs <api-container>
```

You should be able to explain every step between `docker stop` and process exit.

---

## 22. Interview Preparation

### Beginner

#### Q1. What makes a Docker image production-ready?

It should be reproducible, appropriately small, secure, versioned, limited to required runtime dependencies, free of baked secrets, and easy to operate and troubleshoot.

#### Q2. Why use `.dockerignore`?

To prevent unnecessary, local, and sensitive files from entering the Docker build context and potentially the image.

### Intermediate

#### Q3. Why externalize application configuration?

The same immutable image can be promoted across development, staging, and production without rebuilding it for each environment.

#### Q4. Why run Node as a non-root user?

To follow least privilege and reduce the impact of a compromised application process.

#### Q5. What is the difference between a running and healthy container?

Running means the container's main process exists. Healthy means that a defined application health condition passes.

### Advanced

#### Q6. What happens when a container receives `SIGTERM`?

The runtime requests graceful termination. The application should stop accepting new work, finish in-flight work where possible, release resources, close connections, and exit cleanly.

#### Q7. Why is manually modifying a production container bad practice?

The changes are not represented in source or the image, disappear on recreation, and cannot be reliably reproduced. Modify the build definition, create a new image, test it, and redeploy.

#### Q8. How would you make a MERN Docker deployment reproducible?

Use Dockerfiles, lockfiles, versioned images, external configuration, immutable artifacts, persistent storage, health checks, and documented Compose or deployment configuration.

---

## 23. Review Questions From Earlier Lessons

### Docker networking

Why does this normally fail when MongoDB is another container?

```text
mongodb://localhost:27017/mern
```

Because `localhost` refers to the Node container. Use the MongoDB service name.

### Docker volumes

What makes MongoDB data persistent?

Mount a Docker volume at:

```text
/data/db
```

### Docker Compose

Does this guarantee readiness?

```yaml
depends_on:
  - mongodb
```

No. It expresses a dependency relationship or startup ordering. Readiness requires health checks and application-level retry behavior.

### Docker registry

Why is this better than `latest` for controlled deployments?

```text
mern-api:abc123
```

It identifies a specific, traceable image rather than a mutable tag.

### Docker monitoring

What command provides live CPU and memory usage?

```bash
docker stats
```

---

## 24. Quiz

### 1. Which Dockerfile order generally provides better cache reuse?

A.

```dockerfile
COPY . .
RUN npm ci
```

B.

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

C.

```dockerfile
RUN npm ci
COPY . .
```

D. It makes no difference.

**Answer: B.** Stable dependency inputs are placed before frequently changing source.

### 2. What is the main security benefit of `USER node`?

A. Faster networking  
B. Less memory usage  
C. Reduced privileges  
D. Automatic encryption

**Answer: C.** The application does not run with root privileges.

### 3. Where should production secrets ideally come from?

A. Dockerfile  
B. Git repository  
C. Secret-management mechanism  
D. Image tag

**Answer: C.** Secrets should be injected through protected runtime configuration.

### 4. What does a health check provide beyond `Container: Up`?

A. It verifies an application health condition  
B. It increases CPU  
C. It creates a volume  
D. It pushes an image

**Answer: A.** Process state and application health are different signals.

### 5. Why are multi-stage builds useful?

A. They eliminate networking  
B. They separate build dependencies from the final runtime image  
C. They automatically deploy to a cloud provider  
D. They replace Git

**Answer: B.** Only the required runtime output needs to enter the final stage.

### 6. What is the purpose of handling `SIGTERM` in Node.js?

A. Increase MongoDB storage  
B. Graceful shutdown  
C. Build Docker images  
D. Change the image tag

**Answer: B.** The application can close connections and finish work cleanly.

### 7. Why should you not manually install packages inside a production container?

A. Containers do not support packages  
B. The change is not reproducible and disappears when the container is recreated  
C. Docker automatically deletes them  
D. It changes the DNS server

**Answer: B.** The Dockerfile and image should represent the complete runtime state.

### 8. Which is more appropriate for a production artifact?

A. `latest` only  
B. `random`  
C. A specific version or immutable digest  
D. `test`

**Answer: C.** It provides predictable identity and rollback.

### 9. What does build, test, and deploy without rebuilding between environments mean?

A. Mutable infrastructure  
B. Build once and promote the tested artifact  
C. Manual deployment  
D. Container debugging

**Answer: B.** The same image is tested and promoted.

### 10. Your Node container is running, but `/health` returns failure. What does this demonstrate?

A. Container state and application health are different concepts  
B. Docker networking is always broken  
C. The image must be deleted  
D. Volumes are unavailable

**Answer: A.** A live process can still be functionally unhealthy.

### Answer Key

```text
1 -> B
2 -> C
3 -> C
4 -> A
5 -> B
6 -> B
7 -> B
8 -> C
9 -> B
10 -> A
```

### Score guide

- 9-10: Production-ready understanding
- 7-8: Good; review weak areas
- 5-6: Needs revision
- Below 5: Revisit Docker fundamentals and security

---

## 25. Coding Exercise: Graceful Shutdown

Improve the backend shutdown behavior with a `SIGTERM` handler.

The handler should:

1. Log that shutdown started.
2. Stop accepting new HTTP requests.
3. Finish active requests where possible.
4. Close the MongoDB connection.
5. Close the HTTP server.
6. Exit successfully or report a controlled shutdown error.

Then test it:

```bash
docker stop <api-container>
docker logs <api-container>
```

Explain the sequence from `docker stop` to the Node process exiting.

A production implementation should also consider:

- a shutdown timeout
- repeated signals
- errors while closing dependencies
- active connection draining
- readiness removal before termination

---

## 26. Mini Assignment: Production Review

Review the Day 30 MERN project and mark each item:

```text
[ ] Images are appropriately small
[ ] Dependencies are cached effectively
[ ] .dockerignore exists
[ ] Production dependencies only
[ ] Containers do not unnecessarily run as root
[ ] No secrets are inside images
[ ] Configuration is externalized
[ ] MongoDB uses persistent storage
[ ] MongoDB is not unnecessarily exposed
[ ] API has a health endpoint
[ ] API handles graceful shutdown
[ ] Logs are accessible
[ ] Resource usage can be monitored
[ ] Images have traceable tags
[ ] Running containers are not manually modified
```

The goal is not to check every box immediately. Identify what still needs improvement and document the evidence.

---

## 27. Docker Phase Milestone

Your Docker knowledge now covers:

```text
Containers
    |
    v
Images
    |
    v
Dockerfiles
    |
    v
Layers and caching
    |
    v
Compose
    |
    v
Networking
    |
    v
Volumes
    |
    v
Multi-stage builds
    |
    v
Image optimization
    |
    v
Security
    |
    v
Registries
    |
    v
Logging
    |
    v
Monitoring
    |
    v
Troubleshooting
    |
    v
Production best practices
    |
    v
MERN production project
```

The progression is:

> I know Docker commands.

becoming:

> I can design and operate a containerized production application.

---
