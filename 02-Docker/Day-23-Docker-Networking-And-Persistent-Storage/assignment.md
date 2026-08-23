# Day 23 — Assignment: Docker Networking & Persistent Storage

## Objective

Understand how Docker containers communicate on the same private network and how database data persists across container lifecycle events.

---

## Part 1: Understand the Problem

You have two containers:

```text
mern-api
mongodb
```

The Node app is configured like this:

```bash
MONGO_URI=mongodb://localhost:27017/mern
```

### Question

Why does this fail when MongoDB is running in a different container?

### Expected answer

Because inside the Node container, `localhost` refers to the Node container itself, not the MongoDB container. For container-to-container communication, use the MongoDB service name (for example, `mongodb`) on a shared Docker network.

---

## Part 2: Create a Shared Docker Network

Run:

```bash
docker network create mern-network
```

Then verify:

```bash
docker network ls
```

### Explanation

A user-defined Docker network allows containers to communicate using service/container names rather than host IPs or `localhost`.

---

## Part 3: Run MongoDB on the Network

```bash
docker run -d \
  --name mongodb \
  --network mern-network \
  mongo
```

### Question

Should you publish MongoDB port `27017` to the host in this case?

### Expected answer

Usually no. The API can reach MongoDB over the private Docker network, so there is no need to publish `27017` to the host unless a special requirement exists.

---

## Part 4: Run the API on the Same Network

```bash
docker run -d \
  --name mern-api \
  --network mern-network \
  -p 3000:3000 \
  -e MONGO_URI=mongodb://mongodb:27017/mern \
  mern-api:1.1
```

### Question

What hostname should the Node app use?

### Expected answer

Use:

```text
mongodb
```

not `localhost`.

The correct URI is:

```bash
mongodb://mongodb:27017/mern
```

---

## Part 5: Persist MongoDB Data

Create a Docker volume:

```bash
docker volume create mongo-data
```

Run MongoDB with the volume:

```bash
docker run -d \
  --name mongodb \
  --network mern-network \
  -v mongo-data:/data/db \
  mongo
```

### Question

Why is this necessary?

### Expected answer

Without a volume, the database data is stored in the container filesystem, which is lost when the container is removed. A Docker volume keeps the data independent from the container lifecycle.

---

## Part 6: Explain the Architecture

Draw the final architecture:

```text
Host
  │
  ▼
mern-api
  │
  │ mongodb:27017
  ▼
mongodb
  │
  ▼
mongo-data volume
```

Your explanation should include:

- the API is reachable from the host via `3000`
- MongoDB stays on the private Docker network
- MongoDB is not unnecessarily exposed to the public
- the database data is persisted in the named volume

---

## Part 7: Persistence vs Backup

### Question

If MongoDB data is persisted in a Docker volume, is that enough for a production backup strategy?

### Expected answer

No. A volume gives persistence, not a disaster-recovery plan. You still need a backup strategy such as regular database dumps, off-host storage, retention policies, and restore testing.

---

## Part 8: Troubleshooting Practice

Suppose the app still cannot connect.

Check the following in order:

```bash
docker ps
docker network inspect mern-network
docker logs mongodb
docker logs mern-api
```

Then also test from inside the API container:

```bash
docker exec -it mern-api sh
getent hosts mongodb
```

### What are you looking for?

- both containers running
- both containers attached to the same network
- correct service name `mongodb`
- correct MongoDB port
- correct URI syntax
- successful DNS resolution

---

## Final Deliverable

Write a short paragraph explaining:

> Why `localhost` fails in Docker for inter-container communication, how to create a shared network, how to persist MongoDB data, and why a volume is not the same as a backup.

---

## Expected High-Quality Answer

The API should not use `localhost` when connecting to MongoDB in another container because `localhost` points to the current container itself. The fix is to create a shared Docker network and run both the Node API and MongoDB on that network. Then the application should use the database container name, such as `mongodb`, in the connection string instead of `localhost`. MongoDB should mount a named Docker volume such as `mongo-data` to `/data/db`, so the database survives container deletion and restart. However, a Docker volume is only for persistence; it is not a full backup strategy. Production systems still need regular backups, off-host storage, validation, and disaster recovery testing.
