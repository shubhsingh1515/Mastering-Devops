# Day 26 - Commands: Docker Security & Production Hardening

## 1. Check the Runtime User

```bash
docker run -d --name app-demo myapp

docker exec -it app-demo whoami
```

This confirms whether the app is running as root or a non-root user.

## 2. Run a Container as a Non-Root User

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

Then build and run:

```bash
docker build -t myapp:secure .
docker run -d --name myapp-secure -p 3000:3000 myapp:secure
```

## 3. Drop All Linux Capabilities

```bash
docker run -d \
  --name myapp-capdrop \
  --cap-drop=ALL \
  myapp:secure
```

Use this carefully because some apps may require specific capabilities.

## 4. Run with a Read-Only Root Filesystem

```bash
docker run -d \
  --name myapp-ro \
  --read-only \
  -p 3000:3000 \
  myapp:secure
```

## 5. Add a Writable Temporary Directory

```bash
docker run -d \
  --name myapp-ro-tmp \
  --read-only \
  --tmpfs /tmp \
  -p 3000:3000 \
  myapp:secure
```

This is useful when the app writes temporary data at runtime.

## 6. Apply Memory and CPU Limits

```bash
docker run -d \
  --name myapp-limits \
  --memory=512m \
  --cpus=1 \
  -p 3000:3000 \
  myapp:secure
```

## 7. Inspect Container Security Metadata

```bash
docker inspect myapp-secure
```

Look at the `User`, `CapAdd`, `CapDrop`, `ReadOnlyRootFilesystem`, and related fields.

## 8. View Logs

```bash
docker logs myapp-secure
```

## 9. Validate a Health Endpoint

```bash
curl http://localhost:3000/health
```

## 10. Stop a Container

```bash
docker stop myapp-secure
```

## 11. Remove a Container

```bash
docker rm myapp-secure
```

## 12. Remove a Container and Any Attached Volumes

```bash
docker rm -v myapp-secure
```

Use with caution.

## 13. Security Troubleshooting Flow

```bash
docker ps
docker inspect myapp-secure
docker logs myapp-secure
docker exec -it myapp-secure sh
docker exec -it myapp-secure whoami
```

If a read-only filesystem causes failures, check what the app is trying to write and move that to an explicit writable mount.

## 14. Typical Hardening Pattern

```bash
docker run -d \
  --name mern-api-hardening \
  --user 1000:1000 \
  --read-only \
  --cap-drop=ALL \
  --memory=512m \
  --cpus=1 \
  --tmpfs /tmp \
  -p 3000:3000 \
  myapp:secure
```

This is a strong baseline, but it must be tested against the application's real runtime behavior.
