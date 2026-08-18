# Day 21: Docker Interview Questions (with Detailed Answers)

## Beginner Questions

### 1. What is Docker?
**Answer:**
Docker is a platform that allows you to package applications and their dependencies into isolated runtime environments called containers.

This helps reduce environment drift and makes deployment more predictable.

---

### 2. What is a Docker image?
**Answer:**
A Docker image is a packaged, reusable artifact that contains the application code, runtime, dependencies, and configuration required to run the app.

Example:
```text
node-api:1.0
```

This image can be reused to create multiple containers.

---

### 3. What is a Docker container?
**Answer:**
A container is a running instance of a Docker image.

An image is the blueprint. A container is the running version.

Example:
```text
Image: node-api:1.0
Container 1
Container 2
Container 3
```

---

### 4. What is the difference between a virtual machine and a container?
**Answer:**
A VM includes a full guest operating system, while a container shares the host operating system kernel and isolates only the application runtime.

This makes containers lighter and faster than traditional VMs.

---

### 5. Why is Docker useful for a MERN app?
**Answer:**
Docker helps package Node.js, the app code, and dependencies into a consistent runtime environment. This reduces differences between development, testing, and production.

It also helps with:
- reproducibility
- portability
- rollback
- scaling
- CI/CD integration

---

## Intermediate Questions

### 6. What does `docker run -p 8080:3000 app` do?
**Answer:**
This maps the host port `8080` to the container port `3000`.

So the app is accessible at:
```bash
http://localhost:8080
```

Inside the container, the app is still listening on port `3000`.

---

### 7. What is the difference between `RUN` and `CMD` in a Dockerfile?
**Answer:**
- `RUN` executes during the image build phase.
- `CMD` defines the default command executed when the container starts.

Example:
```dockerfile
RUN npm ci
CMD ["node", "server.js"]
```

The first installs dependencies during build. The second starts the app when the container runs.

---

### 8. Does `EXPOSE 3000` publish the port to the host automatically?
**Answer:**
No.

`EXPOSE 3000` only documents the port the application uses. To publish it to the host, you must use port mapping such as:

```bash
docker run -p 3000:3000 app
```

This is a very common Docker interview question.

---

### 9. Why do we usually run `COPY package*.json` before `COPY . .`?
**Answer:**
Because this allows Docker to cache dependency installation more efficiently.

Example:
```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

If only application code changes, Docker can reuse the cached dependency layer instead of reinstalling dependencies every time.

---

### 10. Why do containers usually use `0.0.0.0` for server binding?
**Answer:**
Because inside a container, the application must listen on a network interface that accepts traffic routed from outside the local loopback.

If you bind to `127.0.0.1`, the process may reject traffic from other sources, including container-to-host port mapping.

---

## Advanced Questions

### 11. Why is Docker good for production deployment?
**Answer:**
Docker makes applications portable, reproducible, and easier to deploy consistently across environments.

It promotes:
- stable deployment pipelines
- versioned images
- standard runtime environments
- easier rollback and auditing

---

### 12. What is the main difference between an image and a container in one sentence?
**Answer:**
An image is the packaged application environment, while a container is a running instance of that image.

---

### 13. If a Dockerized Node.js app is not reachable from the host, where do you check first?
**Answer:**
Check these in order:
1. `docker ps`
2. `docker logs <container>`
3. `docker port <container>`
4. `docker exec -it <container> sh`
5. `curl http://localhost:<mapped-port>/health`

This helps determine whether the issue is startup, port mapping, app binding, or application logic.

---

### 14. Why do you not store secrets in a Dockerfile?
**Answer:**
Because Docker images can be shared, stored in registries, and inspected by others. Credentials inside an image are a major security risk.

Production secrets should be provided using:
- environment variables
- secret managers
- Kubernetes secrets
- cloud secret stores

---

### 15. How does Docker help with rollback strategies?
**Answer:**
Because builds produce versioned image tags such as:
```text
mern-api:1.0
mern-api:1.1
```

If a release fails, you can roll back to the previous working image instead of redeploying manually from source.

This is important in DevOps workflows and release engineering.

---

## Practical Interview Answer Example

### Q: Why would you use Docker for a MERN application?
**Strong answer:**
> I would use Docker because it packages the application and all runtime dependencies into a consistent artifact. This reduces differences between local development, testing, and production. It also makes deployment more reproducible, easier to version, easier to rollback, and compatible with CI/CD automation.

Then add:
- container isolation
- standard runtime
- better scaling and portability
- faster deployment cycles

---

## Final Tip

When answering Docker interview questions, always say:
- image = packaged app
- container = running instance
- `RUN` = build time
- `CMD` = runtime
- `EXPOSE` = documents the port
- `-p` = actually publishes the port

This one distinction alone makes you sound much more experienced.
