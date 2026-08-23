# DevOps Mentorship Program — Day 23

## 🐳 Phase 2: Docker & Containers

### 🟢 Topic: Docker Networking & Persistent Storage


**Docker Phase:** 3 / ~10

---

## 🔁 1. Quick Review

### Q1. What is the difference between a Docker image and a container?

```text
Image     → packaged artifact
Container → running instance of an image
```

An image is a reusable blueprint containing the application, runtime, libraries, and filesystem. A container is a running instance created from that image.

### Q2. What does this mean?

```bash
docker run -p 8080:3000 myapp
```

This maps the host port `8080` to the container port `3000`.

```text
Host :8080
    ↓
Container :3000
```

The user accesses the app on the host machine via `localhost:8080`, while the application inside the container listens on port `3000`.

### Q3. Why is this Dockerfile ordering useful?

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

This order allows Docker to cache dependency installation. If only the application source changes, Docker can reuse the installed dependency layer instead of reinstalling everything.

### Q4. Why should production secrets not be baked into a Docker image?

Because images may be shared, stored, copied, or inspected. If a secret is baked into the image, it may become difficult to revoke and easy to leak accidentally.

### Q5. What command shows Docker container logs?

```bash
docker logs <container>
```

---

## 1. 📚 Topic: Docker Networking

A single container is easy to reason about:

```text
Node container
      │
      ▼
Host port
```

But in a real MERN application, you usually have several services:

```text
React
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

When these become separate containers, they need a way to talk to each other. That is where Docker networking becomes essential.

---

## 2. 🧠 The Critical localhost Rule

This is one of the most important Docker concepts.

Suppose you have:

```text
node-api container
mongodb container
```

Inside the Node container, this:

```text
localhost
```

means:

```text
The Node container itself.
```

It does not mean the MongoDB container.

So this is usually wrong:

```bash
MONGO_URI=mongodb://localhost:27017/mydb
```

when MongoDB is running in another container.

### Correct pattern on a user-defined Docker network

```bash
MONGO_URI=mongodb://mongodb:27017/mydb
```

The key idea is:

```text
Node container
     │
     │ mongodb:27017
     ▼
MongoDB container
```

The container name `mongodb` is the DNS name available inside the Docker network.

### Why this matters

`localhost` is always relative to the current container. It never points to another container by default.

This is a common source of the error:

```text
MongoServerSelectionError
```

because the application is trying to reach itself instead of the database container.

---

## 3. 🌐 Create a Docker Network

Create a custom Docker network:

```bash
docker network create mern-network
```

List networks:

```bash
docker network ls
```

Inspect a network:

```bash
docker network inspect mern-network
```

Now run MongoDB on that network:

```bash
docker run -d \
  --name mongodb \
  --network mern-network \
  mongo
```

Run the API on the same network:

```bash
docker run -d \
  --name mern-api \
  --network mern-network \
  -p 3000:3000 \
  mern-api:1.1
```

Now the API can reach MongoDB with:

```text
mongodb:27017
```

---

## 4. 🔌 Container Port vs Host Port

Suppose MongoDB listens inside its container on:

```text
27017
```

Your Node application does not need MongoDB's port published to the host for container-to-container communication.

This is valid:

```text
Node
 │
 │ mongodb:27017
 ▼
MongoDB
```

without this:

```bash
-p 27017:27017
```

### Public exposure concept

Only expose ports that must be reachable from outside the Docker network.

Example architecture:

```text
Internet
   │
   ▼
Nginx :443
   │
Docker network
   │
Node
   │
MongoDB
```

MongoDB usually does not need to be publicly exposed.

### Production rule

- Publish only the frontend or API ports needed by users
- Keep database ports internal unless there is a specific requirement
- Use private network communication between application containers

---

## 5. 🏗️ MERN Docker Network

A simplified production setup looks like this:

```text
                    Internet
                       │
                       ▼
                  Nginx :443
                       │
                 mern-network
                       │
              ┌────────┴────────┐
              ▼                 ▼
          React/Nginx       Node API
                                 │
                                 ▼
                              MongoDB
```

This is better than exposing every service directly. Containers communicate through internal Docker networking instead of exposing every service to the internet.

---

## 6. 🔍 Test Container Connectivity

You can inspect a container with:

```bash
docker inspect mern-api
```

Look at the container's network configuration.

You can also enter a running container:

```bash
docker exec -it mern-api sh
```

From inside that container, you can test connectivity and DNS.

Depending on the base image, tools like `curl`, `wget`, `getent`, or DNS utilities may or may not be installed.

Example:

```bash
getent hosts mongodb
```

If DNS is working, you should see an address for the MongoDB container.

---

## 7. 🚨 Common MERN Docker Failure

Your Node application logs show:

```text
MongoServerSelectionError
```

You check:

```bash
docker ps
```

Both containers are running, so the issue is not simply the absence of a service.

Then you discover the configuration is:

```bash
MONGO_URI=mongodb://localhost:27017/mydb
```

### Why it fails

Inside the Node container:

```text
localhost
```

refers to the Node container itself, not MongoDB.

### Fix

Place both containers on the same Docker network and use:

```bash
mongodb://mongodb:27017/mydb
```

This correctly resolves the MongoDB service by container name.

---

## 8. 📦 Persistent Storage

Networking solves communication, but data durability is another problem.

A container's writable filesystem is not a good place to store a production database long-term.

If the MongoDB container is deleted:

```bash
docker rm mongodb
```

you do not want your production database to disappear with it.

You need persistent storage.

---

## 9. 💾 Docker Volumes

Create a named volume:

```bash
docker volume create mongo-data
```

List volumes:

```bash
docker volume ls
```

Run MongoDB with that volume attached:

```bash
docker run -d \
  --name mongodb \
  --network mern-network \
  -v mongo-data:/data/db \
  mongo
```

The important mapping is:

```text
MongoDB container
       │
       ▼
 /data/db
       │
       ▼
 Docker volume
       │
       ▼
Persistent data
```

This means the database writes are stored outside the container filesystem.

---

## 10. 🧠 Container vs Data Lifecycle

### Without persistent storage

```text
MongoDB container
      │
      ▼
container deleted
      │
      ▼
data may be lost
```

### With a Docker volume

```text
MongoDB container
      │
      ▼
Docker volume
      │
      ▼
container deleted
      │
      ▼
volume remains
```

This allows a new MongoDB container to mount the same volume and continue using the data.

---

## 11. ⚠️ Volume ≠ Backup

This is a very important interview distinction.

A Docker volume provides persistence.

It does not automatically provide a backup strategy.

You still need:

```text
Database
   ↓
Backup
   ↓
Off-host / durable storage
   ↓
Restore testing
```

For MongoDB, you might use `mongodump`, or a managed database backup process, depending on architecture.

### Important idea

Persistence is about keeping data alive across container restarts or removals.

Backup is about being able to recover from system failures, corruption, host loss, accidental deletion, or disaster recovery scenarios.

---

## 12. 🌍 Production MERN Architecture

A better production model is:

```text
                 Internet
                    │
                    ▼
                 Nginx
                    │
              ┌─────┴─────┐
              ▼           ▼
          Frontend      Node API
                           │
                           ▼
                        MongoDB
                           │
                           ▼
                     Persistent data
```

And separately:

```text
MongoDB
   │
   ├── Persistent storage
   │
   └── Backup strategy
```

This separation keeps the database isolated, durable, and recoverable.

---

## 13. 🧪 Hands-on Lab

Create the Docker network:

```bash
docker network create mern-network
```

Create the volume:

```bash
docker volume create mongo-data
```

Run MongoDB:

```bash
docker run -d \
  --name mongodb \
  --network mern-network \
  -v mongo-data:/data/db \
  mongo
```

Check running containers:

```bash
docker ps
```

Inspect the volume:

```bash
docker volume inspect mongo-data
```

Inspect the network:

```bash
docker network inspect mern-network
```

---

## 14. 🔗 Connect Your Node API

Run the API with environment variables:

```bash
docker run -d \
  --name mern-api \
  --network mern-network \
  -p 3000:3000 \
  -e MONGO_URI=mongodb://mongodb:27017/mydb \
  mern-api:1.1
```

The result is:

```text
Host :3000
    │
    ▼
Node container
    │
    │ mongodb:27017
    ▼
MongoDB container
    │
    ▼
mongo-data volume
```

Notice:

- `3000` is published because you want the API reachable outside the Docker network
- `27017` does not need to be published to the host for Node-to-MongoDB communication

---

## 15. 🔐 Environment Variables

Instead of writing:

```dockerfile
ENV MONGO_URI=mongodb://...
```

you can set configuration at runtime:

```bash
-e MONGO_URI=...
```

This is useful for local experimentation and deployment flexibility.

### Best practice

For production, use a proper configuration and secrets management strategy instead of embedding sensitive values in shell history, source code, or Dockerfiles.

The principle is:

```text
Image
  ↓
Reusable artifact

Runtime configuration
  ↓
Environment-specific values
```

The image should be portable; the environment should supply runtime values such as database addresses, ports, and secrets.

---

## 16. 🛠️ Debugging Docker Networking

If Node cannot reach MongoDB, follow this flow:

### Step 1: Are both containers running?

```bash
docker ps
```

### Step 2: Are they on the same network?

```bash
docker network inspect mern-network
```

### Step 3: Is the hostname correct?

```text
mongodb
```

### Step 4: Is MongoDB truly running?

```bash
docker logs mongodb
```

### Step 5: Is the URI correct?

```text
mongodb://mongodb:27017/mydb
```

### Step 6: Can the API resolve MongoDB?

From the API container:

```bash
getent hosts mongodb
```

This gives you a structured troubleshooting path instead of guessing.

---

## 17. 🆚 Docker Network vs Host Network

By default, containers usually use Docker-managed networking.

You may also see:

```bash
--network host
```

This makes the container share the host's network namespace instead of using Docker's isolated network layer.

### Interview answer

```text
Container networking provides isolation and controlled communication between containers, while host networking removes much of that isolation by using the host network namespace.
```

This is not the default choice for ordinary MERN deployments because it reduces isolation and can create conflicts with host ports.

---

## 18. 💼 Interview Preparation

### Beginner

#### Q1. Why doesn't localhost work for connecting from Node to MongoDB in separate containers?

Because `localhost` refers to the Node container itself, not the MongoDB container.

#### Q2. How do containers communicate on a user-defined Docker network?

They use container/service names and Docker's internal DNS.

#### Q3. What is a Docker volume?

A Docker volume is persistent storage managed by Docker that can outlive an individual container.

---

### Intermediate

#### Q4. Why shouldn't you expose MongoDB with `-p 27017:27017` unnecessarily?

Because the Node container can communicate with MongoDB over the private Docker network. Publishing the port increases exposure without giving a required benefit.

#### Q5. Does deleting a container delete a Docker volume?

Normally, removing the container does not automatically remove a separately managed named volume.

#### Q6. Is a Docker volume a backup?

No. It provides persistence, not full disaster recovery or backup safety.

---

### Advanced

#### Q7. Your Node container is running, MongoDB is running, but the API cannot connect to MongoDB. Walk me through your investigation.

A strong answer:

```text
Check both containers
        ↓
Check their network membership
        ↓
Verify MongoDB hostname
        ↓
Verify port
        ↓
Check MongoDB logs
        ↓
Check Node application logs
        ↓
Test DNS from Node container
        ↓
Verify environment configuration
        ↓
Check authentication/database configuration
```

Then mention the common mistake:

```text
mongodb://localhost:27017
```

should usually become:

```text
mongodb://mongodb:27017
```

when `mongodb` is the service/container hostname on the shared Docker network.

---

## 19. 📝 Quiz

### 1. Inside a Node container, what does localhost refer to?

- A. The MongoDB container
- B. The Docker host
- C. The Node container itself
- D. The Internet

### 2. How do two containers communicate using a user-defined Docker network?

- A. Through container/service names and Docker networking
- B. Only through public IPs
- C. Through SSH
- D. Through Nginx

### 3. Which command creates a Docker network?

- A. `docker network create`
- B. `docker create network`
- C. `docker net init`
- D. `docker connect`

### 4. Which command creates a named volume?

- A. `docker volume create`
- B. `docker storage init`
- C. `docker disk create`
- D. `docker mount create`

### 5. What is the purpose of a MongoDB volume?

- A. Expose MongoDB publicly
- B. Persist database data independently of a container's lifecycle
- C. Encrypt MongoDB automatically
- D. Replace backups

### 6. Does a volume automatically constitute a backup strategy?

- A. Yes
- B. No

### 7. Why might you avoid publishing port 27017?

- A. MongoDB cannot use TCP
- B. The application can communicate over the private Docker network
- C. Docker doesn't support 27017
- D. Nginx requires it

### 8. Which command inspects a Docker network?

- A. `docker network inspect`
- B. `docker network show-config`
- C. `docker inspect-network-config`
- D. `docker net status`

### 9. Which URI is appropriate when MongoDB's container hostname is mongodb?

- A. `mongodb://localhost:27017/mydb`
- B. `mongodb://mongodb:27017/mydb`
- C. `mongodb://127.0.0.1:80/mydb`
- D. `mongodb://docker:3000/mydb`

### 10. What is a major benefit of separating Nginx, Node, and MongoDB into containers?

- A. All services automatically become public
- B. Components can be isolated, deployed, and managed independently
- C. Backups are no longer required
- D. Security configuration becomes unnecessary

### ✅ Answer Key

```text
C
A
A
A
B
B
B
A
B
B
```

---

## 22. Practical Assessment

### Production Scenario

You have:

```text
mern-api
mongodb
```

Both containers show:

```text
Up
```

But your API logs show:

```text
MongoServerSelectionError
```

Your current configuration is:

```bash
MONGO_URI=mongodb://localhost:27017/mern
```

### Explain:

1. Why this configuration fails
2. How you would create a shared network
3. How you would connect both containers
4. What hostname the Node application should use
5. Why MongoDB doesn't need a publicly published port
6. How you would persist MongoDB data
7. Why persistence still isn't enough without backups

### Expected architecture

```text
              Docker Host
                   │
          mern-network
                   │
        ┌──────────┴──────────┐
        │                     │
   mern-api               mongodb
        │                     │
        │ mongodb:27017       │
        └─────────────────────┘
                                  │
                                  ▼
                             mongo-data
```

---

## 23. Month 2 Cumulative Project Update

Your project now needs:

### Network

```text
mern-network
```

with:

```text
nginx
node-api
mongodb
```

communicating through internal Docker networking.

### Storage

MongoDB must use:

```text
mongo-data
```

or an equivalent persistent storage design.

### Security

```text
Internet
   │
   ▼
Nginx
   │
   ▼
Node
   │
   ▼
MongoDB
```

Only required public ports should be exposed.

### Configuration

Document:

- `MONGO_URI`
- `PORT`
- `NODE_ENV`
- `JWT_SECRET`

and distinguish:

```text
configuration
vs
secrets
```

### Reliability

Your project documentation must now answer:

- What happens to MongoDB data if the MongoDB container is deleted?
- How would you restore the database after losing the host?

---

## 24. Final Interview Challenge

Answer aloud:

> "Design a Dockerized production architecture for a MERN application. Explain networking, port exposure, and database persistence."

A strong answer:

```text
Internet
   ↓
Nginx
   ↓
Private Docker network
   ├── Node API
   │     ↓
   │   MongoDB
   │     ↓
   │   Persistent volume
   │
   └── Frontend/static assets
```

Then explain:

- Nginx is the public entry point
- Node communicates with MongoDB through the internal Docker network
- Containers use service/container names rather than localhost for peer communication
- MongoDB data uses persistent storage
- MongoDB is not unnecessarily exposed publicly
- Persistent storage is complemented by an independent backup strategy

---

## 25. Today's Key Takeaways

You learned:

✅ Docker networks
✅ Container-to-container communication
✅ Why localhost causes common Docker mistakes
✅ Container/service DNS
✅ Host ports vs container ports
✅ Docker volumes
✅ MongoDB persistence
✅ Persistence vs backup
✅ Internal vs public service exposure
✅ MERN Docker networking troubleshooting
✅ Production Docker architecture

The mental model to remember:

```text
                 Internet
                    │
                    ▼
                  Nginx
                    │
             Docker network
                    │
             ┌──────┴──────┐
             ▼             ▼
           Node          Other services
             │
             ▼
          MongoDB
             │
             ▼
        Persistent data
             │
             ▼
           Backup
```

### The single Docker networking rule worth memorizing

```text
Inside a container, localhost means that container—not another container and not the Docker host.
```

---

## ✅ End-of-Day Summary

Networking and persistence are two of the most essential concepts in real-world Docker architecture. A container does not magically know how to talk to another container. You must place services on the same Docker network and use the service name as the hostname. For data durability, use Docker volumes and treat backups as a separate operational necessity.

This is the point where Docker moves from “basic container run” to “production-ready architecture thinking.”
