# Day 22 — Assignment: Docker Images, Layers & Build Optimization

## Objective
Optimize a Node.js Docker image by improving layer ordering, build-context control, image size, security, and runtime behavior.

---

## Part 1: Review the Starting Dockerfile

Use this intentionally inefficient Dockerfile:

```dockerfile
FROM node:22

WORKDIR /app

COPY . .

RUN npm install

EXPOSE 3000

CMD ["node", "server.js"]
```

Answer these questions:

1. Why can `COPY . .` before `RUN npm install` make builds slow?
2. Which files should be excluded from the build context?
3. Which dependency command is more reproducible when a lockfile exists?
4. Why might the full `node:22` image be larger than necessary?
5. Why should the application avoid running as root?

---

## Part 2: Create `.dockerignore`

Create a `.dockerignore` file with at least:

```text
node_modules
.git
.env
npm-debug.log
coverage
```

Add other files that are not required by the build. Explain why each selected entry should be excluded.

### Expected answer

- `node_modules`: local dependencies may be platform-specific and should be installed inside the image.
- `.git`: repository history is not needed at runtime and increases context size.
- `.env`: prevents accidental copying of local secrets.
- `npm-debug.log`: local diagnostic artifacts are unnecessary.
- `coverage`: test output is not needed in the production runtime.

---

## Part 3: Improve Layer Ordering

Create an improved Dockerfile:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 3000
CMD ["node", "server.js"]
```

### Explanation

The package manifests change less frequently than source code. Copying them first allows Docker to reuse the `npm ci` layer when only `server.js` or another source file changes.

The dependency layer should be rebuilt when `package.json` or `package-lock.json` changes because the installed dependency set may have changed.

---

## Part 4: Build and Inspect the Image

From the Day 21 Node API directory:

```bash
cd ~/devops-course/day21/node-api
docker build -t mern-api:1.1 .
docker images
docker history mern-api:1.1
```

Record:

- image size
- number of visible history entries
- base image
- dependency installation step
- source-copy step

Run the image:

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

### Expected result

The request should reach the application through host port `3001`, which maps to container port `3000`:

```text
Host :3001
     ↓
Container :3000
```

If the request fails, check:

```bash
docker ps
docker logs mern-api-v11
docker inspect mern-api-v11
```

---

## Part 5: Perform the Cache Experiment

1. Build the initial image:
   ```bash
   docker build -t mern-api:1.1 .
   ```
2. Change only `server.js`.
3. Build again:
   ```bash
   docker build -t mern-api:1.2 .
   ```
4. Observe which steps are cached.
5. Change `package.json`.
6. Build again:
   ```bash
   docker build -t mern-api:1.3 .
   ```

### Expected answer

When only `server.js` changes, Docker should normally reuse the base image, manifest copy, and dependency installation layers. When `package.json` or the lockfile changes, the manifest layer changes, so the dependency installation step must normally run again.

Cache behavior can also be affected by the builder, build arguments, copied files, and other instruction inputs, so confirm behavior from the actual build output.

---

## Part 6: Create a Production Backend Image

Create a production-oriented Dockerfile:

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

Explain why this version is better.

### Expected answer

- `node:22-alpine` may reduce image size, subject to compatibility testing.
- Dependency manifests are copied before installation, improving cache reuse.
- `npm ci --omit=dev` avoids development-only dependencies in the runtime dependency set.
- The final stage does not include the dependency-installation process or unnecessary build tools.
- `USER node` applies least privilege and avoids running the API as root.
- Only runtime files should be copied into the final image.

For a real backend with multiple modules, copy the complete runtime source tree or compiled output required by the application.

---

## Part 7: Multi-Stage React Build

Create a frontend Dockerfile:

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

Answer:

1. Which stage contains the Node build tools?
2. What is copied into the final Nginx image?
3. Why is this better than running the final frontend image with all Node development dependencies?

### Expected answer

The `build` stage contains the Node runtime, source, dependencies, and build tools. Only the generated `/app/dist` files are copied into the Nginx runtime stage. This reduces the runtime image contents and removes tools that are not needed to serve static files.

---

## Part 8: Security Review

Write a short security review covering:

- image base selection
- non-root runtime
- secrets
- Linux capabilities
- read-only filesystem
- vulnerability scanning
- image updates

### Expected answer

Use a supported and appropriately minimal base image. Run the application as a non-root user where possible. Keep secrets outside the image and inject them at runtime. Drop unnecessary capabilities and use a read-only filesystem where the app supports it. Scan the image and dependencies regularly, and rebuild when base-image security updates are released.

---

## Part 9: Practical Assessment

The original image is `1.2 GB` and builds take `10 minutes`. Developers frequently change `server.js`.

Identify at least five improvements and classify each one:

| Improvement | Build speed | Runtime size | Security |
|-------------|-------------|--------------|----------|
| Add `.dockerignore` | Yes | Often | Yes |
| Copy manifests first | Yes | No | No |
| Use `npm ci` | Reproducibility | No | Indirectly |
| Use production dependencies | No | Yes | Yes |
| Use multi-stage build | Sometimes | Yes | Yes |
| Use non-root user | No | No | Yes |
| Choose suitable base image | Pull speed | Yes | Depends on support |

Add a short explanation for your choices.

---

## Part 10: Month 2 Project Deliverables

Your containerized MERN application must include:

### Backend

- Dockerfile
- `.dockerignore`
- optimized dependency layer
- non-root runtime
- health endpoint

### Frontend

- Node build stage
- generated production assets
- Nginx runtime stage

### Evidence

Record the output or results of:

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

Document:

- image size before and after optimization
- build time before and after optimization
- which layers were reused
- why you selected the base images
- how secrets were kept out of the image
- how the runtime was tested

---

## Detailed Answer Summary

A strong solution should explain that Docker optimization is not one trick. It is a combination of:

1. controlling the build context with `.dockerignore`
2. placing stable dependency steps before frequently changing source steps
3. using lockfile-based installation with `npm ci`
4. excluding development dependencies from runtime images
5. using multi-stage builds where build and runtime requirements differ
6. running as a non-root user
7. keeping secrets out of image layers
8. selecting a compatible and supported base image
9. inspecting image history and measuring results
10. testing the optimized image for behavior, security, and reproducibility
