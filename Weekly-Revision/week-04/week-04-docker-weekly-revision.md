# DevOps Mentorship Program — Week 4 Sunday Revision

> **Sunday Special:** Reinforcing Docker Week (Days 21–23)
>
> **Focus:** Docker fundamentals, image layering and build optimization, networking, persistent storage, MERN production troubleshooting, and interview readiness
>
---

## 📊 Progress Tracker

### ✅ Linux Phase Complete
- Days 1–19 — Linux Foundation
- Day 20 — Linux Phase Revision & Assessment

### 🐳 Docker Phase
- Day 21 — Docker Fundamentals
- Day 22 — Images, Layers & Build Optimization
- Day 23 — Docker Networking & Persistent Storage
- Day 24 — Weekly Revision & Practical Assessment

**Docker Phase:** 3 / ~10 topics complete

---

## 🔁 1. Review From Earlier Topics

### Q1. Why should a production Node.js application normally avoid running as `root`?

Because running as `root` gives the process full system privileges. If the application is compromised or misconfigured, the attacker gains a much larger blast radius than a normal restricted user would have.

In production, the principle is least privilege:
- the app should only have the permissions it needs
- service files should not be world-writable
- the app should not have shell-level administrative access unless necessary

This is important for both security and operational safety.

### Q2. What command checks listening Linux sockets?

```bash
ss -tulpn
```

This command helps inspect what processes are listening on which ports.

- `-t` = TCP sockets
- `-u` = UDP sockets
- `-l` = listening sockets
- `-p` = show process information
- `-n` = show numeric addresses and ports

When a service is not reachable, this is one of the fastest ways to see whether the process is active and listening on the expected port.

### Q3. What command bypasses Nginx when Node listens on port 3000?

```bash
curl http://localhost:3000/health
```

This is the direct “backend health check.” If the backend is healthy, the app is likely running correctly. If Nginx returns a 502 or 504, this bypass check helps isolate whether the issue is in Nginx or in the application upstream.

### Q4. Why can `df -h` look healthy while file creation still fails?

Because the problem may not be disk space, but inode exhaustion.

```bash
df -i
```

A filesystem can appear to have free blocks while still running out of usable file entries. This often happens when there are millions of tiny files, log files, temporary files, or cache artifacts.

So production diagnosis should include:
- `df -h` for block usage
- `df -i` for inode exhaustion
- `du -sh` to find large directories
- `find` to locate stale temporary or log files

### Q5. What's the difference between persistence and backup?

A Docker volume provides persistence because the data remains outside the container filesystem and survives container restarts or removal.

A backup is a separate recovery mechanism designed for disaster recovery, accidental deletion, corruption, host failure, or restore testing.

In simple terms:
- persistence keeps data alive
- backup helps you recover data if something goes wrong

---

## 🧠 2. Docker Week — Core Mental Model

You should now be able to reason about Docker as a layered system:

```text
Dockerfile
    ↓
Build context
    ↓
Image layers
    ↓
Docker image
    ↓
Container
    ↓
Network
    ↓
Persistent storage
```

This is the mental model a DevOps engineer needs when designing real containerized systems.

For a MERN application, the architecture usually looks like this:

```text
                    Internet
                       │
                       ▼
                    Nginx
                       │
                 Docker network
                       │
              ┌────────┴────────┐
              ▼                 ▼
          Frontend           Node API
                                │
                                ▼
                             MongoDB
                                │
                                ▼
                           Volume/Storage
```

This visually explains the major principles:
- the public interface is typically Nginx
- the application services are internal to the Docker network
- MongoDB is not exposed broadly unless required
- database data should live on persisted storage, not in a container filesystem

---

## 📝 3. Weekly Quiz

## Questions

### 1. What command builds a Docker image?

A. `docker build`

B. `docker create-image`

C. `docker compile`

D. `docker make`

### 2. What is the difference between an image and a container?

A. They're identical

B. Image is the packaged artifact; container is an instance of it

C. Container is the build configuration

D. Image is always running

### 3. What does this mean?

```bash
-p 8080:3000
```

A. Container 8080 → host 3000

B. Host 8080 → container 3000

C. Both ports become public

D. MongoDB uses port 3000

### 4. Does `EXPOSE 3000` publish the port?

A. Yes

B. No

### 5. Why copy dependency manifests before application source?

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

A. Better Docker cache reuse

B. Faster DNS

C. Required for SSH

D. It exposes port 3000

### 6. What does `.dockerignore` do?

A. Stops Docker

B. Excludes files from the build context

C. Deletes Docker volumes

D. Encrypts images

### 7. Inside a Node container, what does `localhost` mean?

A. MongoDB container

B. Docker host

C. Node container itself

D. Nginx container

### 8. How should Node normally address a MongoDB container called `mongodb` on the same user-defined network?

A. `localhost:27017`

B. `mongodb:27017`

C. `127.0.0.1:3000`

D. `docker:27017`

### 9. What does a Docker volume provide?

A. Automatic disaster recovery

B. Persistent storage

C. Automatic encryption

D. Public networking

### 10. What is the purpose of a multi-stage Docker build?

A. Run multiple operating systems

B. Keep build-time dependencies out of the final runtime image

C. Replace Docker networking

D. Automatically create backups

---

## ✅ Answer Key

1. **A** — `docker build` creates an image from a Dockerfile and build context.
2. **B** — An image is a packaged artifact; a container is a running instance created from that image.
3. **B** — Host port `8080` is mapped to container port `3000`.
4. **B** — `EXPOSE` documents the port; it does not publish it to the host.
5. **A** — Copying dependency manifests first improves Docker cache reuse.
6. **B** — `.dockerignore` prevents unnecessary files from being sent to the build context.
7. **C** — From inside a container, `localhost` means the container itself.
8. **B** — On the same Docker network, the correct address is usually `mongodb:27017`.
9. **B** — A Docker volume provides persistent storage.
10. **B** — Multi-stage builds keep build-time dependencies out of the final runtime image.

### 🎯 Scoring

- 9–10 → 🟢 Excellent
- 7–8 → 🟢 Strong
- 5–6 → 🟡 Review
- <5 → 🔴 Revisit Days 21–23

---

## 💻 4. Practical Assessment — Containerized MERN Incident

## Scenario

Your production setup is:

```text
Internet
   │
   ▼
Nginx
   │
   ▼
Node API
   │
   ▼
MongoDB
```

All three are Docker containers.

Users report:

> "The frontend loads, but API requests fail."

Node logs contain:

```text
MongoServerSelectionError
```

`docker ps` shows:

```text
nginx      Up
mern-api   Up
mongodb    Up
```

Your API configuration contains:

```bash
MONGO_URI=mongodb://localhost:27017/mern
```

---

## Your diagnosis

The fact that the containers are `Up` does not prove the entire dependency chain works.

Inside `mern-api`:

```text
localhost
   ↓
mern-api container
```

MongoDB is a separate container.

If both are attached to:

```text
mern-network
```

then the API should normally connect using:

```text
mongodb://mongodb:27017/mern
```

This is a classic Docker networking issue. The application is trying to reach itself instead of the database container.

The root cause is not necessarily that MongoDB is down; it is often that the app's connection string is wrong for a multi-container environment.

---

## 🧪 5. Troubleshooting Sequence

### Step 1 — Check containers

```bash
docker ps
```

This confirms the services are running and whether any container unexpectedly exited or restarted.

### Step 2 — Check network

```bash
docker network inspect mern-network
```

You need to confirm that both the API and MongoDB containers are attached to the same network. If they are not on the same network, name resolution fails and the app cannot reach the database by service name.

### Step 3 — Check MongoDB logs

```bash
docker logs mongodb
```

You want to confirm whether MongoDB is healthy, initialized, listening, or producing authentication or database errors.

### Step 4 — Check API logs

```bash
docker logs mern-api
```

This helps identify whether the application fails at startup, at database connection, or during request handling.

### Step 5 — Check DNS from the API container

```bash
docker exec mern-api getent hosts mongodb
```

This confirms that the API container can resolve the MongoDB service name on the Docker network.

### Step 6 — Verify runtime configuration

Check how `MONGO_URI` is being supplied.

Common issues include:
- wrong environment variable name
- wrong URI syntax
- using `localhost` instead of the database service name
- missing port or database name
- incorrect credentials or auth source

### Step 7 — Check persistence

```bash
docker volume inspect mongo-data
```

This confirms the database volume is present and mounted to the expected path.

The big picture is:

```text
Container
   ↓
Network
   ↓
DNS
   ↓
MongoDB
   ↓
Application configuration
   ↓
Persistent data
```

Each layer matters. If one fails, the whole system may appear healthy at a superficial level while the application cannot function correctly.

---

## 🧪 6. Practical Design Assessment

Design the following:

```text
MERN application
```

with:

- Nginx
- Node.js
- MongoDB
- Docker network
- MongoDB volume
- No unnecessary public MongoDB exposure

Your target should resemble:

```text
                  Internet
                     │
                     ▼
                   Nginx
                     │
              ───────┴───────
              Docker Network
              │             │
              ▼             ▼
         Frontend        Node API
                            │
                            ▼
                         MongoDB
                            │
                            ▼
                       mongo-data
```

### Explain:

1. Which ports are public.
2. Which services communicate privately.
3. Why Node doesn't use `localhost` for MongoDB.
4. Why MongoDB needs persistent storage.
5. Why the volume isn't a backup.
6. Where runtime secrets should be supplied.

### Detailed answer

#### 1. Which ports are public?

Usually only the public-facing web entrypoint is exposed. In most MERN deployments, Nginx is the public-facing service and should publish ports like `80` or `443`.

The Node API may also publish a port externally if it is directly accessed, but in many production patterns it is only reachable through Nginx or an internal reverse proxy.

MongoDB should not be publicly exposed unless there is a special operational reason.

#### 2. Which services communicate privately?

The Node API and MongoDB should communicate on the internal Docker network. They do not need external exposure. The frontend and API may also communicate through internal networking or through Nginx depending on the deployment pattern.

#### 3. Why Node doesn't use `localhost` for MongoDB

Because `localhost` in a container refers to the container itself. MongoDB is not running inside the Node container; it is a separate container on the shared network. The correct model is to use the service name, such as `mongodb`, not the loopback address.

#### 4. Why MongoDB needs persistent storage

If the MongoDB container is recreated or deleted, its writable filesystem is lost unless data is mounted to a Docker volume. Persistent storage ensures that database files remain available across restarts and lifecycle events.

#### 5. Why the volume isn't a backup

A volume gives persistence, not recovery. It does not substitute for:
- automated snapshots
- database backup scripts
- off-host storage
- restore testing
- retention policies

If the host is lost, the volume may also be lost unless it is stored on durable and replicated external storage.

#### 6. Where runtime secrets should be supplied

Secrets like database passwords, API keys, JWT secrets, and credentials should be injected at runtime via environment variables, secret files, or a proper secrets manager. They should not be baked into the image or committed to source control.

---

## 🎤 7. Mock DevOps Interview

### Interviewer:

> "Explain how Docker networking works for a MERN application."

A strong answer should cover:

```text
Containers can join Docker networks
        ↓
Containers on the same network communicate privately
        ↓
Docker provides internal DNS
        ↓
Services can use names such as mongodb
        ↓
Only required ports are published externally
```

The idea is not that all containers are web-exposed. Instead, Docker networks help internal services discover and talk to each other cleanly and securely.

---

### Interviewer:

> "Why shouldn't you use `localhost` for MongoDB?"

Strong answer:

> Because each container has its own network namespace. From inside the Node container, `localhost` refers to the Node container itself. MongoDB should instead be addressed using its Docker network hostname, such as `mongodb`.

This is one of the most common Docker interview questions because it reveals whether someone understands container isolation and service discovery.

---

### Interviewer:

> "What's the difference between a volume and a backup?"

Strong answer:

> A volume provides persistent storage independently of a container's lifecycle. A backup is a separate recovery mechanism designed to restore data after corruption, accidental deletion, host loss, or another failure.

This distinction is critical in real systems. A database can be persistent and still be unrecoverable without a proper backup process.

---

## 🏗️ 8. Month 2 Project Checkpoint

## Containerized Production MERN Application

At this point your project should contain:

```text
mern-project/
│
├── frontend/
│   ├── Dockerfile
│   └── .dockerignore
│
├── backend/
│   ├── Dockerfile
│   └── .dockerignore
│
├── nginx/
│   └── configuration
│
└── documentation/
    ├── architecture
    ├── networking
    ├── storage
    └── troubleshooting
```

The runtime architecture should document:

```text
Public
  │
  ▼
Nginx
  │
  ▼
Private Docker network
  │
  ├── Frontend
  ├── Node API
  └── MongoDB
           │
           ▼
       Persistent volume
```

This is the shape of a production-ready containerized deployment: a public entrypoint, internal private communication, and durable data management.

---

## 🎯 9. Production Checklist

Before considering the Docker deployment ready, verify:

- Images build successfully.
- `.dockerignore` excludes unnecessary files.
- Dependency layers are cache-friendly.
- Runtime images don't contain unnecessary build tooling.
- Containers don't run as root where unnecessary.
- Secrets aren't baked into images.
- Containers share the intended network.
- Node uses the MongoDB service/container hostname.
- MongoDB isn't unnecessarily publicly exposed.
- MongoDB uses persistent storage.
- A separate backup strategy exists.
- Health endpoints work.
- Container logs are accessible.
- Failure scenarios have been tested.

This checklist reflects real-world engineering discipline. A containerized system is not considered production-ready just because it starts. It also needs secure configuration, network correctness, and durable data handling.

---

## 🎤 10. Final Interview Challenge

Answer this aloud:

> **"A Node.js container is healthy, MongoDB is healthy, but Node cannot connect to MongoDB. What are the first five things you check?"**

A strong response:

```text
1. Are both containers on the same Docker network?
2. Is the MongoDB hostname correct?
3. Is the Node URI using mongodb:27017 rather than localhost?
4. Does Docker DNS resolve mongodb from the Node container?
5. What do the Node and MongoDB logs say?
```

Then continue into:

```text
Authentication
Database configuration
Firewall/network rules
Persistent storage
Application startup configuration
```

This is exactly the kind of layered debugging an effective DevOps engineer does in production: verify the network path, the configuration, the logs, and the operational state rather than jumping straight to random restarts.

---

## 📖 11. Week 4 Docker Takeaways

You now understand the essential Docker foundation:

```text
Images
   ↓
Containers
   ↓
Ports
   ↓
Layers
   ↓
Caching
   ↓
.dockerignore
   ↓
Networks
   ↓
Volumes
   ↓
MERN architecture
```

The most important production lessons so far:

> **Build artifacts should be reproducible.**

> **Containers should contain only what they need.**

> **Container-to-container communication should use the container network, not accidental host assumptions.**

> **Persistent data must have an explicit storage and backup strategy.**

These are not just theory points. They are the foundation for building reliable deployment systems in Docker and beyond.

