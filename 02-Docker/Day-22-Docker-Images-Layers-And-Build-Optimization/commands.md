# Day 22: Docker Images, Layers & Build Optimization Commands

## Build and inspect images

| Command | Purpose | Example |
|---------|---------|---------|
| `docker build -t name:tag .` | Build an image from the current context | `docker build -t mern-api:1.1 .` |
| `docker build --no-cache -t name:tag .` | Build without using cached layers | `docker build --no-cache -t mern-api:test .` |
| `docker images` | List local images and sizes | `docker images` |
| `docker image ls` | Alternative image listing command | `docker image ls` |
| `docker history image:tag` | Show image layers and build history | `docker history mern-api:1.1` |
| `docker inspect image:tag` | Show detailed image metadata | `docker inspect mern-api:1.1` |
| `docker image rm image:tag` | Remove a local image | `docker image rm mern-api:1.0` |

## Run and inspect containers

| Command | Purpose | Example |
|---------|---------|---------|
| `docker run -d --name name image` | Run in detached mode | `docker run -d --name api mern-api:1.1` |
| `docker run -p host:container image` | Publish a container port | `docker run -p 3001:3000 mern-api:1.1` |
| `docker ps` | List running containers | `docker ps` |
| `docker ps -a` | List all containers | `docker ps -a` |
| `docker logs name` | Show container logs | `docker logs api` |
| `docker logs -f name` | Follow logs live | `docker logs -f api` |
| `docker inspect name` | Show container metadata | `docker inspect api` |
| `docker exec -it name sh` | Open a shell in a running container | `docker exec -it api sh` |
| `docker stop name` | Stop a container | `docker stop api` |
| `docker rm name` | Remove a stopped container | `docker rm api` |

## Dockerfile instructions

```dockerfile
# Select a base image
FROM node:22-alpine

# Set the working directory
WORKDIR /app

# Copy dependency manifests first for cache reuse
COPY package*.json ./

# Install dependencies during build
RUN npm ci

# Copy source after dependencies
COPY . .

# Document the application port
EXPOSE 3000

# Set the default runtime command
CMD ["node", "server.js"]
```

## Recommended Node API Dockerfile

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

Copy all runtime files required by the real application. The example copies only `server.js` for clarity.

## Recommended `.dockerignore`

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

Do not ignore a directory that is intentionally required by the build. For example, a prebuilt frontend `dist` folder may need to remain in the context in some workflows.

## Multi-stage React build

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

## Cache experiment

```bash
# First build
docker build -t mern-api:1.1 .

# Change only server.js, then build again
docker build -t mern-api:1.2 .

# Change package.json, then build again
docker build -t mern-api:1.3 .

# Compare with a no-cache build
docker build --no-cache -t mern-api:no-cache .
```

Expected behavior: the source-only change should preserve the dependency layer, while a package manifest change should invalidate it.

## Hands-on commands

```bash
cd ~/devops-course/day21/node-api

docker build -t mern-api:1.1 .
docker images
docker history mern-api:1.1

docker run -d --name mern-api-v11 -p 3001:3000 mern-api:1.1
curl http://localhost:3001/health

docker logs mern-api-v11
docker inspect mern-api-v11
docker exec -it mern-api-v11 sh

docker stop mern-api-v11
docker rm mern-api-v11
```

## Measuring image size

```bash
docker images mern-api

docker image inspect mern-api:1.1 --format '{{.Size}}'
```

## Useful optimization checks

```bash
# Show the build context contents before building
find . -maxdepth 2 -type f

# Check whether local dependencies exist
du -sh node_modules 2>/dev/null

# Inspect image history for large RUN or COPY steps
docker history --no-trunc mern-api:1.1
```

## Important reminders

- `EXPOSE` documents a port; it does not publish it.
- `.dockerignore` controls build-context inclusion, not runtime secrets management.
- `COPY package*.json ./` before `RUN npm ci` improves cache reuse.
- `npm ci` is generally preferable for reproducible lockfile-based builds.
- Use `USER node` or another non-root user where practical.
- Validate compatibility before switching to Alpine.
