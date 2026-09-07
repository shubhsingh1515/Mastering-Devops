# Day 30 - Assignment: Production-Ready MERN Docker Stack

## Objective

Build and document a reproducible Dockerized MERN application that demonstrates the concepts from Days 21-29: images, caching, networking, volumes, Compose, security, registries, logging, monitoring, and troubleshooting.

---

## Part 1: Prepare the Repository

Create:

```text
mern-docker/
+-- frontend/
+-- backend/
+-- nginx/
+-- compose.yaml
+-- .dockerignore
+-- .gitignore
+-- .env.example
+-- README.md
```

Do not commit real credentials. Use placeholder values in `.env.example`.

---

## Part 2: Containerize the Backend

Create a backend Dockerfile that:

- uses an appropriate maintained Node base image
- sets `WORKDIR`
- copies `package*.json` before source
- uses `npm ci --omit=dev` where appropriate
- copies runtime source
- runs as a non-root user
- exposes the application port
- starts the API with an exec-form `CMD`

Example shape:

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
USER node
EXPOSE 3000
CMD ["node", "src/server.js"]
```

The API must listen on `0.0.0.0`, not only `127.0.0.1`, and must provide `/health`.

---

## Part 3: Containerize the Frontend

Use either:

- a multi-stage Node build followed by an Nginx runtime image, or
- the frontend container pattern appropriate for your framework.

The final runtime image should not include unnecessary build tools or development dependencies.

Document why you selected the base image and how the build artifacts reach the runtime stage.

---

## Part 4: Add Compose Services

Define at least:

- `frontend`
- `api`
- `mongodb`

Requirements:

- all required services use the custom `app-network`
- MongoDB mounts `mongo-data:/data/db`
- API uses `mongodb://mongodb:27017/mern`
- MongoDB has no unnecessary host port mapping
- the API has a health check
- MongoDB has a health check compatible with the selected image
- service dependencies are documented

Start with:

```bash
docker compose config
docker compose up -d --build
docker compose ps
```

---

## Part 5: Verify Networking

Run:

```bash
docker network ls
docker network inspect <project>_app-network
docker compose exec api getent hosts mongodb
docker compose exec api printenv MONGO_URI
```

Expected database hostname:

```text
mongodb
```

Explain why this is wrong inside the API container:

```text
mongodb://localhost:27017/mern
```

---

## Part 6: Verify Persistence

1. Create a test document.
2. Remove the MongoDB container:

```bash
docker compose rm -sf mongodb
```

3. Recreate MongoDB:

```bash
docker compose up -d mongodb
```

4. Confirm that the document remains.
5. Document why this test proves persistence but not disaster recovery.

Then describe your independent MongoDB backup and restore process.

---

## Part 7: Apply Security Controls

Verify:

```bash
docker compose exec api whoami
docker inspect <api-container>
```

Apply where compatible:

- non-root user
- `.dockerignore`
- no secrets in the Dockerfile
- `.env` excluded from Git
- minimal suitable image
- dropped capabilities
- read-only filesystem
- explicit writable `/tmp` if required
- CPU and memory limits
- minimal public ports

Record any control that could not be enabled and explain the application compatibility reason.

---

## Part 8: Build, Tag, and Publish

Build a traceable image:

```bash
docker build -t mern-api:abc123 ./backend
```

Tag it:

```bash
docker tag mern-api:abc123 registry.example.com/team/mern-api:abc123
```

Push it if you have an authorized registry:

```bash
docker login registry.example.com
docker push registry.example.com/team/mern-api:abc123
```

If no registry is available, document the exact commands and complete the build and tag locally.

Explain why production should not rely on `latest` and how the image is connected to a Git commit or digest.

---

## Part 9: Observe and Troubleshoot

Run:

```bash
docker compose logs --tail 100 api
docker compose logs -f api
docker stats --no-stream
docker system df
docker inspect <api-container>
curl http://localhost:3000/health
```

Create an incident hypothesis for each scenario:

1. API is `Up`, but requests fail.
2. API exits with code `137`.
3. MongoDB cannot be resolved by name.
4. Disk usage is full.
5. API restarts continuously.

For each, list evidence, likely causes, immediate recovery, and long-term prevention.

---

## Part 10: Create `health-check.sh`

The script must:

1. Check whether the API container is running.
2. Check whether MongoDB is running.
3. Print both statuses.
4. Return a non-zero status if either is not running.

Bonus: test `/health` and API-to-MongoDB connectivity rather than only checking process state.

---

## Part 11: Write the Project README

Document:

- architecture diagram
- how to start and stop the stack
- environment variables
- public and internal ports
- Docker networks
- MongoDB volume
- backup strategy
- health checks
- log commands
- resource monitoring
- security decisions
- image build and tagging
- registry workflow
- rollback process
- troubleshooting steps

The README should enable another developer to clone the repository and operate the project without guessing.

---

## Part 12: Interview Assessment

Answer aloud:

> Explain how you containerized and deployed a MERN application.

Your answer should cover:

1. Separate frontend, API, and database services.
2. Compose for repeatable orchestration.
3. Service-name DNS for API-to-MongoDB communication.
4. Named volume and independent database backups.
5. Optimized and non-root application images.
6. Runtime secrets instead of baked credentials.
7. Health checks, logs, and resource monitoring.
8. Registry image tags and immutable deployment references.
9. Same tested artifact promoted from staging to production.
10. Rollback to a previous known-good image.

---

## Submission Checklist

- [ ] Frontend container created
- [ ] Backend container created
- [ ] MongoDB service configured
- [ ] Compose file validates
- [ ] Custom network configured
- [ ] MongoDB volume configured
- [ ] API uses `mongodb`, not `localhost`
- [ ] Health endpoint works
- [ ] Health checks are configured
- [ ] Images use cache-friendly Dockerfiles
- [ ] `.dockerignore` and `.gitignore` exist
- [ ] Secrets are not committed or baked into images
- [ ] API runs as non-root where practical
- [ ] Logs are accessible
- [ ] Resource monitoring was performed
- [ ] Image was tagged with a version or commit identifier
- [ ] Registry workflow was completed or documented
- [ ] Persistence was tested
- [ ] Rollback was documented
- [ ] `health-check.sh` was created
- [ ] Project README is complete
