# Day 38 - Assignment: Docker Registry & Image Lifecycle

## Objective

Practice the complete Docker artifact lifecycle locally: build, inspect, tag, publish when a registry is available, pull, run, promote, and roll back a known image.

Use a real registry only if you have one. `registry.example.com` is a placeholder and should not be pushed to.

## Part 1: Build Two Traceable Releases

Build two versions of your MERN API or another Dockerized service:

```bash
docker build -t mern-api:v1.0.0 ./backend
docker build -t mern-api:v1.1.0 ./backend
```

Also create commit-style tags:

```bash
docker tag mern-api:v1.0.0 mern-api:commit-a83f29c
docker tag mern-api:v1.1.0 mern-api:commit-b91e42d
```

Document what changed between the releases and how the tags identify the source.

## Part 2: Inspect the Artifacts

Run:

```bash
docker image ls mern-api
docker image inspect mern-api:v1.0.0
docker history mern-api:v1.0.0
```

Record:

- image size
- creation time
- image ID
- exposed ports
- configured entrypoint or command
- important environment metadata

## Part 3: Validate the Image Before Publishing

Run the first release:

```bash
docker run --rm -p 3000:3000 mern-api:v1.0.0
```

From another terminal, check the health endpoint:

```bash
curl http://localhost:3000/health
```

PowerShell alternative:

```powershell
Invoke-WebRequest http://localhost:3000/health
```

Document whether the container was merely running or whether the application was actually healthy.

## Part 4: Publish to a Registry

If you have access to a registry, authenticate and tag the image:

```bash
docker login <your-registry>
docker tag mern-api:v1.1.0 <your-registry>/mern-api:v1.1.0
docker tag mern-api:v1.1.0 <your-registry>/mern-api:commit-b91e42d
```

Push both references:

```bash
docker push <your-registry>/mern-api:v1.1.0
docker push <your-registry>/mern-api:commit-b91e42d
```

Never commit registry credentials to this repository.

## Part 5: Pull and Run the Published Artifact

From another Docker environment, or after removing the local copy, run:

```bash
docker pull <your-registry>/mern-api:v1.1.0
docker run --rm -p 3000:3000 <your-registry>/mern-api:v1.1.0
```

Confirm that the pulled image behaves like the tested image.

## Part 6: Use a Production Compose Reference

Create or update a production-style Compose file so the API uses `image:` rather than `build:`:

```yaml
services:
  api:
    image: <your-registry>/mern-api:v1.1.0
```

Run:

```bash
docker compose pull api
docker compose up -d api
docker compose ps api
docker compose logs --tail 100 api
```

Explain why production should normally pull an already-built image instead of rebuilding source on the server.

## Part 7: Simulate Promotion

Document this promotion flow:

```text
Build commit-b91e42d
       |
       v
Test and scan
       |
       v
Registry
       |
       v
Staging
       |
       v
Approval
       |
       v
Production uses the same image
```

Record the image tag and digest used in each environment. Explain why rebuilding for production would weaken reproducibility.

## Part 8: Simulate a Rollback

Deploy `v1.1.0`, then simulate a failed health check or failed application response. Return to `v1.0.0`:

```yaml
services:
  api:
    image: <your-registry>/mern-api:v1.0.0
```

Apply the rollback:

```bash
docker compose pull api
docker compose up -d --no-deps api
```

Document:

- current version
- previous known-good version
- failure symptom
- evidence from logs or health checks
- rollback command
- validation result

## Part 9: Explain Retention

Write a short image-retention policy. Include:

- how many previous releases remain available
- which images cannot be deleted
- how long production artifacts are retained
- who may delete images
- how rollback references are recorded

## Part 10: Troubleshoot a Pull Failure

Create a troubleshooting checklist for:

```text
pull access denied
manifest unknown
```

Your checklist must cover:

- registry hostname
- repository path
- tag or digest
- authentication
- pull permission
- DNS and network access
- registry availability
- host architecture compatibility

## Part 11: Database Compatibility Note

Explain why rolling back an application image does not automatically roll back a database schema. Describe why backward-compatible migrations and expand-and-contract releases reduce risk.

## Part 12: Create a Release Record

Create `RELEASE.md` containing:

- release version
- Git commit
- image repository
- image tag
- image digest
- build date
- scan result
- staging result
- production deployment time
- previous rollback target

## Success Criteria

- You built at least two traceable image versions.
- You inspected image metadata and history.
- You validated an image through a health endpoint.
- You can tag and push an image without exposing credentials.
- You can explain build-once-deploy-many.
- You can deploy the same artifact to staging and production.
- You can roll back to a retained known-good image.
- You can troubleshoot common image pull failures.
- You can explain why image retention and database compatibility matter.
