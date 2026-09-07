# DevOps Mentorship Program - Day 30

## Phase 2: Docker & Containers

### Monthly Cumulative Real-World Docker Project & Assessment

**Type:** Monthly cumulative project and practical assessment  
**Estimated time:** 25-30 minutes  
**Difficulty:** Intermediate  
**Project:** Production-style Dockerized MERN Application

Today is not a new topic. This checkpoint combines the Docker skills learned so far into one reproducible, secure, persistent, observable, and deployable MERN stack.

---

## 1. Learning Objectives

By the end of this assessment, you should be able to:

- Containerize a MERN application.
- Design Docker networking correctly.
- Persist MongoDB data.
- Use Docker Compose to operate the stack.
- Build optimized Docker images.
- Apply basic container security.
- Tag and push images to a registry.
- Troubleshoot failed containers.
- Inspect logs and resource usage.
- Explain the architecture in a technical interview.

The central goal is to think like a DevOps engineer rather than only memorizing Docker commands.

---

## 2. Real-World Scenario

Imagine joining a company that has this application:

```text
                    Internet
                       |
                       v
                   React app
                       |
                       v
                Node/Express API
                       |
                       v
                    MongoDB
```

The development team currently starts each component manually. Your job is to Dockerize the entire application so that it is:

- reproducible
- secure
- networked correctly
- persistent
- observable
- versioned
- deployable to another environment

---

## 3. Target Architecture

Your final architecture should look approximately like this:

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
                      |  Node/Express  |
                      |       API      |
                      +-------+-------+
                              |
                       Docker network
                              |
                              v
                      +---------------+
                      |    MongoDB     |
                      +-------+-------+
                              |
                              v
                       Docker volume
                         mongo-data
```

### Critical networking rule

From inside the API container:

```text
API -> mongodb:27017
```

Not:

```text
API -> localhost:27017
```

Inside a container, `localhost` means that same container. The API must use the MongoDB service name provided by Docker's internal DNS.

---

## 4. Expected Project Structure

A reasonable project structure is:

```text
mern-docker/
|
+-- frontend/
|   +-- package.json
|   +-- src/
|   +-- Dockerfile
|
+-- backend/
|   +-- package.json
|   +-- src/
|   +-- tests/
|   +-- Dockerfile
|
+-- nginx/
|   +-- nginx.conf
|
+-- compose.yaml
+-- .dockerignore
+-- .gitignore
+-- .env.example
+-- README.md
```

Do not commit a real `.env` file containing credentials. Commit an `.env.example` with placeholder names and safe example values instead.

Each directory has a clear responsibility:

- `frontend/`: React source and frontend image definition
- `backend/`: Node/Express API, tests, and backend image definition
- `nginx/`: public reverse-proxy and static-asset configuration
- `compose.yaml`: local and assessment stack topology
- `.dockerignore`: files excluded from Docker build context
- `.gitignore`: files excluded from source control
- `.env.example`: documented configuration names without real secrets
- `README.md`: architecture, operation, and recovery documentation

---

## 5. Compose Design

The Compose architecture should define the application services, network, and volume:

```yaml
services:
  frontend:
    build: ./frontend

  api:
    build: ./backend

  mongodb:
    image: mongo:8

volumes:
  mongo-data:

networks:
  app-network:
```

A more complete starting point is:

```yaml
services:
  frontend:
    build: ./frontend
    networks:
      - app-network
    depends_on:
      api:
        condition: service_healthy

  api:
    build: ./backend
    environment:
      NODE_ENV: production
      MONGO_URI: mongodb://mongodb:27017/mern
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/health', response => process.exit(response.statusCode === 200 ? 0 : 1)).on('error', () => process.exit(1))"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s

  mongodb:
    image: mongo:8
    networks:
      - app-network
    volumes:
      - mongo-data:/data/db
    healthcheck:
      test: ["CMD-SHELL", "mongosh --quiet --eval 'db.adminCommand(\"ping\").ok' | grep 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

networks:
  app-network:

volumes:
  mongo-data:
```

The exact health-check command must match the tools present in the selected images. Do not copy a check that requires a missing binary.

### Service responsibilities

- `frontend`: serves or builds the React application.
- `api`: serves HTTP requests and connects to MongoDB through `mongodb:27017`.
- `mongodb`: stores application data and mounts the persistent volume.
- `nginx`: optionally serves as the public entry point, terminates TLS, serves static files, and proxies API requests.

---

## 6. Networking Requirements

### Publicly exposed

Normally expose only the intended web entry point:

```text
Internet -> Nginx -> private Docker network
```

For development, you may publish a frontend or API port to the host for testing. In production, the API can remain internal behind Nginx.

### Internal services

The API and MongoDB should communicate privately on `app-network`:

```text
Nginx
  |
  v
API:3000
  |
  v
MongoDB:27017
```

Avoid this unless there is a specific, controlled requirement:

```yaml
mongodb:
  ports:
    - "27017:27017"
```

MongoDB does not need a host port mapping for the API to reach it internally. Publishing the port increases exposure and attack surface.

### Verify networking

```bash
docker compose config
docker network ls
docker network inspect <project>_app-network
docker compose exec api getent hosts mongodb
```

The exact network name may include the Compose project name. Confirm that the API and MongoDB are attached to the same network and that `mongodb` resolves to an address.

---

## 7. Persistence Requirement

MongoDB must survive container recreation. Add:

```yaml
volumes:
  mongo-data:

services:
  mongodb:
    volumes:
      - mongo-data:/data/db
```

The mental model is:

```text
Container = replaceable
Volume    = persistent
Backup    = independent recovery mechanism
```

### Persistence test

Start the stack:

```bash
docker compose up -d
```

Check status:

```bash
docker compose ps
```

Create or record a test document. Remove only the MongoDB container:

```bash
docker compose rm -sf mongodb
```

Recreate it:

```bash
docker compose up -d mongodb
```

Confirm that the test data still exists.

This demonstrates container persistence, but it does not demonstrate disaster recovery. Also document a separate MongoDB backup and restore process.

---

## 8. Image Optimization

A backend Dockerfile should separate stable dependency inputs from frequently changing source code:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY . .

USER node

EXPOSE 3000

CMD ["node", "src/server.js"]
```

Why this order matters:

1. The base image provides the runtime.
2. `WORKDIR` makes paths predictable.
3. Copying manifests first allows dependency-layer caching.
4. `npm ci --omit=dev` installs lockfile-based production dependencies only.
5. Source is copied after dependencies, so source changes do not always reinstall dependencies.
6. `USER node` applies least privilege.
7. `EXPOSE` documents the container port; it does not publish the port.
8. `CMD` defines the default runtime process.

For an application with a build step, use a multi-stage build:

```dockerfile
FROM node:22-alpine AS build

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

The final stage should contain only the runtime assets and runtime tools it needs.

---

## 9. `.dockerignore` and Git Safety

Create `.dockerignore`:

```text
node_modules
.git
.env
npm-debug.log
coverage
Dockerfile
compose.yaml
```

The exact entries may vary by project. Do not exclude files required by a build stage.

Why it matters:

- reduces build context size
- prevents local dependencies from being copied
- avoids test and coverage artifacts
- reduces accidental secret inclusion
- makes builds faster and more predictable

Also create `.gitignore`:

```text
.env
node_modules/
coverage/
*.log
```

`.dockerignore` is not a complete secrets-management system. Production secrets must be injected securely at runtime.

---

## 10. Security Requirements

### Run as non-root

Use a non-root runtime user where practical:

```bash
docker exec -it <container> whoami
```

The expected result should not be `root` for the application process.

### Do not bake secrets into images

Never use:

```dockerfile
ENV MONGO_PASSWORD=my-secret-password
```

Do not copy `.env` into the image or commit it to Git. Use protected runtime configuration, Compose secrets for suitable local scenarios, or a platform secret manager.

### Use suitable minimal images

A compatible `node:22-alpine` image may reduce image size, but compatibility and security updates matter more than size alone. Test native dependencies before standardizing on Alpine.

### Limit privileges

Consider:

```bash
docker run \
  --cap-drop=ALL \
  --read-only \
  --tmpfs /tmp \
  myapp
```

Do not apply these restrictions blindly. First determine what the application needs to write and which capabilities it needs, then test the hardened image.

### Limit resources

Use observed workload to choose limits:

```bash
docker run \
  --memory=512m \
  --cpus=1 \
  myapp
```

Resource limits reduce the blast radius of a memory leak, runaway process, or compromised container.

---

## 11. Health Checks

A running container is not necessarily a healthy application:

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

A health check should be:

- fast
- repeatable
- safe to call repeatedly
- free of side effects
- based on a meaningful condition

Consider separate liveness and readiness behavior in larger deployments:

- liveness asks whether the process should continue running
- readiness asks whether it can currently serve useful traffic

If MongoDB is temporarily unavailable, the API may be alive but not ready for traffic.

---

## 12. Build and Test the Stack

Build and start:

```bash
docker compose up -d --build
```

Check status:

```bash
docker compose ps
```

View all logs:

```bash
docker compose logs
```

View API logs:

```bash
docker compose logs api
docker compose logs -f api
```

Test the API:

```bash
curl http://localhost:3000/health
```

If Nginx is the public entry point, also test the public route and bypass Nginx where appropriate to isolate reverse-proxy failures.

---

## 13. Production Troubleshooting Challenge

Suppose:

```text
frontend  Up
api       Up
mongodb   Up
```

but the frontend reports:

```text
Failed to fetch API
```

Use a structured investigation.

### Step 1: API logs

```bash
docker compose logs --tail 100 api
```

Look for startup errors, database errors, unhandled exceptions, and configuration problems.

### Step 2: Network

```bash
docker network ls
docker network inspect <network>
```

Confirm the API and MongoDB share the intended network.

### Step 3: DNS from the API

```bash
docker compose exec api getent hosts mongodb
```

A successful result confirms that the service name resolves inside the API container.

### Step 4: MongoDB connection string

Verify:

```text
mongodb://mongodb:27017/mern
```

not:

```text
mongodb://localhost:27017/mern
```

### Step 5: Runtime configuration

```bash
docker inspect <api-container>
docker compose exec api printenv MONGO_URI
```

Check environment variables, image, mounts, network attachments, and ports. Avoid sharing inspection output if it contains secrets.

---

## 14. Resource Troubleshooting

If the Node container is killed:

```bash
docker ps -a
docker inspect <container>
docker stats --no-stream
```

On a Linux host:

```bash
free -h
df -h
```

Exit code `137` commonly means that a process received `SIGKILL`, possibly due to memory pressure. Do not automatically conclude that it was an OOM kill. Confirm with container limits, memory metrics, host evidence, and logs.

Investigation chain:

```text
Container stopped
      |
      v
Exit code and logs
      |
      v
Resource metrics
      |
      v
Container and host limits
      |
      v
Application behavior
```

---

## 15. Logging Assessment

Use:

```bash
docker logs <container>
docker logs --tail 100 <container>
docker logs -f <container>
docker logs -t <container>
docker compose logs -f api
```

Production mindset:

```text
Do not only restart
        |
        v
Ask why it failed
        |
        v
Collect evidence
        |
        v
Fix the cause
        |
        v
Verify recovery
```

Applications should normally write structured logs to stdout and stderr when the runtime platform collects those streams. Avoid logging secrets or unmanaged files that can fill a container's writable layer.

---

## 16. Registry Assessment

Build the API image with a traceable identifier:

```bash
docker build -t mern-api:abc123 ./backend
```

Tag it for a registry:

```bash
docker tag \
  mern-api:abc123 \
  registry.example.com/team/mern-api:abc123
```

Push it:

```bash
docker push registry.example.com/team/mern-api:abc123
```

Another environment can pull it:

```bash
docker pull registry.example.com/team/mern-api:abc123
```

Prefer:

```text
mern-api:abc123
```

or an immutable digest. Avoid relying on:

```text
mern-api:latest
```

for controlled production releases.

The artifact tested in staging should be the artifact deployed in production.

---

## 17. Monthly Project Challenge

### Project: Production-Ready MERN Docker Stack

Build:

```text
                  Internet
                     |
                     v
                   Nginx
                     |
                     v
               React frontend
                     |
                     v
                  Node API
                     |
                     v
                  MongoDB
                     |
                     v
              Persistent volume
```

### Requirements

#### Docker

- frontend container
- backend container
- MongoDB container
- Docker Compose
- custom network
- persistent volume

#### Networking

- API communicates with MongoDB through the service name
- MongoDB is not unnecessarily exposed
- host and container ports are documented correctly

#### Images

- suitable base image
- `.dockerignore`
- cache-friendly layer ordering
- multi-stage build where appropriate
- production-only dependencies

#### Security

- non-root application user
- no secrets in Dockerfiles or images
- `.env` excluded from Git
- minimal privileges
- minimal suitable base image

#### Operations

- health endpoint
- Compose health check
- accessible logs
- resource monitoring with `docker stats`
- disk-usage inspection
- documented recovery procedure

#### Registry

- version or commit-based image tag
- registry push
- image pull in another environment
- previous known-good image retained for rollback

---

## 18. Interview Scenario

> We have a MERN application running in Docker Compose. The Node API cannot connect to MongoDB, but both containers are running. What would you do?

A strong answer is:

```text
1. Check API logs
2. Check MongoDB logs
3. Verify both containers share a network
4. Verify MongoDB service-name DNS
5. Check MONGO_URI
6. Confirm the URI uses mongodb:27017 rather than localhost
7. Test DNS and connectivity from the API container
8. Inspect runtime configuration
9. Fix the root cause
10. Verify application health
```

Do not begin with an unexplained restart. First collect evidence that distinguishes a configuration error, network error, database readiness problem, authentication problem, or application bug.

---

## 19. Monthly Assessment Quiz

### Q1. Inside a Compose network, what hostname should an API use for a MongoDB service named `mongodb`?

A. `localhost`  
B. `127.0.0.1`  
C. `mongodb`  
D. `host.docker.internal`

**Answer:** C. The service name is resolved by Docker's internal DNS.

### Q2. What is the primary purpose of a MongoDB volume?

A. Image optimization  
B. Persistent data storage  
C. DNS resolution  
D. CPU management

**Answer:** B. It keeps data outside the disposable container layer.

### Q3. What does this mean?

```yaml
ports:
  - "3000:3000"
```

A. Host port 3000 maps to container port 3000  
B. Container port 3000 maps only internally  
C. Two containers use port 3000  
D. MongoDB uses port 3000

**Answer:** A.

### Q4. What is the main purpose of `.dockerignore`?

A. Prevent containers from starting  
B. Reduce unnecessary build context  
C. Encrypt images  
D. Create networks

**Answer:** B.

### Q5. Why copy `package*.json` before the rest of the source?

A. To expose ports  
B. To improve Docker layer-cache reuse  
C. To create a volume  
D. To configure DNS

**Answer:** B.

### Q6. What does `depends_on` not guarantee by itself?

A. That a declared dependency exists  
B. That Compose understands the relationship  
C. Application readiness  
D. Startup metadata

**Answer:** C. Readiness requires health checks and resilient application behavior.

### Q7. Which is preferable for identifying a production image?

A. `latest`  
B. `test`  
C. A specific version, commit identifier, or immutable digest  
D. `new`

**Answer:** C.

### Q8. What is an advantage of running a container as non-root?

A. It makes MongoDB faster  
B. It reduces privileges available to the process  
C. It eliminates all vulnerabilities  
D. It automatically encrypts traffic

**Answer:** B.

### Q9. Which command provides live CPU and memory information?

A. `docker images`  
B. `docker stats`  
C. `docker tag`  
D. `docker history`

**Answer:** B.

### Q10. A container exits with code 137. What is a reasonable first investigation?

A. DNS records  
B. Git branches  
C. Memory pressure and supporting evidence  
D. Image tags

**Answer:** C. Exit code 137 commonly indicates `SIGKILL`, but evidence is required to confirm the cause.

---

## 20. Health-Check Script Exercise

Create `health-check.sh`:

```bash
#!/usr/bin/env bash

set -u

api_status="$(docker inspect --format='{{.State.Status}}' mern-api 2>/dev/null || true)"
mongodb_status="$(docker inspect --format='{{.State.Status}}' mongodb 2>/dev/null || true)"

printf 'Checking API...\n'
if [ "$api_status" = "running" ]; then
  printf 'API: OK\n'
else
  printf 'API: %s\n' "${api_status:-missing}"
  exit 1
fi

printf '\nChecking MongoDB...\n'
if [ "$mongodb_status" = "running" ]; then
  printf 'MongoDB: OK\n'
else
  printf 'MongoDB: %s\n' "${mongodb_status:-missing}"
  exit 1
fi

printf '\nStack status: HEALTHY\n'
```

Make it executable on Linux:

```bash
chmod +x health-check.sh
./health-check.sh
```

This checks process state only. For a stronger production check, extend it to verify the API health endpoint and API-to-MongoDB connectivity.

A useful lesson is:

```text
Container running
       !=
Application dependency path healthy
```

---

## 21. Hands-On Lab

Run the complete stack:

```bash
docker compose up -d --build
```

Verify services:

```bash
docker compose ps
```

Inspect the network:

```bash
docker network ls
docker network inspect <network>
```

Inspect volumes:

```bash
docker volume ls
docker volume inspect <volume>
```

Check logs:

```bash
docker compose logs --tail 100
docker compose logs -f api
```

Monitor resources:

```bash
docker stats
docker system df
```

Test the API:

```bash
curl http://localhost:<api-port>/health
```

### Failure experiment

Temporarily change:

```text
MONGO_URI=mongodb://localhost:27017/mern
```

Recreate the API and observe the failure:

```bash
docker compose up -d --force-recreate api
docker compose logs api
```

Restore:

```text
MONGO_URI=mongodb://mongodb:27017/mern
```

Recreate the API and verify recovery. This reinforces the localhost-in-container rule.

---

## 22. Project Deliverables

Your repository should contain:

```text
mern-docker/
+-- frontend/
+-- backend/
+-- nginx/
+-- compose.yaml
+-- Dockerfiles
+-- .dockerignore
+-- .gitignore
+-- .env.example
+-- README.md
```

The project README must document:

- architecture
- how to run the stack
- environment variables
- ports
- networks
- volumes
- health checks
- troubleshooting commands
- image build workflow
- registry workflow
- security decisions
- rollback process
- backup strategy

This README is part of the assessment and prepares you to explain the project in an interview.

---

## 23. What an Interviewer Should Hear

If asked, "Tell me about a Docker project you have worked on," a strong answer is:

> I containerized a MERN application using separate containers for the frontend, API, and MongoDB. I used a user-defined Docker network so the API communicated with MongoDB through the service name instead of localhost. MongoDB data was persisted through a named volume. I optimized the images using appropriate base images, dependency-layer caching, `.dockerignore`, production dependencies, and multi-stage builds where appropriate. I applied basic container hardening, added health checks, inspected logs and resource usage, and used versioned image tags for registry-based deployment and rollback.

This demonstrates architecture, operations, security, persistence, and deployment thinking rather than only command familiarity.

---

## 24. Linux Skills Still Used in Docker

The Linux foundation remains important:

```bash
df -h       # disk usage
df -i       # inode usage
free -h     # memory
tail -f     # follow a log file
ss -lntp    # listening TCP sockets
top         # process and CPU inspection
```

Docker adds an execution and packaging layer, but troubleshooting still depends on processes, sockets, filesystems, memory, permissions, logs, and networking.

---

## 25. Recommended Documentation

Review official documentation for:

- Docker Compose
- Docker networking
- Docker volumes
- Dockerfiles
- multi-stage builds
- container security
- image tagging and registries
- Docker logging
- Docker resource management
- Docker health checks

Do not try to memorize every option. Focus on understanding why each component exists and what evidence each command provides.

---

## 26. Day 30 Summary

You have now completed a significant Docker milestone:

```text
Linux
  |
  v
Docker fundamentals
  |
  v
Images and Dockerfiles
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
Optimization
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
MERN production Docker project
```

### Core DevOps principles

- Containers should be replaceable.
- Important data should be persistent.
- Services should communicate through the correct network abstraction.
- Images should be reproducible and identifiable.
- Secrets should not live inside images or Git.
- Logs, metrics, and health checks are essential for troubleshooting.
- Restarting a broken container is not the same as fixing the problem.
- The artifact tested should be the artifact deployed.

### Final completion target

Your project is complete when another developer can clone the repository, understand the architecture, start the stack, verify health, inspect logs, test persistence, identify image versions, and follow the documented recovery process.
