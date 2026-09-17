# Day 40 - Assignment: Docker Production Readiness & Final Container Assessment

## Objective

Review or design a production-style Dockerized MERN deployment. Demonstrate that the system is secure, reproducible, observable, persistent, health-checked, correctly networked, and recoverable.

## Scenario

Your project contains:

```text
frontend/
backend/
nginx/
mongodb
```

The deployment must satisfy:

```text
1. Nginx is publicly accessible.
2. The API is internal.
3. MongoDB is internal.
4. MongoDB data persists.
5. The API exposes health information.
6. Containers do not run unnecessarily as root.
7. Secrets are not baked into images.
8. Images are versioned and traceable.
9. Images are scanned.
10. A known-good previous version can be rolled back.
```

## Part 1: Architecture Review

Draw or describe:

```text
Internet -> Nginx -> Frontend/API -> MongoDB -> Persistent volume
```

Document:

- public ports
- internal ports
- Docker networks
- service names used for internal DNS
- volume ownership and purpose
- where TLS terminates

Explain why MongoDB and the API do not need to be public host ports.

## Part 2: Dockerfile Review

Review the frontend and backend Dockerfiles. Record:

```text
Base image and version:
Lockfile-aware install:
Multi-stage build:
Runtime user:
Production dependencies only:
Health check:
Secret exposure risk:
Unnecessary packages:
```

Correct at least three production issues, such as a mutable base image, root execution, secret baking, missing `.dockerignore`, or development dependencies in the runtime image.

## Part 3: Compose Review

Create or update:

```text
docker-compose.prod.yml
```

Use production image references instead of building source on the production host where appropriate. Verify:

- only Nginx publishes public host ports
- API and MongoDB use private networks
- MongoDB has a named persistent volume
- restart behavior is intentional
- health checks are meaningful
- environment configuration is externalized
- dependencies use service names

Validate it:

```bash
docker compose -f docker-compose.prod.yml config
docker compose -f docker-compose.prod.yml config --images
```

## Part 4: Build, Scan, and Record

Build a versioned image:

```bash
docker build --pull -t mern-api:assessment-v1 ./backend
```

Inspect it:

```bash
docker image inspect mern-api:assessment-v1
docker history --no-trunc mern-api:assessment-v1
docker run --rm mern-api:assessment-v1 id
```

Scan it:

```bash
trivy image --severity HIGH,CRITICAL mern-api:assessment-v1
```

Record the image tag, digest if available, scanner result, open findings, remediation, and release decision.

## Part 5: Health and Failure Test

Start the stack:

```bash
docker compose -f docker-compose.prod.yml up -d
```

Check:

```bash
docker compose -f docker-compose.prod.yml ps
curl -i http://localhost/health
curl -i http://localhost/ready
docker compose -f docker-compose.prod.yml logs --tail 100 api
```

Then test a controlled dependency failure in a non-production environment. Observe whether:

- the API remains alive where intended
- readiness changes appropriately
- database-dependent requests return controlled errors
- logs identify the dependency failure
- the restart policy creates a loop

Restore the dependency and validate recovery.

## Part 6: Incident Drill

Simulate or analyze this configuration:

```text
MONGO_URI=mongodb://localhost:27017/mern
```

Explain why it fails inside the API container and correct it to use the MongoDB Compose service name:

```text
MONGO_URI=mongodb://mongodb:27017/mern
```

Use evidence:

```bash
docker compose -f docker-compose.prod.yml logs api
docker network inspect <network-name>
docker compose -f docker-compose.prod.yml config
```

## Part 7: Rollback Plan

Deploy a current and previous image reference:

```text
Current:  mern-api:v2.3.0
Previous: mern-api:v2.2.0
```

Write the exact rollback commands and validation steps. Before declaring the rollback safe, document:

```text
Database schema compatibility:
Configuration compatibility:
Background job compatibility:
Persistent data considerations:
Health checks after rollback:
```

## Part 8: Required Deliverables

Create these files in your project:

```text
docker-compose.prod.yml
Dockerfile
.dockerignore
DEPLOYMENT.md
RUNBOOK.md
SECURITY.md
```

The documentation must explain:

```text
Architecture
Networking
Configuration and secrets
Persistence and backup distinction
Health and readiness
Security controls
Deployment flow
Rollback procedure
Troubleshooting procedure
```

## Part 9: Final Readiness Report

Write a short report using this format:

```text
Release:
Image digest:
Git revision:
Staging result:
Health result:
Security scan result:
Open risks:
Rollback target:
Database compatibility:
Production decision: approve / remediate / block
```

## Reflection Questions

1. Why is `docker compose up -d` not enough evidence of production readiness?
2. Why should only Nginx normally publish host ports?
3. What is the difference between liveness and readiness?
4. Why is persistence not the same as backup?
5. What evidence would you preserve during a 500-error incident?
6. How can a database migration make application rollback unsafe?
