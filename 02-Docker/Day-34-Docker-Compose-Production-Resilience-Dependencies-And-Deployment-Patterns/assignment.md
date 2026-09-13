# Day 34 - Assignment: Resilient MERN Compose Stack

## Objective

Improve your Day 33 MERN Compose project so that service readiness is visible, dependencies are handled deliberately, failures can be diagnosed, and the API shuts down gracefully.

## Part 1: Add an API Health Endpoint

Add:

```http
GET /health
```

Return:

```json
{
  "status": "ok"
}
```

The endpoint must return HTTP `200` when the API process can serve requests.

## Part 2: Add a Compose Health Check

Add an API health check:

```yaml
healthcheck:
  test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
  interval: 30s
  timeout: 5s
  retries: 3
  start_period: 15s
```

Verify that `wget` exists in the API image. If it does not, choose a command supported by the image and document the decision.

## Part 3: Add a MongoDB Health Check

Use an image-compatible MongoDB command, for example:

```yaml
healthcheck:
  test: ["CMD-SHELL", "mongosh --quiet --eval 'db.adminCommand({ ping: 1 }).ok' | grep 1"]
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 20s
```

Verify the command manually inside the container. Do not assume that every MongoDB image has the same client executable.

## Part 4: Model Dependencies

Configure the API to depend on MongoDB's health condition where supported:

```yaml
depends_on:
  mongodb:
    condition: service_healthy
```

Configure Nginx to depend on the API health condition where supported. Explain why these conditions improve startup ordering but do not replace application retry logic.

## Part 5: Choose Restart Policies

Select and document restart policies for:

- MongoDB.
- API.
- Nginx.

Explain why you selected `on-failure`, `on-failure:5`, `unless-stopped`, or another policy. Include the risks of hiding a bug behind a restart loop.

## Part 6: Add Graceful Shutdown

Update the Node API to handle `SIGTERM` and `SIGINT`:

1. Stop accepting new traffic.
2. Allow in-flight requests to finish where possible.
3. Close the HTTP server.
4. Close the MongoDB connection.
5. Exit within a defined timeout.

Do not log credentials or full secret-bearing connection strings.

## Part 7: Verify Normal Operation

Run:

```bash
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs --tail 100 api
curl -f http://localhost/api/health
```

Record the health state of each service.

## Part 8: Break the Health Check

Temporarily change the API health path to:

```text
/incorrect-health
```

Observe:

```bash
docker compose ps
docker inspect <api-container>
docker compose logs --tail 100 api
```

Restore `/health` and verify recovery.

## Part 9: Simulate a Dependency Failure

Temporarily change:

```dotenv
MONGO_URI=mongodb://localhost:27017/mern
```

Restart the API and inspect:

```bash
docker compose up -d api
docker compose ps -a
docker compose logs --tail 100 api
docker compose logs --tail 100 mongodb
```

Explain why `localhost` points to the API container rather than MongoDB. Restore:

```dotenv
MONGO_URI=mongodb://mongodb:27017/mern
```

## Part 10: Write a Troubleshooting Runbook

Document the steps for these incidents:

### API is restarting

Include logs, exit state, environment configuration, health state, network membership, DNS, MongoDB readiness, and resource usage.

### Nginx returns `502`

Include Nginx logs, API logs, upstream DNS, API port, API health, and the complete public request path.

## Success Criteria

- `/health` returns `200` when the API is usable.
- Compose reports meaningful health state.
- Health-check commands exist in the runtime images.
- MongoDB and API dependencies are documented.
- Restart policies are selected intentionally.
- API shutdown handles `SIGTERM` cleanly.
- A broken health check can be detected and repaired.
- A wrong MongoDB hostname can be diagnosed without blindly restarting the whole stack.
- The README documents container state, application state, and dependency state separately.
