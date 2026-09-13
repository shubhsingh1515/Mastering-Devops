# Day 33 - Commands: Docker Production Configuration and Environment Management

## 1. Create the Lesson Directory

PowerShell:

```powershell
New-Item -ItemType Directory -Path .\Day-33-Docker-Production-Configuration-And-Environment-Management
```

Bash:

```bash
mkdir Day-33-Docker-Production-Configuration-And-Environment-Management
```

## 2. Create Environment Files

```powershell
Copy-Item .env.example .env
```

Edit `.env` with local values. Keep `.env` ignored by Git.

## 3. Validate Git Ignore Rules

```bash
git status --short
git check-ignore -v .env
```

The second command should show the ignore rule that matches `.env`.

## 4. Render Resolved Compose Configuration

```bash
docker compose config
docker compose config --services
docker compose config --environment
```

Review output carefully because resolved configuration may contain secrets.

## 5. Validate Without Starting

```bash
docker compose config --quiet
```

No output normally means the Compose model is valid.

## 6. Build and Start the Stack

```bash
docker compose up -d --build
```

## 7. Check Service State

```bash
docker compose ps
docker compose ps -a
```

`Up` means the container process is running; it does not automatically mean the application is healthy.

## 8. Read API Logs

```bash
docker compose logs api
docker compose logs --tail 100 api
docker compose logs -f api
```

## 9. Read MongoDB Logs

```bash
docker compose logs --tail 100 mongodb
```

## 10. Inspect a Safe Variable

```bash
docker compose exec api printenv NODE_ENV
docker compose exec api printenv PORT
```

Avoid dumping the complete environment in production.

## 11. Inspect the Mongo URI Carefully

For a local lab with no password in the URI:

```bash
docker compose exec api printenv MONGO_URI
```

Redact credentials before sharing any output.

## 12. Inspect Network Membership

```bash
docker network ls
docker network inspect <project>_mern-network
```

Confirm that `nginx`, `api`, and `mongodb` are attached as intended.

## 13. Test Internal DNS

```bash
docker compose exec api getent hosts mongodb
docker compose exec nginx getent hosts api
```

If the image lacks `getent`, use an image-specific network diagnostic tool or inspect the network from Docker.

## 14. Test the Public Path

```bash
curl http://localhost/api/health
```

Expected example:

```json
{"status":"ok"}
```

## 15. Stop and Remove Containers Without Removing Data

```bash
docker compose down
```

## 16. Stop and Remove Containers and Volumes

Use with care because this deletes Compose-managed volume data:

```bash
docker compose down -v
```

## 17. Deliberately Test a Missing Variable

Use a required Compose expression:

```yaml
environment:
  MONGO_URI: ${MONGO_URI:?MONGO_URI must be provided}
```

Temporarily remove `MONGO_URI` from `.env`, then run:

```bash
docker compose config
```

Compose should fail with the required-variable message.

## 18. Deliberately Test the Wrong Mongo Host

Temporarily set:

```dotenv
MONGO_URI=mongodb://localhost:27017/mern
```

Restart the API:

```bash
docker compose up -d api
```

Read the logs:

```bash
docker compose logs --tail 100 api
```

Restore:

```dotenv
MONGO_URI=mongodb://mongodb:27017/mern
```

Then restart and verify the API recovers.

## 19. Inspect a Container Without Printing Everything

```bash
docker inspect <container>
docker inspect --format='{{json .Config.Env}}' <container>
```

Treat the second command as sensitive output.

## 20. Check Resource Usage

```bash
docker stats --no-stream
docker system df
```

## 21. Stop the Stack After the Lab

```bash
docker compose down
```
