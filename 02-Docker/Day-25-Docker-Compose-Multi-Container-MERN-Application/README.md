# DevOps Mentorship Program - Day 25

## Phase 2: Docker & Containers

### Topic: Docker Compose - Running a Multi-Container MERN Application



## Learning Objectives

By the end of this lesson, you should be able to:

- Explain what Docker Compose solves
- Define multiple application services in `compose.yaml`
- Understand the difference between a Compose service and a container
- Connect services through Compose networking and service-name DNS
- Attach a named volume for persistent MongoDB data
- Pass runtime configuration through environment variables
- Start, inspect, log, rebuild, and stop an entire stack
- Explain why `depends_on` is not the same as readiness
- Add a basic health check
- Troubleshoot a Compose-based MERN deployment
- Describe a production-style Compose architecture in an interview

---

## 1. Review From Earlier Lessons

### Q1. Inside a Node container, what does `localhost` refer to?

It refers to the Node container itself. It does not refer to MongoDB or automatically refer to the Docker host.

### Q2. How should Node connect to a MongoDB container named `mongodb`?

```text
mongodb://mongodb:27017/mern
```

The hostname is the Compose service name, not `localhost` and not a temporary container IP address.

### Q3. What does this port mapping do?

```bash
docker run -p 8080:3000 app
```

```text
Host port 8080
       |
       v
Container port 3000
```

Users connect to port `8080` on the host. The application listens on port `3000` inside the container.

### Q4. What is the difference between a volume and a backup?

A volume keeps data available across container replacement. A backup is an independent recovery mechanism used after deletion, corruption, or a larger failure.

### Q5. Why copy `package*.json` before application source in a Dockerfile?

Docker can cache the dependency-installation layer when only source files change. This makes later image builds faster.

---

## 2. The Problem Docker Compose Solves

A real MERN application may include several services:

```text
Nginx
Node API
MongoDB
Redis
Background worker
React frontend
```

Starting each container manually requires repeated management of:

- Networks
- Volumes
- Environment variables
- Port mappings
- Restart policies
- Service dependencies
- Build commands

Docker Compose lets you describe the application stack in one declarative configuration file. The file becomes a repeatable definition of the application's topology.

```text
compose.yaml
      |
      v
Application stack
      |
  +---+----------+
  v   v          v
Nginx API     MongoDB
```

Compose does not remove the need to understand Docker. It organizes the containers, networks, volumes, and configuration that you would otherwise create separately.

---

## 3. A Basic Compose File

Create a file named `compose.yaml`:

```yaml
services:
  api:
    build: ./backend
    ports:
      - "3000:3000"

  mongodb:
    image: mongo:8
```

This defines two services:

- `api`: built from the Dockerfile in `./backend`
- `mongodb`: created from the MongoDB image

When the stack starts, Compose creates containers for these services and creates a default application network. Both services can communicate on that network.

Validate the file before starting anything:

```bash
docker compose config
```

This command renders the resolved configuration and reports many YAML or Compose configuration errors early.

---

## 4. Services Versus Containers

`api` and `mongodb` are service definitions. A service describes how Compose should create and run a logical application component.

When you run Compose, it creates containers from those definitions:

```text
Compose service definition
            |
            v
         Container
```

A service is the logical component. A container is one running instance of that component. If an API is scaled, one service can have multiple containers:

```text
api service
     |
 +---+---+---+
 v   v   v
API1 API2 API3
```

This distinction matters when reading status output, scaling services, and designing deployments.

---

## 5. Compose Networking and Service-Name DNS

Compose places services in a shared default network unless you configure another network. Docker provides internal DNS for services on that network.

If the service is named `mongodb`, the API can connect to:

```text
mongodb:27017
```

The API should use:

```text
mongodb://mongodb:27017/mern
```

It should not use:

```text
mongodb://localhost:27017/mern
```

Inside the API container, `localhost` means the API container. It does not mean the MongoDB container.

Do not hardcode an address such as:

```text
mongodb://172.18.0.5:27017/mern
```

Container IP addresses can change when the stack is recreated. The service name is the stable logical endpoint provided by Compose DNS.

Host-to-container port publishing and container-to-container communication are separate concepts. The API may need `3000:3000` so a developer can reach it from the host, but MongoDB does not need `27017:27017` for the API to reach it internally.

---

## 6. Persistent MongoDB Storage

Containers are replaceable. Database data should not depend on the writable layer of a disposable MongoDB container.

Add a named volume:

```yaml
services:
  mongodb:
    image: mongo:8
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

The mapping means:

```text
MongoDB container: /data/db
             |
             v
Docker named volume: mongo-data
```

The named volume normally remains when a container is removed and can be mounted by its replacement.

A volume is not a backup. It can still be deleted, corrupted, encrypted by malware, or lost with the storage system. Production systems also need backup, restore testing, retention, and disaster-recovery planning.

---

## 7. Complete MERN-Style Compose Example

A simplified backend and database stack can look like this:

```yaml
services:
  api:
    build: ./backend
    environment:
      NODE_ENV: production
      MONGO_URI: mongodb://mongodb:27017/mern
    ports:
      - "3000:3000"
    depends_on:
      mongodb:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/health', response => process.exit(response.statusCode === 200 ? 0 : 1)).on('error', () => process.exit(1))"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 15s

  mongodb:
    image: mongo:8
    volumes:
      - mongo-data:/data/db
    healthcheck:
      test: ["CMD-SHELL", "mongosh --quiet --eval 'db.adminCommand(\"ping\").ok' | grep 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

  frontend:
    build: ./frontend
    ports:
      - "8080:80"
    depends_on:
      api:
        condition: service_healthy

volumes:
  mongo-data:
```

Important details:

- `build` tells Compose where to find a service Dockerfile and build context.
- `image` uses a prebuilt image from a registry or local cache.
- `environment` provides runtime configuration without rebuilding the image.
- `ports` publishes selected container ports to the host.
- `depends_on` expresses a dependency relationship.
- `condition: service_healthy` waits for the declared health check in Compose implementations that support this condition.
- `healthcheck` defines a command whose exit code represents health.
- `mongo-data:/data/db` persists MongoDB files.

Health check commands must match the tools available in the selected image. A health check is useful only when its command tests a meaningful condition.

---

## 8. Why `depends_on` Is Not Enough

A short dependency declaration is common:

```yaml
depends_on:
  - mongodb
```

This expresses startup ordering or dependency metadata. It does not always prove that MongoDB has finished initialization and can accept connections.

The possible sequence is:

```text
MongoDB container starts
          |
          v
MongoDB is still initializing
          |
          v
API starts and connects
          |
          v
Connection temporarily fails
```

A robust application combines infrastructure checks with application behavior:

- Define a useful MongoDB health check
- Use a health-aware dependency condition where supported
- Add connection retry and backoff logic in the API
- Log connection failures clearly
- Check service logs during startup

Readiness is an application concern as well as a container concern. Even a healthy database can become unavailable later, so the API should handle reconnects and transient failures.

---

## 9. Start, Inspect, and Stop the Stack

Run these commands from the directory containing `compose.yaml`:

```bash
docker compose config
docker compose build
docker compose up -d --build
docker compose ps
docker compose logs
docker compose logs -f api
docker compose exec api sh
docker compose down
```

The usual workflow is:

```text
Validate -> Build -> Start -> Check status -> Read logs -> Test -> Stop
```

`docker compose down` removes the Compose-created containers and network. It does not normally remove named volumes unless you explicitly add `--volumes`.

Be careful with:

```bash
docker compose down --volumes
```

That removes the stack's named volumes and can delete local database data.

---

## 10. Inspect the Stack

The generated network name depends on the Compose project name. Use Compose itself to understand the resolved project:

```bash
docker compose ps
docker compose config
docker network ls
docker volume ls
```

For deeper inspection:

```bash
docker compose ps -a
docker compose logs api
docker compose logs mongodb
docker compose exec api sh
docker inspect <container-id-or-name>
```

If the API cannot connect to MongoDB, check:

1. Both services are running
2. The API uses `mongodb` as the hostname
3. The services share a network
4. MongoDB logs show successful initialization
5. The `MONGO_URI` value is present inside the API container
6. The API retries transient startup failures
7. The MongoDB internal port is `27017`

---

## 11. Production-Style MERN Thinking

A development stack might publish the API and frontend directly:

```text
Host :3000 -> API
Host :8080 -> Frontend
```

A more intentional production topology often uses Nginx as the public entry point:

```text
Internet
   |
   v
Nginx - public entry point
   |
   +--> Frontend
   |
   +--> API
          |
          v
       MongoDB
          |
          v
    Persistent volume
```

Keep internal services private whenever possible. In particular, MongoDB should not be published to the host or Internet unless there is a specific, controlled operational requirement.

Typical production concerns include:

- TLS termination and secure headers at Nginx
- Private network paths for API-to-database traffic
- Secret management outside the Compose file
- Health checks and restart policies
- Resource limits and logging
- MongoDB authentication and authorization
- Tested database backups
- Separate development and production configuration

Non-sensitive settings can be passed through environment variables. Sensitive credentials should come from a suitable secret-management mechanism and should not be committed to source control.

---

## 12. Hands-On Lab Summary

Create this structure:

```text
day25/
├── compose.yaml
└── backend/
    ├── Dockerfile
    └── server.js
```

The backend should expose a `/health` endpoint and listen on `0.0.0.0:3000`. The Compose file should:

- Build the API from `./backend`
- Run MongoDB as a separate service
- Set `MONGO_URI` to `mongodb://mongodb:27017/mern`
- Attach `mongo-data` to `/data/db`
- Add a MongoDB health check
- Start the stack with `docker compose up -d --build`
- Verify the status with `docker compose ps`
- Test the API with `curl http://localhost:3000/health`

Remember that the host uses `localhost:3000`, while the API container uses `mongodb:27017` to reach MongoDB.

---

## Key Takeaways

- Compose defines and runs a multi-container application declaratively.
- A service is a logical definition; a container is a running instance.
- Services on the same Compose network find each other through service names.
- `localhost` inside a container refers to that container.
- Named volumes keep MongoDB data outside the disposable container lifecycle.
- Volumes provide persistence, not independent backups.
- `depends_on` expresses dependency but is not a complete readiness strategy.
- Health checks and application retry logic work together.
- Production architectures should expose only intentional public entry points.
- Compose should describe application topology while important data remains durable.
