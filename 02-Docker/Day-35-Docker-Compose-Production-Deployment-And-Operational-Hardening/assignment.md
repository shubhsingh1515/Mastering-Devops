# Day 35 - Assignment: Production Compose Operational Hardening

## Objective

Harden your Day 34 MERN Compose project so that the public boundary, internal services, persistence, health signals, shutdown behavior, logs, and troubleshooting workflow are clearly documented and tested.

## Part 1: Review Service Exposure

Document and implement the intended exposure model:

```text
Nginx     -> public
API       -> internal
MongoDB   -> internal
```

Remove unnecessary host port mappings. Keep only the ports required for the public request path or an explicitly documented operational task.

## Part 2: Verify the Network

Run:

```bash
docker network ls
docker network inspect <your-network>
```

Prove that Nginx can resolve the API and that the API can resolve MongoDB using service names.

## Part 3: Verify Persistence

Document:

```text
MongoDB /data/db -> mongo-data volume
```

Explain the difference between persistence and backup. Do not delete the volume during this exercise unless you have verified that the data is disposable.

## Part 4: Review Restart Policies

Choose and document policies for Nginx, API, and MongoDB. Explain why an infrastructure service might use `unless-stopped` while an API might use `on-failure` or `on-failure:5`.

Include a short explanation of how a wrong `MONGO_URI` can create a restart storm.

## Part 5: Verify Health Checks

Run:

```bash
docker compose ps
docker inspect <api-container>
docker inspect <mongodb-container>
```

Confirm that health-check commands actually exist in the runtime images. Record whether each check represents liveness, readiness, or only a shallow process check.

## Part 6: Test Graceful Shutdown

Run:

```bash
docker stop <api-container>
docker logs <api-container>
```

Verify that the API handles `SIGTERM`, stops accepting traffic, closes resources, and exits within its configured `stop_grace_period`.

## Part 7: Verify Logs and Resources

Run:

```bash
docker compose logs --tail 100 api
docker compose logs --tail 100 nginx
docker stats --no-stream
```

Confirm that logs go to standard output/error and that secrets are not printed.

## Part 8: Failure Simulation

Perform and document each test:

### Wrong MongoDB hostname

```dotenv
MONGO_URI=mongodb://wrong-host:27017/mern
```

Diagnose with logs, Compose configuration, and network inspection. Restore the correct service name.

### Wrong Nginx upstream

Temporarily use:

```nginx
proxy_pass http://wrong-api:3000;
```

Observe the `502`, diagnose it, and restore `api:3000`.

### Broken API health check

Point the check at `/does-not-exist`, observe `unhealthy`, then restore `/health`.

## Part 9: Create a Diagnostic Script

Create `docker-diagnose.sh` or an equivalent PowerShell script that:

1. Shows Compose service state.
2. Shows recent API logs.
3. Shows recent Nginx logs.
4. Shows network information.
5. Shows resource usage.
6. Tests `http://localhost/api/health`.
7. Returns a non-zero exit code when the public health request fails.

## Part 10: Update the Project README

Add a `Production Operations` section containing:

- Service exposure.
- Internal network design.
- MongoDB volume and backup distinction.
- Health checks.
- Restart policies.
- Graceful shutdown.
- Logging strategy.
- Resource monitoring.
- Failure investigation commands.
- The limitation that basic Compose alone does not guarantee zero-downtime deployment.

## Success Criteria

- Only the intended public entry point is exposed.
- Internal service discovery uses Compose service names.
- Persistent data is stored in a named volume.
- Persistence and backup are documented separately.
- Health and restart behavior are verified rather than assumed.
- Graceful shutdown is tested.
- Logs are available through Docker without leaking secrets.
- Failure simulations can be diagnosed and repaired.
- The diagnostic script reports a failed public health path with a non-zero exit code.
