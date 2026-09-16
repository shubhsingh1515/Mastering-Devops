# Day 37 - Assignment: Docker Production Deployment Strategies & Rollback

## Objective

Practice safe deployment and rollback for a Dockerized MERN application using versioned image artifacts, health validation, and controlled release promotion.

## Part 1: Build a Versioned Release

Build a release image for your backend or API service:

```bash
docker build -t mern-api:v1 ./backend
```

Then create a second release by making a small application change and rebuilding:

```bash
docker build -t mern-api:v2 ./backend
```

Document:

- what changed between the two builds
- how the versions differ
- how the image tags identify the release

## Part 2: Validate the Release Artifact

Run the container and confirm the app responds healthily.

```bash
docker run --rm -p 3000:3000 mern-api:v1
```

Then check:

```bash
curl http://localhost:3000/health
```

If your app uses a proxy or Compose stack, adjust the URL appropriately.

## Part 3: Deploy a New Version

Update your deployment config to use `mern-api:v2` and then restart the stack:

```bash
docker compose up -d --build
```

Check:

```bash
docker compose ps
docker compose logs --tail 100 api
```

Confirm the health endpoint behaves as expected.

## Part 4: Simulate Production Failure

Intentionally break the new release or make it fail health validation. Then observe:

- unhealthy status
- logs
- Nginx behavior
- API response failures

Document exactly why the version failed and whether the issue is related to:

- app logic
- config
- DB connectivity
- health check mismatch
- deployment config

## Part 5: Rollback to the Previous Version

Return to the known-good image:

```bash
docker build -t mern-api:v1 ./backend
```

Then update your Compose file to point to `v1` and deploy it again:

```bash
docker compose up -d
```

Confirm:

```bash
curl http://localhost/api/health
```

Document the rollback steps and explain why this rollback was easier than a rebuild-from-scratch process.

## Part 6: Explain Immutable Deployment

Write a short explanation of why the production system should use versioned or immutable image references rather than `latest`.

Include:

- traceability
- rollback readiness
- safer troubleshooting
- release accountability

## Part 7: Database Compatibility

Write a short note about database migration safety.

Discuss:

- why app rollback and database rollback are not identical
- why backward-compatible migrations are safer
- what happens if a new app expects a schema that the old app cannot handle

## Part 8: Create a Deployment Document

Create a file named `DEPLOYMENT.md` with sections for:

- Release build
- Image versioning
- Registry promotion
- Deployment process
- Health validation
- Rollback strategy
- Incident response
- Database migration strategy

## Part 9: Deploy Script Exercise

Create a script called `deploy.sh` that accepts a version argument:

```bash
./deploy.sh abc123
```

The script should:

1. validate the image version input
2. pull the specific image
3. update the deployment config
4. recreate or restart the API service
5. check the service status
6. check health
7. print success or failure

If health fails, print a message instructing rollback to the previous known-good version.

Bonus:

Store:

```bash
PREVIOUS_VERSION
CURRENT_VERSION
```

so a `rollback.sh` script can use them later.

## Success Criteria

- You built at least two tagged versions of the same API image.
- You can explain why one version is different from the other.
- You can deploy and validate a new version safely.
- You can perform an operational rollback.
- You can explain why health checks are required before accepting traffic.
- You can explain why database migration safety matters during rollback.
- You documented your deployment, rollback, and incident process.
