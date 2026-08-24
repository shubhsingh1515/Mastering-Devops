# Day 25 - Assignment: Docker Compose MERN Stack

## Objective

Use Docker Compose to define and operate a small multi-container application consisting of a Node API and MongoDB. Practice service-name DNS, persistent storage, environment configuration, health checks, and troubleshooting.

## Prerequisites

You need:

- Docker Desktop or Docker Engine with the Compose plugin
- A backend directory with a working Node application
- A `/health` endpoint that returns HTTP 200
- The backend listening on `0.0.0.0:3000`

## Part 1: Create the Project Structure

```text
day25/
├── compose.yaml
└── backend/
    ├── Dockerfile
    └── server.js
```

## Part 2: Define the Services

Create `compose.yaml` with an `api` service and a `mongodb` service.

Requirements:

- Build the API from `./backend`
- Publish API port `3000`
- Use the MongoDB image
- Set `MONGO_URI` to `mongodb://mongodb:27017/mern`
- Do not publish MongoDB port `27017` unless you have a specific reason

Start with:

```yaml
services:
  api:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      MONGO_URI: mongodb://mongodb:27017/mern

  mongodb:
    image: mongo:8
```

Validate it:

```bash
docker compose config
```

## Part 3: Add Persistent Storage

Add a named volume at the MongoDB data directory:

```yaml
services:
  mongodb:
    image: mongo:8
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

### Question

What would happen to database files stored only in the container's writable layer when the container is removed?

### Expected answer

They would normally be lost with the removed container. The named volume keeps the data separate from the container lifecycle.

## Part 4: Add a Health Check

Add a MongoDB health check using a command available in the selected MongoDB image:

```yaml
healthcheck:
  test: ["CMD-SHELL", "mongosh --quiet --eval 'db.adminCommand(\"ping\").ok' | grep 1"]
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 20s
```

Then add a dependency condition to the API where supported:

```yaml
depends_on:
  mongodb:
    condition: service_healthy
```

### Important

This improves startup coordination, but the API should still retry database connections. A health check is not a replacement for resilient application behavior.

## Part 5: Start and Verify

```bash
docker compose up -d --build
docker compose ps
curl http://localhost:3000/health
```

Record:

- The API container status
- The MongoDB container status
- The health endpoint response
- The named volume shown by `docker volume ls`

## Part 6: Inspect Service Discovery

Run:

```bash
docker compose exec api printenv MONGO_URI
docker compose logs api
docker compose logs mongodb
```

### Question

Why must the API use `mongodb` instead of `localhost`?

### Expected answer

`localhost` inside the API container refers to the API container itself. `mongodb` is the service name resolved through Docker's internal network DNS.

## Part 7: Test Persistence

1. Create or insert a test record in MongoDB.
2. Remove only the MongoDB container:

```bash
docker compose rm -s mongodb
```

3. Recreate the service:

```bash
docker compose up -d mongodb
```

4. Confirm that the test record still exists.

### Question

Why does the record remain?

### Expected answer

The database files are mounted in the named `mongo-data` volume, which remains after the container is removed.

## Part 8: Troubleshooting Scenario

Change the API configuration temporarily to:

```text
mongodb://localhost:27017/mern
```

Restart the API:

```bash
docker compose up -d --force-recreate api
```

Observe the failure in the logs:

```bash
docker compose logs api
```

Restore the correct hostname:

```text
mongodb://mongodb:27017/mern
```

Recreate the API and confirm recovery.

## Part 9: Production Design Questions

Answer in your own words:

1. Why should MongoDB usually remain unpublished to the host?
2. Why is a volume not the same as a backup?
3. Why should real credentials not be committed to `compose.yaml`?
4. What does `depends_on` communicate, and what does it not guarantee?
5. What role could Nginx play in a production-style MERN architecture?

## Part 10: Submission Checklist

- [ ] `compose.yaml` validates with `docker compose config`
- [ ] The API and MongoDB services start together
- [ ] The API uses `mongodb` as the database hostname
- [ ] MongoDB uses a named volume mounted at `/data/db`
- [ ] MongoDB has a meaningful health check
- [ ] `/health` returns HTTP 200 through `localhost:3000`
- [ ] API and MongoDB logs were inspected
- [ ] Persistence was tested after MongoDB container replacement
- [ ] You can explain why a volume is not a backup
- [ ] You can explain why `depends_on` is not the same as readiness
