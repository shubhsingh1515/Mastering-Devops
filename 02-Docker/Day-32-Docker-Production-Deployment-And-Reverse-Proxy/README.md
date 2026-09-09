# DevOps Mentorship Program - Day 32

## Phase 2: Docker & Containers

### Docker Production Deployment & Reverse Proxy
 
**Level:** Intermediate to Professional  
**Focus:** Running a Dockerized MERN application behind Nginx in a production-style architecture.

> Today continues the Docker phase. CI/CD is not introduced yet.

---

## 1. Learning Objectives

By the end of this lesson, you should understand:

- Why production applications commonly use a reverse proxy
- How Nginx fits in front of a Dockerized MERN application
- The difference between host ports and container ports
- How Docker services communicate internally
- Why internal services should not be exposed unnecessarily
- How Nginx routes traffic to a Node API
- How to troubleshoot a `502 Bad Gateway`
- How this architecture prepares for load balancing, HTTPS, scaling, and later CI/CD

---

## 2. Why Use a Reverse Proxy?

A development deployment may expose Node directly:

```text
Internet
    |
    v
Node.js :3000
```

Users then access a URL such as:

```text
http://example.com:3000
```

A production deployment commonly puts Nginx at the public boundary:

```text
Internet
    |
    v
Nginx :80 / :443
    |
    +--> React frontend
    |
    +--> Node API
             |
             v
          MongoDB
```

Nginx becomes the public entry point. It can handle TLS termination, static files, routing, request limits, headers, compression, and forwarding to internal services.

---

## 3. What Is a Reverse Proxy?

A reverse proxy receives client requests and forwards them to internal application services.

For example:

```text
Client
  |
  | GET /api/users
  v
Nginx
  |
  | proxy_pass http://api:3000
  v
Node API
```

The client does not need to know the API container name, internal IP, or internal port. This creates a useful boundary:

```text
PUBLIC
  |
  v
Nginx
  |
PRIVATE DOCKER NETWORK
  |
  +--> Frontend
  +--> API
          |
          v
       MongoDB
```

---

## 4. Production MERN Architecture

A production-style Docker topology is:

```text
                         Internet
                             |
                             v
                      +---------------+
                      |     Nginx     |
                      |    :80/:443   |
                      +-------+-------+
                              |
               +--------------+--------------+
               |                             |
               v                             v
        React frontend                  Node/Express API
                                               |
                                               v
                                           MongoDB
                                               |
                                               v
                                           Volume
```

Services may include:

- `nginx`
- `frontend`
- `api`
- `mongodb`

All services that need to communicate internally join the appropriate Docker network. MongoDB uses persistent storage mounted at `/data/db`.

---

## 5. Host Ports Versus Container Ports

Consider this Compose configuration:

```yaml
api:
  ports:
    - "3000:3000"
```

The mapping is:

```text
HOST                 CONTAINER
3000 ---------------> 3000
```

A request to `localhost:3000` on the Docker host reaches port `3000` inside the API container.

However, Nginx is another container. It should not leave the Docker network and route through the host. It can communicate directly with:

```text
http://api:3000
```

This distinction is important:

```text
Host-to-container access       -> published host port
Container-to-container access -> service name and container port
```

If Nginx is the only public entry point, the API may not need a `ports` section at all. It only needs to expose its internal listening port to other services on the network.

---

## 6. Docker Service Discovery

Compose provides internal DNS for services on the same network.

Nginx can use:

```text
http://api:3000
```

The API can use:

```text
mongodb://mongodb:27017/mern
```

Do not use these for separate containers:

```text
http://localhost:3000
mongodb://localhost:27017/mern
```

Inside the Nginx container, `localhost` means Nginx. Inside the API container, `localhost` means the API. Service names provide stable logical endpoints even when container IPs change.

---

## 7. Why MongoDB Should Stay Private

Avoid exposing every service:

```text
Internet
   +--> :80    Nginx
   +--> :443   Nginx
   +--> :3000  API
   +--> :27017 MongoDB
```

Prefer:

```text
Internet
    |
    v
Nginx
    |
    v
API
    |
    v
MongoDB
```

MongoDB is normally reachable only by trusted application components. Do not publish `27017` unless there is a specific, controlled operational requirement.

This follows the principle:

> Expose only what needs to be exposed.

Reducing public ports reduces attack surface and makes the intended request path easier to reason about.

---

## 8. Nginx Configuration

A simple reverse-proxy configuration is:

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://frontend:80;
    }

    location /api/ {
        proxy_pass http://api:3000;
    }
}
```

The request paths are:

```text
/api/users
     |
     v
Nginx
     |
     v
api:3000
```

and:

```text
/
     |
     v
Nginx
     |
     v
frontend:80
```

The exact `proxy_pass` path behavior depends on the trailing slash and location definition. Test the resulting URL paths rather than assuming the URI is rewritten exactly as expected.

Nginx may also serve the frontend directly, which is often simpler than running a React development server in production.

---

## 9. React Production Pattern

Do not normally run the React development server as the production frontend.

Prefer:

```text
React source
     |
     v
npm run build
     |
     v
Static dist assets
     |
     v
Nginx
```

A multi-stage image separates build-time Node tooling from the runtime web server:

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

The final image contains Nginx and static assets rather than the entire Node development environment.

---

## 10. Compose Architecture

A simplified production-minded Compose design is:

```yaml
services:
  nginx:
    build: ./nginx
    ports:
      - "80:80"
    depends_on:
      api:
        condition: service_healthy
    networks:
      - mern-network

  api:
    build: ./backend
    environment:
      MONGO_URI: mongodb://mongodb:27017/mern
    networks:
      - mern-network
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/health', response => process.exit(response.statusCode === 200 ? 0 : 1)).on('error', () => process.exit(1))"]
      interval: 10s
      timeout: 5s
      retries: 3

  mongodb:
    image: mongo:8
    volumes:
      - mongo-data:/data/db
    networks:
      - mern-network

volumes:
  mongo-data:

networks:
  mern-network:
```

Notice that the API does not need:

```yaml
ports:
  - "3000:3000"
```

when Nginx is the only service that needs to reach it. Nginx can access `api:3000` over the internal network.

`depends_on` expresses startup dependency. It is not a complete readiness strategy unless paired with a meaningful health check and application retry behavior.

---

## 11. Public Versus Private Ports

A focused production configuration may expose only Nginx:

```yaml
nginx:
  ports:
    - "80:80"

api:
  # no public host port

mongodb:
  # no public host port
```

The traffic model is:

```text
PUBLIC
Host :80 / :443
      |
      v
    Nginx

PRIVATE
Nginx -> api:3000
API   -> mongodb:27017
```

This does not mean the API has no port. It still listens on `3000` inside its container; that port is simply not published to the host.

---

## 12. The Famous 502 Bad Gateway

Suppose a client requests:

```text
GET /api/users
```

and Nginx returns:

```text
502 Bad Gateway
```

Conceptually:

```text
Client
  |
  v
Nginx
  |
  X
  v
API upstream
```

A `502` generally means that the proxy could not obtain a valid response from its upstream. It does not prove that Nginx itself is the root cause.

Possible causes include:

- API container is not running
- API process crashed
- API listens on the wrong port
- API binds only to an unexpected interface
- wrong Docker service name
- Nginx and API are on different networks
- Nginx configuration is invalid
- Docker DNS or connectivity failure
- API health or dependency failure

---

## 13. How to Troubleshoot a 502

Do not restart everything immediately. Trace the request path.

### Step 1: Check services

```bash
docker compose ps -a
```

### Step 2: Read Nginx logs

```bash
docker compose logs --tail 100 nginx
```

### Step 3: Read API logs

```bash
docker compose logs --tail 100 api
```

### Step 4: Inspect the network

```bash
docker network ls
docker network inspect <network>
```

Confirm that Nginx and API share a network.

### Step 5: Test API DNS from Nginx

```bash
docker compose exec nginx getent hosts api
```

If the name does not resolve, investigate service names and network configuration.

### Step 6: Test API connectivity from Nginx

If the Nginx image includes a suitable tool:

```bash
docker compose exec nginx wget -qO- http://api:3000/health
```

If this succeeds, Nginx can reach the API and the problem may be the Nginx route or proxy configuration. If it fails, continue with API, network, port, and health investigation.

### Request-path model

```text
Client
  |
  v
Nginx configuration
  |
  v
Docker network
  |
  v
Service DNS
  |
  v
API port and listener
  |
  v
API health
  |
  v
MongoDB dependency
```

---

## 14. Health Checks and Reverse Proxying

The architecture should verify more than container existence:

```text
Nginx
  |
  v
API
  |
  +--> Process running?
  +--> Port listening?
  +--> /health responding?
  +--> Database reachable where appropriate?
```

A process can be running but unable to serve requests. Nginx should route traffic only to a service that is ready according to the deployment's health model.

Avoid putting expensive dependency checks into liveness if a temporary database outage should remove an instance from traffic rather than restart every instance. Separate liveness and readiness semantics when the platform supports them.

---

## 15. User Request Flow

Consider:

```text
https://example.com/api/users
```

The flow is:

1. DNS resolves `example.com` to the public server or load balancer.
2. The request reaches Nginx on port `80` or `443`.
3. Nginx matches `/api/`.
4. Nginx forwards to `http://api:3000`.
5. Node/Express processes the request.
6. Node connects to `mongodb:27017`.
7. MongoDB returns data.
8. The response travels back through Node and Nginx to the browser.

```text
MongoDB
   |
   v
Node API
   |
   v
Nginx
   |
   v
Browser
```

This complete request flow is useful interview material and a practical troubleshooting map.

---

## 16. Why This Architecture Scales

Today:

```text
Nginx
  |
  v
API x 1
```

Later:

```text
             Nginx
               |
       +-------+-------+
       v       v       v
     API-1   API-2   API-3
```

Nginx or a load balancer can distribute requests across multiple API instances. This leads naturally to:

- load balancing
- horizontal scaling
- high availability
- rolling deployments
- cloud load balancers
- health-based routing

Those topics come later, but the reverse-proxy boundary is the foundation.

---

## 17. Practical Implementation

### Step 1: Create the Nginx directory

```bash
mkdir nginx
```

Create `nginx/nginx.conf`.

### Step 2: Add a basic configuration

```nginx
server {
    listen 80;

    location /api/ {
        proxy_pass http://api:3000;
    }
}
```

### Step 3: Create the Nginx Dockerfile

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/conf.d/default.conf
```

### Step 4: Add Nginx to Compose

```yaml
nginx:
  build: ./nginx
  ports:
    - "80:80"
  depends_on:
    - api
  networks:
    - mern-network
```

Connect Nginx and API to the same network.

### Step 5: Rebuild

```bash
docker compose up -d --build
```

### Step 6: Verify

```bash
docker compose ps
curl http://localhost/api/health
```

If the API responds, the path is working:

```text
Browser
   |
   v
Nginx
   |
   v
Node API
   |
   v
MongoDB
```

---

## 18. Common Mistakes

### Mistake 1: Nginx uses `localhost`

Usually wrong:

```nginx
proxy_pass http://localhost:3000;
```

Correct when API is another Compose service:

```nginx
proxy_pass http://api:3000;
```

### Mistake 2: Publishing every service

Avoid exposing all of these without a reason:

```yaml
mongodb:
  ports:
    - "27017:27017"

api:
  ports:
    - "3000:3000"

nginx:
  ports:
    - "80:80"
```

Prefer Nginx as the public boundary, with API and MongoDB private where appropriate.

### Mistake 3: Using a React development server in production

Prefer:

```text
React source -> build -> static files -> Nginx
```

### Mistake 4: Assuming `depends_on` means ready

It expresses dependency behavior, not complete application readiness. Use health checks and retry logic.

### Mistake 5: Restarting on every 502

Diagnose Nginx logs, API logs, network, DNS, port, and health before restarting.

---

## 19. Best Practices

- Expose `80` and `443` through the intended reverse proxy.
- Keep API and MongoDB private when possible.
- Use service names such as `api` and `mongodb`, not container IPs.
- Keep Nginx configuration in Git.
- Add `/health` and meaningful readiness behavior.
- Keep containers replaceable.
- Do not edit running production containers manually.
- Use versioned images and preserve rollback artifacts.
- Test the full request path from Nginx to MongoDB.
- Document the public and private port model.

---

## 20. Hands-On Lab

Use the Day 30 or Day 31 MERN project.

### Phase 1: Start

```bash
docker compose up -d --build
```

### Phase 2: Verify services

```bash
docker compose ps
```

### Phase 3: Test through Nginx

```bash
curl http://localhost/api/health
```

### Phase 4: Inspect the network

```bash
docker network inspect <network>
```

### Phase 5: Enter Nginx

```bash
docker compose exec nginx sh
```

Then test the API from inside Nginx if the image contains the tool:

```bash
wget -qO- http://api:3000/health
```

### Phase 6: Deliberate failure

Change:

```nginx
proxy_pass http://api:3000;
```

to:

```nginx
proxy_pass http://wrong-api:3000;
```

Rebuild or recreate Nginx:

```bash
docker compose up -d --build nginx
curl http://localhost/api/health
```

Observe the failure, then diagnose:

```bash
docker compose logs nginx
docker compose ps
docker network inspect <network>
```

Restore `http://api:3000`, rebuild, and verify recovery.

---

## 21. Quiz

### 1. Nginx and Node are separate containers. Which address is normally correct for Nginx to reach Node?

A. `localhost:3000`  
B. `api:3000`  
C. `127.0.0.1:3000`  
D. Host public IP only

**Answer: B.** `api` is resolved by Docker service discovery.

### 2. What is the primary purpose of a reverse proxy?

A. Store MongoDB data  
B. Forward client requests to backend services  
C. Build Docker images  
D. Create Git branches

**Answer: B.**

### 3. Which service should generally not be publicly exposed in a basic MERN Docker architecture?

A. Nginx  
B. MongoDB  
C. HTTPS endpoint  
D. Reverse proxy

**Answer: B.**

### 4. Nginx returns `502 Bad Gateway`. What should you do first?

A. Delete all containers  
B. Diagnose the upstream path and backend connectivity  
C. Delete the Docker volume  
D. Reinstall Docker

**Answer: B.**

### 5. Why should Nginx not use a hard-coded container IP?

A. IPs never work in Docker  
B. Container IPs can change  
C. Nginx does not support IP addresses  
D. Docker requires DNS

**Answer: B.**

### 6. What does this mean?

```yaml
ports:
  - "80:80"
```

A. Host port 80 maps to container port 80  
B. Container port 80 maps to MongoDB  
C. Two containers share port 80  
D. Nginx uses port 8080

**Answer: A.**

### 7. What is a common production approach for React?

A. Run the development server forever  
B. Build static assets and serve them through a production web server  
C. Run MongoDB inside React  
D. Put React source code in MongoDB

**Answer: B.**

### 8. Why might the API have no `ports` section in Compose?

A. It cannot communicate  
B. Nginx can reach it over the internal Docker network  
C. Node does not use ports  
D. Compose deletes the API

**Answer: B.**

### 9. Which path represents a correct request flow?

A. MongoDB -> Browser -> Nginx -> API  
B. Browser -> Nginx -> API -> MongoDB  
C. Browser -> MongoDB -> Nginx  
D. API -> Browser -> MongoDB

**Answer: B.**

### 10. A container is `Up`, but Nginx returns `502`. What does this tell you?

A. The backend must be healthy  
B. Container state alone does not prove upstream connectivity  
C. Docker networking is definitely working  
D. MongoDB is definitely broken

**Answer: B.**

### Answer Key

```text
1 -> B
2 -> B
3 -> B
4 -> B
5 -> B
6 -> A
7 -> B
8 -> B
9 -> B
10 -> B
```

### Score guide

- 9-10: Excellent
- 7-8: Good; review weak areas
- 5-6: Review Docker networking
- Below 5: Revisit Compose and reverse proxy fundamentals

---

## 22. Diagnostic Script Exercise

Create `diagnose-stack.sh`:

```bash
#!/usr/bin/env bash

set -u

printf '=== SERVICES ===\n'
docker compose ps -a

printf '\n=== API LOGS ===\n'
docker compose logs --tail 50 api

printf '\n=== NGINX LOGS ===\n'
docker compose logs --tail 50 nginx

printf '\n=== NETWORKS ===\n'
docker network ls

api_status="$(docker compose ps --status running --services | Select-String '^api$')"

printf '\n=== RESULT ===\n'
if [ -n "$api_status" ]; then
  printf 'API container: RUNNING\n'
else
  printf 'API container: NOT RUNNING\n'
  exit 1
fi

if ! curl --fail --silent http://localhost/api/health > /dev/null; then
  printf 'Public API health check: FAILED\n'
  exit 1
fi

printf 'Public API health check: OK\n'
```

The `Select-String` line is PowerShell-specific and should be replaced on Linux/macOS with a POSIX-compatible check such as `grep`. Adapt the script to your host environment rather than copying it blindly.

The script should report services, logs, network information, API state, and an HTTP health result. A non-zero exit code makes it useful in automation later.

---

## 23. Mini Assignment

Update the Day 30 or Day 31 project README with:

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

Document:

- which services are publicly exposed
- which services are internal
- which Docker network they use
- how Nginx reaches the API
- how the API reaches MongoDB
- how to diagnose a `502`
- where persistent data lives
- how health checks work
- how to test the public request path

This documentation is part of the project, not an optional extra.

---

## 24. Interview Preparation

### Beginner

#### Q1. What is a reverse proxy?

A server that receives client requests and forwards them to backend services.

#### Q2. Why use Nginx in front of Node.js?

It can provide reverse proxying, TLS termination, static-file serving, routing, load balancing, headers, and request handling at the public boundary.

### Intermediate

#### Q3. Why should Nginx use `api:3000` instead of `localhost:3000`?

Nginx and the API are separate containers. Inside Nginx, `localhost` refers to Nginx itself. `api` resolves to the API service through Docker networking.

#### Q4. What does `502 Bad Gateway` mean?

It generally means the proxy could not obtain a valid response from its upstream. Investigate proxy configuration, Docker networking, service discovery, backend availability, the listening port, and backend health.

#### Q5. Why should MongoDB not be publicly exposed when only the API needs it?

Public exposure unnecessarily increases attack surface. MongoDB can remain reachable only by trusted application components on the internal network.

### Advanced

#### Q6. How would you troubleshoot a production `502`?

Check Nginx logs, API logs, service state, network membership, DNS resolution for `api`, the API listening port and interface, connectivity from Nginx, the API health endpoint, and API dependencies.

#### Q7. Why are service names preferable to container IPs?

Service names provide stable logical discovery while container IP addresses can change when services are recreated.

#### Q8. How does this architecture prepare for scaling?

Nginx provides a stable public boundary. Multiple API containers can later be placed behind it or behind a load balancer, with health-aware routing and rolling deployment strategies.

---

## 25. Review From Earlier Lessons

### Networking

Why does this fail when MongoDB is another container?

```text
mongodb://localhost:27017
```

Because `localhost` points to the API container, not MongoDB.

### Volumes

What protects MongoDB data during container recreation?

A Docker volume mounted at `/data/db`.

### Image optimization

Why use this order?

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

It improves layer-cache reuse.

### Security

Why should secrets not be in a Dockerfile?

They can become embedded in image layers and exposed through image inspection or distribution.

### Monitoring

What shows live container CPU and memory usage?

```bash
docker stats
```

---

## 26. Production Checklist

Before calling the reverse-proxy deployment complete, verify:

- [ ] Nginx is the intended public entry point.
- [ ] API and MongoDB are private where appropriate.
- [ ] Nginx uses `api:3000`, not `localhost:3000`.
- [ ] API uses `mongodb:27017`, not `localhost:27017`.
- [ ] All communicating services share the intended network.
- [ ] MongoDB is not unnecessarily published.
- [ ] React is built for production.
- [ ] Static assets are served by a production web server.
- [ ] API and MongoDB health behavior is documented.
- [ ] Nginx logs and API logs are accessible.
- [ ] A `502` diagnostic sequence is documented.
- [ ] Service names are used instead of container IPs.
- [ ] Nginx configuration is version-controlled.
- [ ] Public and private ports are documented.
- [ ] Persistent data has a volume and independent backup strategy.

---

## 27. Day 32 Summary

The key concepts are:

```text
Reverse proxy
Client -> Nginx -> Backend

Service discovery
Nginx -> api:3000
API   -> mongodb:27017

Public/private architecture
PUBLIC:  Nginx
PRIVATE: API, MongoDB

Production React
React source -> build -> static files -> Nginx

502 troubleshooting
502
  -> Nginx logs
  -> API logs
  -> network
  -> DNS
  -> port
  -> health
```

### Key interview principle

> Do not troubleshoot containers in isolation. Trace the request from the client through every dependency boundary.

---