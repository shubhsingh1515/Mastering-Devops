# Day 22 — Detailed Answer Guide

This guide provides direct answers for the practical work in the Day 22 lesson.

## Starting Dockerfile Review

```dockerfile
FROM node:22
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "server.js"]
```

### Why is this inefficient?

`COPY . .` copies the entire build context before dependency installation. A change to `server.js`, a test file, or another copied file can invalidate the layer before `RUN npm install`. Docker may then reinstall dependencies even though the dependency manifests did not change.

### Improved order

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

The package manifests are stable inputs for the dependency layer. Source changes after that point can reuse the expensive dependency installation step.

## `.dockerignore` Answer

```text
node_modules
.git
.gitignore
.env
npm-debug.log
coverage
```

- `node_modules` should be installed in the image for the target environment.
- `.git` contains repository history that the runtime does not need.
- `.env` may contain credentials and must not be copied accidentally.
- Logs and coverage output are generated artifacts, not application runtime requirements.

`.dockerignore` reduces context size, but production secrets still need proper runtime secret management.

## Cache Experiment Answer

Changing only `server.js` should normally rebuild the source-copy layer and later layers while reusing the base image, manifest copy, and dependency installation layer.

Changing `package.json` or `package-lock.json` changes the input to the dependency layer. Docker should therefore rebuild the dependency installation step.

Actual cache behavior depends on all instruction inputs and the builder, so the build output is the evidence.

## Production Backend Answer

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

This version:

1. separates dependency preparation from runtime setup
2. uses a potentially smaller compatible base
3. installs only production dependencies
4. copies dependencies into a clean runtime stage
5. avoids unnecessary build tools in the final stage
6. runs the application as a non-root user

The real application must copy every source or compiled file it needs at runtime.

## React Multi-Stage Answer

The Node build stage contains the source code, Node runtime, dependencies, and build tools. It creates `/app/dist`.

The Nginx runtime stage receives only `/app/dist`:

```dockerfile
COPY --from=build /app/dist /usr/share/nginx/html
```

This keeps Node and frontend development tooling out of the final web-serving image.

## Practical Assessment Answer

At least five valid improvements are:

1. Replace the full base image with a compatible slim or Alpine image.
2. Add `.dockerignore`.
3. Copy `package*.json` before source code.
4. Use `npm ci` with a lockfile.
5. Install production dependencies only.
6. Use a multi-stage build.
7. Run as a non-root user.
8. Copy only files required at runtime.
9. Inspect `docker history` for large layers.
10. Build and scan the image in CI.

### Build-speed improvements

Layer ordering, `.dockerignore`, dependency caching, and avoiding unnecessary context improve build speed.

### Runtime-size improvements

A suitable base image, production dependencies, multi-stage builds, and selective copying reduce runtime size.

### Security improvements

Non-root execution, no baked-in secrets, minimal packages, regular updates, and vulnerability scanning improve security.

## Port and Health Check Answer

If the application listens on container port `3000`, run:

```bash
docker run -d --name mern-api-v11 -p 3001:3000 mern-api:1.1
```

Then test:

```bash
curl http://localhost:3001/health
```

If it fails, check:

```bash
docker ps
docker logs mern-api-v11
docker inspect mern-api-v11
```

The Node application should listen on `0.0.0.0`, not only `127.0.0.1`, so traffic arriving through the container network can reach it.

## Final Principle

A production image should contain what the application needs at runtime, not everything that happened to exist in the development environment. Optimization must preserve compatibility, security, reproducibility, and observable application behavior.
