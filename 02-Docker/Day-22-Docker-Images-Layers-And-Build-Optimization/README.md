# DevOps Mentorship Program — Day 22

## 🐳 Phase 2: Docker & Containers

### 🟢 Topic: Docker Images, Layers & Build Optimization



## 📊 Progress Tracker

### ✅ Linux Phase Complete
- Days 1–19 — Linux Foundation
- Day 20 — Linux Phase Revision & Assessment

### 🐳 Docker Phase
- Day 21 — Docker Fundamentals
- Day 22 — Images, Layers & Build Optimization

**Docker Phase:** 2 / ~10

---

## 🔁 1. Quick Review

### Q1. What is the difference between an image and a container?

```text
Image     → packaged artifact
Container → running instance of an image
```

An image is a read-only package containing the application, runtime, libraries, and filesystem content. A container is a running process created from that image.

### Q2. What does this do?

```bash
docker run -p 8080:3000 myapp
```

It maps the host port `8080` to port `3000` inside the container:

```text
Host :8080
   ↓
Container :3000
```

Users connect to `localhost:8080`, while the application inside the container listens on port `3000`.

### Q3. Does `EXPOSE 3000` publish port 3000 to the host?

No. `EXPOSE` documents the port the container expects to use. Publishing requires a mapping such as:

```bash
docker run -p 3000:3000 myapp
```

### Q4. Which command shows container logs?

```bash
docker logs <container>
```

### Q5. Why should a Node.js application inside a container commonly listen on `0.0.0.0` rather than only `127.0.0.1`?

`127.0.0.1` refers to the container's loopback interface. Traffic arriving through the container network may use another interface, so the application should listen on `0.0.0.0` when it needs to accept connections from outside the container.

---

## 1. 📚 Why Docker Layers Matter

Last lesson you built:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

Docker does not treat this as one giant operation. Each instruction can produce a filesystem layer or affect the resulting image configuration.

A useful conceptual model is:

```text
Layer 1 → Base Node image
Layer 2 → Working/application filesystem changes
Layer 3 → Package manifests
Layer 4 → Installed dependencies
Layer 5 → Application source
```

The exact internal representation is more nuanced, but this model helps explain build caching.

---

## 2. 🎯 Why Layers Matter in Production

Imagine your application has `500 MB` of dependencies and you change only `server.js`.

A poorly structured Dockerfile may reinstall dependencies even though they have not changed. A well-structured Dockerfile lets Docker reuse the dependency layer:

```text
First build:
  npm ci → expensive

Second build:
  source changed
       ↓
  reuse dependency layer
       ↓
  copy new source
       ↓
  much faster
```

This matters in CI/CD because faster builds reduce feedback time, compute cost, and deployment delays.

---

## 3. 🧠 Docker Build Cache

Consider:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

CMD ["node", "server.js"]
```

If only `server.js` changes, Docker can generally reuse:

- the base image layer
- the working directory step
- the package manifest copy
- the dependency installation layer

It then rebuilds the later source-copy layer.

If `package.json` or `package-lock.json` changes, the manifest layer changes and the dependency installation layer must normally be rebuilt. That is the desired behavior because the dependency graph may have changed.

Cache reuse depends on the instruction, its inputs, the build context, and the builder. Never assume a layer is cached without checking the build output.

---

## 4. ❌ A Poorly Ordered Dockerfile

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY . .

RUN npm ci

CMD ["node", "server.js"]
```

Here, the complete source tree is copied before dependencies are installed. A source-code change can invalidate the layer before `RUN npm ci`, making the dependency installation run again.

### Better ordering

```dockerfile
COPY package*.json ./

RUN npm ci

COPY . .
```

This separates infrequently changing dependency inputs from frequently changing application source. It is one of the most common Docker interview questions.

---

## 5. 📦 `.dockerignore`

When you run:

```bash
docker build .
```

Docker uses the current directory as its build context. Files in that context may be sent to the builder and may become available to `COPY` instructions.

Create a `.dockerignore` file:

```text
node_modules
.git
.gitignore
.env
npm-debug.log
coverage
Dockerfile
README.md
```

Depending on your application, entries such as `dist` may need to be adjusted. A frontend build may intentionally need to copy an existing `dist` directory, while a multi-stage build usually creates `dist` inside the builder stage.

### Why it matters

A good `.dockerignore`:

- reduces the build context
- speeds up builds
- prevents local dependencies from being copied
- avoids test and coverage artifacts
- reduces accidental secret inclusion
- makes builds more predictable

---

## 6. 🔐 Why `.env` Belongs in `.dockerignore`

Suppose a local `.env` contains:

```text
MONGO_URI=...
JWT_SECRET=...
```

You do not want `COPY . .` to accidentally place these secrets inside the image.

```text
.dockerignore
       │
       └── .env
```

However, `.dockerignore` is not a complete secrets-management solution. Production secrets should be supplied at runtime through environment configuration, secret stores, orchestrator secrets, or another approved mechanism. They should not be baked into image layers, Dockerfile instructions, source control, or build logs.

---

## 7. 📏 Image Size

Check local image sizes:

```bash
docker images
```

Example:

```text
REPOSITORY   TAG   SIZE
mern-api     1.0   950MB
```

A large image may be caused by:

- a full, unnecessary base image
- development dependencies
- build tools included in the runtime image
- files missing from `.dockerignore`
- test and coverage artifacts
- unnecessary source files
- an oversized `node_modules` directory

Image size is important because it affects registry storage, pull time, startup time, deployment speed, and vulnerability scanning. Size alone is not the only quality measure: compatibility, security, reproducibility, and build speed also matter.

---

## 8. 🪶 Alpine Images

Earlier we used:

```dockerfile
FROM node:22-alpine
```

Alpine variants are commonly smaller than full Debian or Ubuntu-based Node images.

However, smaller is not automatically better. Alpine uses `musl` libc rather than `glibc`, and some native Node dependencies may behave differently or require extra packages.

Choose a base image based on:

- compatibility
- security support
- required native libraries
- image size
- build time
- runtime behavior

Test the actual application instead of choosing an image only because it is small.

---

## 9. 🏗️ Multi-Stage Builds

Multi-stage builds use separate stages for building and running an application:

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

The build stage may contain source code, development dependencies, and build tools. The runtime stage receives only the generated frontend assets.

```text
Build stage
    │
    ├── Source
    ├── Dependencies
    └── Build tools
          │
          ▼
       /dist
          │
          ▼
Runtime stage
    │
    └── Only production assets
```

The build environment does not have to become the runtime environment. This can significantly reduce the final image size and attack surface.

---

## 10. 🌍 MERN Example — React Frontend

A React application generally does not need its entire Node development environment at runtime:

```text
React source
    ↓
npm install
    ↓
npm run build
    ↓
static files
    ↓
Nginx
```

A multi-stage Docker build is a natural fit. Node and frontend build tools remain in the build stage, while Nginx serves only the generated static files.

---

## 11. 🟢 MERN Backend Example

For a Node.js API, the runtime image should contain only what the application needs to execute.

```dockerfile
FROM node:22-alpine AS deps

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

FROM node:22-alpine

WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY package*.json ./
COPY server.js ./

EXPOSE 3000

USER node

CMD ["node", "server.js"]
```

`USER node` connects directly to the Linux least-privilege principle. The application does not need to run as root.

For a real application, copy every runtime file required by the API, not only `server.js`. This may include route modules, configuration templates, compiled code, and package metadata.

---

## 12. 🔐 Docker Security Connection

Your Linux security knowledge transfers directly into containers.

```text
Bad:
Container
   ↓
root
   ↓
Application

Better where practical:
Container
   ↓
non-root user
   ↓
Node.js
```

Other important controls include:

- use a minimal suitable base image
- remove unnecessary packages
- never bake secrets into images
- use a read-only filesystem where practical
- grant minimal Linux capabilities
- update base images regularly
- scan images for vulnerabilities
- pin and review dependencies
- run health and security tests

Container isolation is not a replacement for application security or host security.

---

## 13. 🔍 Inspect Image History

Inspect the image layers and commands used to construct an image:

```bash
docker history mern-api:1.0
```

This helps identify unexpectedly large layers and understand how the image was built:

```text
IMAGE
↓
RUN npm ci
↓
COPY source
↓
CMD
```

Do not put secrets in Dockerfile commands. Image history can expose command arguments and values from build instructions.

---

## 14. 🧪 Hands-on Lab

Go back to your Day 21 application:

```bash
cd ~/devops-course/day21/node-api
```

Create `.dockerignore`:

```text
node_modules
.git
.env
npm-debug.log
coverage
```

Build:

```bash
docker build -t mern-api:1.1 .
```

Inspect the image:

```bash
docker images
```

Inspect its history:

```bash
docker history mern-api:1.1
```

Run it:

```bash
docker run -d \
  --name mern-api-v11 \
  -p 3001:3000 \
  mern-api:1.1
```

Test it:

```bash
curl http://localhost:3001/health
```

The mapping is:

```text
Host :3001
     ↓
Container :3000
```

Useful cleanup commands:

```bash
docker logs mern-api-v11
docker stop mern-api-v11
docker rm mern-api-v11
```

---

## 15. 🔄 Build Cache Experiment

Change only `server.js` and rebuild:

```bash
docker build -t mern-api:1.2 .
```

Watch the build output. Docker should reuse layers before the changed source layer, especially the dependency installation layer.

Now change `package.json` and rebuild:

```bash
docker build -t mern-api:1.3 .
```

The dependency-related layer should need rebuilding because the package manifest changed.

This experiment demonstrates why Dockerfile instruction order affects CI/CD build speed.

---

## 16. 🛠️ Production Build Optimization Checklist

For a Node/MERN Docker image, think through this sequence:

```text
1. Small appropriate base
       ↓
2. .dockerignore
       ↓
3. Dependency files first
       ↓
4. Dependency installation
       ↓
5. Source later
       ↓
6. Multi-stage build where useful
       ↓
7. Production dependencies only
       ↓
8. Non-root runtime
       ↓
9. No secrets
       ↓
10. Scan and test
```

Every optimization should preserve application behavior and reproducibility.

---

## 17. 💼 Interview Preparation

### Beginner

**Q1. What is a Docker layer?**

A Docker layer represents part of the filesystem or image configuration produced during an image build. Docker can reuse unchanged build layers through its cache.

**Q2. Why use `.dockerignore`?**

To keep unnecessary, sensitive, and local development files out of the build context and image.

**Q3. Why copy `package*.json` before application source?**

So the dependency-installation layer can remain cached when only application source changes.

### Intermediate

**Q4. How do you reduce Docker image size?**

Use an appropriate minimal base image, a `.dockerignore` file, production dependencies only, multi-stage builds, and a runtime stage without unnecessary tools. Also mention compatibility and security rather than focusing on size alone.

**Q5. What is a multi-stage build?**

A Dockerfile with multiple build stages where artifacts from an earlier stage are selectively copied into a separate runtime stage. This keeps unnecessary build dependencies out of the final image.

### Advanced Scenario

**Question:** Your CI pipeline takes 12 minutes to build a Node.js Docker image. The application code is only a few MB. What would you investigate?

```text
Inspect Dockerfile
       ↓
Check layer ordering
       ↓
Check npm install/ci
       ↓
Check build context
       ↓
Check .dockerignore
       ↓
Check cache effectiveness
       ↓
Check base-image pull time
       ↓
Check multi-stage opportunities
```

If `package.json` rarely changes but source code changes frequently, dependency installation should be placed in a layer that can be reused.

---

## 18. 📝 Quiz

1. Why are Docker layers useful?
   - A. They automatically encrypt containers
   - B. They enable reuse and caching of unchanged build steps
   - C. They replace Linux permissions
   - D. They create DNS records

2. What file controls which files are excluded from Docker build context?
   - A. `.dockerignore`
   - B. `.dockerexclude`
   - C. `.dockerconfig`
   - D. `.ignore`

3. Why should `package*.json` usually be copied before source?
   - A. To expose port 3000
   - B. To improve dependency-layer caching
   - C. To start Node
   - D. To enable SSH

4. What command shows image history?
   - A. `docker layers`
   - B. `docker history`
   - C. `docker cache`
   - D. `docker inspect-history`

5. What is the main purpose of a multi-stage build?
   - A. Run two databases
   - B. Keep build-time dependencies out of the final runtime image
   - C. Automatically scale containers
   - D. Replace Kubernetes

6. Should production secrets be baked into an image?
   - A. Yes
   - B. No

7. Which is generally preferable for a production application?
   - A. Run as root whenever possible
   - B. Run as a non-root user where practical
   - C. Disable all permissions
   - D. Use `chmod 777`

8. If `server.js` changes but `package.json` does not, what should ideally happen?
   - A. Dependencies needlessly reinstall
   - B. Docker can reuse the dependency layer
   - C. The base image must rebuild
   - D. MongoDB must restart

9. Which command builds an image?
   - A. `docker compile`
   - B. `docker build`
   - C. `docker package`
   - D. `docker make`

10. What is a major benefit of a multi-stage React build?
    - A. It lets the final image omit unnecessary build tooling
    - B. It makes MongoDB faster
    - C. It removes HTTP
    - D. It replaces Nginx automatically

### ✅ Answer Key

1. B
2. A
3. B
4. B
5. B
6. B
7. B
8. B
9. B
10. A

---

## 19. 🧪 Practical Assessment

### Scenario

Your current Dockerfile is:

```dockerfile
FROM node:22

WORKDIR /app

COPY . .

RUN npm install

EXPOSE 3000

CMD ["node", "server.js"]
```

The image is `1.2 GB`, builds take `10 minutes`, and developers frequently change `server.js`.

### Your Task

Identify at least five improvements.

### Strong Answer

```text
Use a suitable smaller base image
       ↓
Add .dockerignore
       ↓
Copy package files first
       ↓
Use npm ci where appropriate
       ↓
Use production dependencies only
       ↓
Consider multi-stage builds
       ↓
Run as non-root
       ↓
Avoid copying unnecessary artifacts
```

### Detailed Explanation

1. **Use a suitable base image:** A compatible Alpine or slim image may reduce pull and storage time.
2. **Add `.dockerignore`:** Exclude `node_modules`, `.git`, `.env`, coverage, logs, and other unnecessary files.
3. **Copy dependency manifests first:** This lets Docker cache dependency installation when only source changes.
4. **Use `npm ci`:** It uses the lockfile and provides reproducible installation when a lockfile exists.
5. **Install production dependencies only:** Use `npm ci --omit=dev` for a runtime image when development dependencies are unnecessary.
6. **Use multi-stage builds:** Keep compilers, test tools, and frontend build tools out of the runtime image.
7. **Run as non-root:** Reduce the impact of an application-level compromise.
8. **Avoid unnecessary files:** Copy only what the application needs at runtime.
9. **Measure the result:** Compare `docker images`, build durations, and `docker history` before and after.
10. **Test compatibility and security:** A smaller image is not successful if the app fails or required security updates are missing.

---

## 20. 🏗️ Month 2 Cumulative Project

### Containerized Production MERN Application

Your Month 2 project now requires:

### Backend

- Dockerfile
- `.dockerignore`
- optimized dependency layer
- non-root runtime
- health endpoint

### Frontend

Use a multi-stage build:

```text
Node build stage
       ↓
Production static assets
       ↓
Nginx runtime stage
```

### Image Quality

Document:

- image size
- build time
- layer structure
- base image choice
- security considerations

### Required Commands

Demonstrate:

```bash
docker build
docker images
docker history
docker run
docker ps
docker logs
docker inspect
docker exec
```

---

## 21. 🎤 Final Interview Challenge

Answer aloud:

> How would you optimize a Dockerfile for a production Node.js application?

Your answer should include:

```text
Appropriate base image
        ↓
.dockerignore
        ↓
Copy dependency manifests
        ↓
Install dependencies
        ↓
Copy source
        ↓
Use build cache
        ↓
Production dependencies
        ↓
Multi-stage build where appropriate
        ↓
Non-root user
        ↓
No baked-in secrets
        ↓
Image scanning/testing
```

The key principle is:

> A production image should contain what the application needs at runtime—not everything that happened to be present during development.

---

## 📖 22. Today's Key Takeaways

You learned:

- ✅ Docker images are built from layers.
- ✅ Layers enable build caching.
- ✅ Dockerfile instruction order matters.
- ✅ `.dockerignore` reduces build context and accidental inclusion.
- ✅ Dependency manifests should usually be copied before source.
- ✅ Multi-stage builds separate build and runtime environments.
- ✅ Production images should avoid unnecessary dependencies.
- ✅ Containers should run as non-root where practical.
- ✅ Secrets should not be baked into images.
- ✅ Docker optimization includes build speed, image size, compatibility, security, and reproducibility.

Your mental model is now:

```text
Dockerfile
    ↓
Build context
    ↓
Layers
    ↓
Cache
    ↓
Image
    ↓
Container
    ↓
Production application
```

---

## ⏭️ Next Lesson — Day 23

### 🐳 Docker Networking & Persistent Storage

We'll connect containers to the MERN architecture:

```text
Nginx
  ↓
Node API
  ↓
MongoDB
```

You'll learn:

- container networks
- container-to-container communication
- host vs container networking
- why `localhost` causes Docker mistakes
- volumes
- persistent MongoDB data
- MERN Docker networking failures
- production examples
- Docker networking and persistence interview questions
