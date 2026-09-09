# Day 32 - Assignment: MERN Reverse Proxy Deployment

## Objective

Place the Day 30 or Day 31 MERN application behind Nginx and prove that the request path works through Docker service discovery, private networking, health checks, and documented troubleshooting.

---

## Part 1: Add Nginx

Create:

```text
nginx/
+-- nginx.conf
+-- Dockerfile
```

Use this Dockerfile:

```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

Start with this configuration:

```nginx
server {
    listen 80;

    location /api/ {
        proxy_pass http://api:3000;
    }
}
```

Explain why `api:3000` is correct and `localhost:3000` is usually wrong inside Nginx.

---

## Part 2: Update Compose

Add an `nginx` service:

```yaml
services:
  nginx:
    build: ./nginx
    ports:
      - "80:80"
    depends_on:
      - api
    networks:
      - mern-network
```

Connect Nginx and API to the same network. Keep MongoDB private and do not publish `27017` without a documented reason.

Validate and start:

```bash
docker compose config
docker compose up -d --build
docker compose ps
```

---

## Part 3: Test the Request Path

Call the API through Nginx:

```bash
curl http://localhost/api/health
```

Expected response:

```json
{
  "status": "ok"
}
```

Document this flow:

```text
Client -> Nginx -> api:3000 -> mongodb:27017
```

If the API is published directly for development, compare the direct path with the reverse-proxy path:

```bash
curl http://localhost:3000/health
curl http://localhost/api/health
```

---

## Part 4: Verify Service Discovery

Run:

```bash
docker compose exec nginx getent hosts api
docker compose exec api getent hosts mongodb
docker network inspect <network>
```

Explain:

- why service names are stable logical endpoints
- why container IPs should not be hardcoded
- why host ports are not required for internal communication

---

## Part 5: Production Frontend Pattern

If using React, build static assets rather than running the development server in production.

Use a multi-stage build:

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

Document:

- which stage contains build dependencies
- which stage serves runtime files
- why the final image does not need the full Node toolchain

---

## Part 6: Public and Private Ports

Document the intended port model:

```text
Public:
  Nginx host 80/443

Private:
  Nginx -> api:3000
  API   -> mongodb:27017
```

Remove unnecessary host mappings such as:

```yaml
mongodb:
  ports:
    - "27017:27017"
```

The API may also omit its host `ports` mapping if Nginx is the only client that needs it.

---

## Part 7: Deliberate 502 Exercise

Change:

```nginx
proxy_pass http://api:3000;
```

to:

```nginx
proxy_pass http://wrong-api:3000;
```

Rebuild Nginx:

```bash
docker compose up -d --build nginx
```

Call:

```bash
curl http://localhost/api/health
```

Expected result: the request fails, commonly with a `502 Bad Gateway` or another proxy error.

Diagnose without randomly restarting all services:

```bash
docker compose logs nginx
docker compose logs api
docker compose ps
docker network inspect <network>
docker compose exec nginx getent hosts api
```

Restore `api:3000`, rebuild, and verify recovery.

---

## Part 8: Write the 502 Runbook

Create a short runbook with this sequence:

1. Check Compose service state.
2. Read Nginx logs.
3. Read API logs.
4. Validate Nginx configuration.
5. Confirm Nginx and API share a network.
6. Resolve `api` from inside Nginx.
7. Test `http://api:3000/health` from Nginx.
8. Confirm the API listens on the expected port and interface.
9. Check API dependencies and readiness.
10. Test the public URL again.

For every step, record what a success or failure means.

---

## Part 9: Create `diagnose-stack.sh`

The script should:

1. Print Compose service state.
2. Print recent API logs.
3. Print recent Nginx logs.
4. Print Docker network information.
5. Report whether the API service is running.
6. Test `http://localhost/api/health`.
7. Return a non-zero exit code if the health test fails.

Adapt the script to your operating system. On Windows PowerShell, use PowerShell commands instead of Unix `grep` syntax.

---

## Part 10: Update the Project README

Document:

```text
Internet
   |
   v
Nginx
   |
   +--> React frontend
   |
   +--> Node API
           |
           v
        MongoDB
           |
           v
        Volume
```

Include:

- public services
- private services
- Docker network name
- Nginx-to-API address
- API-to-MongoDB address
- public and internal ports
- health checks
- 502 troubleshooting
- persistence and backup location
- startup and shutdown commands

---

## Submission Checklist

- [ ] Nginx Dockerfile created
- [ ] Nginx configuration stored in Git
- [ ] Nginx added to Compose
- [ ] Nginx and API share a network
- [ ] API uses `api:3000` from Nginx
- [ ] API uses `mongodb:27017` for MongoDB
- [ ] MongoDB is not unnecessarily published
- [ ] React uses a production build pattern
- [ ] Public health request succeeds
- [ ] Deliberate `502` was created and diagnosed
- [ ] Nginx and API logs were inspected
- [ ] Network and service DNS were verified
- [ ] Diagnostic script created
- [ ] Project README updated
