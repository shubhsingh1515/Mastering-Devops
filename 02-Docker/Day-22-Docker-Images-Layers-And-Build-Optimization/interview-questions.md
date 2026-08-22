# Day 22 — Interview Questions: Docker Images, Layers & Build Optimization

## Beginner

### 1. What is a Docker layer?

A layer is a filesystem or image-configuration change produced during an image build. Docker can reuse unchanged layers through its build cache.

### 2. Why use `.dockerignore`?

It prevents unnecessary, local, generated, or sensitive files from entering the Docker build context. This can improve build speed and reduce accidental image contents.

### 3. Why copy `package*.json` before the source code?

Dependency manifests normally change less frequently than source code. Copying them first allows Docker to cache the dependency installation layer when only application source changes.

### 4. Does `EXPOSE 3000` publish a port?

No. It documents the intended container port. Publishing requires a runtime mapping such as `-p 3001:3000`.

### 5. What command displays image history?

```bash
docker history image:tag
```

---

## Intermediate

### 6. How do you reduce Docker image size?

Use an appropriate minimal base image, add a `.dockerignore`, install only production dependencies, use multi-stage builds, and avoid copying test output, local dependencies, build tools, and unnecessary source files into the runtime image. Verify compatibility and security rather than optimizing size blindly.

### 7. What is a multi-stage build?

It is a Dockerfile containing multiple stages. One stage builds the application and another stage runs it. Only required artifacts are copied from the build stage into the runtime stage.

### 8. Why is `npm ci` often preferred in Docker builds?

It installs from the lockfile and is designed for clean, reproducible installations. It also fails when the lockfile and package manifest are inconsistent, making dependency problems visible.

### 9. What happens to the cache when `package.json` changes?

The layer that copies the package manifest changes, so Docker normally invalidates the following dependency installation layer. Dependencies need to be installed again because the dependency graph may have changed.

### 10. What happens when only `server.js` changes in a well-ordered Dockerfile?

Docker can usually reuse the base, manifest, and dependency layers, then rebuild the later source-copy layer and subsequent steps.

### 11. Why might Alpine create compatibility problems?

Alpine uses `musl` libc rather than `glibc`. Some native modules or operating-system assumptions may require extra packages or behave differently. The image must be tested with the actual application.

### 12. Why should a production container run as non-root?

If the application is compromised, non-root execution limits what the process can change inside the container and can reduce the impact of an escape or misconfiguration. It is one layer of defense, not a complete security strategy.

---

## Advanced

### 13. Your CI build takes 12 minutes. What do you investigate?

Inspect the Dockerfile order, build context size, `.dockerignore`, dependency installation, cache hit rate, base-image pull time, multi-stage opportunities, registry configuration, and whether package manifests change unnecessarily. Confirm findings using build output, image history, and timing measurements.

### 14. How would you optimize a Node.js backend Dockerfile?

A strong approach is:

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

The exact copied files depend on the application. Explain the dependency cache, production dependencies, final runtime stage, and non-root user.

### 15. Should production secrets be included in `.env` and copied into an image?

No. `.env` should normally be ignored from the build context, but production secrets should also be supplied through runtime configuration or a dedicated secret-management system. Avoid secrets in source control, Dockerfiles, build arguments, image layers, and logs.

### 16. What should you inspect after optimizing an image?

Inspect image size, history, vulnerability scan results, runtime user, exposed ports, application health, logs, dependency behavior, and whether the container is reproducible from a clean build.

### 17. How do multi-stage builds help a React application?

The Node build stage compiles the frontend. The Nginx runtime stage contains only generated static files and the web server, so Node development tools and source dependencies are not included in the final image.

### 18. Is the smallest image always the best image?

No. The image must also be compatible, supported, secure, reproducible, observable, and maintainable. A slightly larger image may be the right choice if it avoids native-library problems or improves operational reliability.

---

## Scenario Practice

### Scenario 1: Every source change reinstalls dependencies

**Likely cause:** The Dockerfile copies the entire source tree before running `npm install` or `npm ci`.

**Improvement:** Copy `package*.json` first, install dependencies, then copy application source.

### Scenario 2: The image contains credentials

**Likely cause:** `.env` was present in the build context and `COPY . .` included it, or a secret was written in a Dockerfile command.

**Improvement:** Remove the secret from the image and history, add `.env` to `.dockerignore`, rotate exposed credentials, and inject secrets at runtime.

### Scenario 3: The application fails after changing to Alpine

**Likely cause:** A native dependency requires libraries or behavior associated with `glibc`, or required build packages are missing.

**Improvement:** Inspect the failing dependency, install only required packages, use a compatible base image, and run the full application test suite.

### Scenario 4: The container starts but cannot serve traffic

**Likely causes:** The application listens only on `127.0.0.1`, the port mapping is wrong, the process exited, or the health route is incorrect.

**Checks:**

```bash
docker ps
docker logs <container>
docker inspect <container>
curl http://localhost:3001/health
```

Confirm that the app listens on `0.0.0.0`, the container listens on `3000`, and the host mapping uses `-p 3001:3000`.
