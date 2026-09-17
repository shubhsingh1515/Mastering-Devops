# Day 40 - Commands: Docker Production Readiness & Final Assessment

Replace placeholder names, paths, image tags, and network names with values from your project.

## 1. Validate Compose Configuration

```bash
docker compose -f docker-compose.prod.yml config
docker compose -f docker-compose.prod.yml config --services
docker compose -f docker-compose.prod.yml config --images
```

Use the resolved configuration to confirm image references, service names, environment keys, networks, volumes, health checks, and published ports.

## 2. Pull the Approved Artifact

```bash
docker compose -f docker-compose.prod.yml pull
docker pull registry.example.com/mern-api:v2.3.0
```

For exact content:

```bash
docker pull registry.example.com/mern-api@sha256:<digest>
```

## 3. Start a Production-Style Stack

```bash
docker compose -f docker-compose.prod.yml up -d
```

Start only selected services when testing:

```bash
docker compose -f docker-compose.prod.yml up -d mongodb api nginx
```

## 4. Check Service State and Health

```bash
docker compose -f docker-compose.prod.yml ps
docker ps
docker inspect --format='{{json .State.Health}}' <container-name>
```

Check the public endpoint:

Linux/macOS:

```bash
curl -i http://localhost/health
curl -i http://localhost/ready
```

PowerShell:

```powershell
Invoke-WebRequest http://localhost/health
Invoke-WebRequest http://localhost/ready
```

## 5. Read Logs

```bash
docker compose -f docker-compose.prod.yml logs --tail 100
docker compose -f docker-compose.prod.yml logs --tail 100 api
docker compose -f docker-compose.prod.yml logs --tail 100 nginx
docker compose -f docker-compose.prod.yml logs --tail 100 mongodb
docker compose -f docker-compose.prod.yml logs -f api
```

Do not print secrets while collecting configuration or logs.

## 6. Inspect Containers

```bash
docker inspect <container-name>
docker top <container-name>
docker stats --no-stream
docker port <container-name>
```

Inspect the configured runtime user and command:

```bash
docker inspect --format='User={{.Config.User}} Cmd={{json .Config.Cmd}}' <container-name>
```

## 7. Inspect Networks and DNS

```bash
docker network ls
docker network inspect <network-name>
```

Run a temporary diagnostic container on the application network:

```bash
docker run --rm --network <network-name> busybox nslookup mongodb
docker run --rm --network <network-name> busybox wget -qO- http://api:3000/health
```

Use a diagnostic image that is approved for your environment. Do not add debugging tools to the production runtime image only for incident convenience.

## 8. Check the API-to-MongoDB Configuration

Inside a container, `localhost` means that same container. The internal URI should normally use the Compose service name:

```text
mongodb://mongodb:27017/mern
```

Not:

```text
mongodb://localhost:27017/mern
```

Review the resolved configuration without exposing secret values:

```bash
docker compose -f docker-compose.prod.yml config
```

## 9. Check Volumes and Persistence

```bash
docker volume ls
docker volume inspect <volume-name>
docker compose -f docker-compose.prod.yml down
docker compose -f docker-compose.prod.yml up -d mongodb
```

`down` without `-v` normally preserves named volumes. Do not use `docker compose down -v` against production data unless data destruction is explicitly intended and approved.

## 10. Check Restart and Exit Behavior

```bash
docker compose -f docker-compose.prod.yml ps
docker inspect --format='Status={{.State.Status}} ExitCode={{.State.ExitCode}} Restarts={{.RestartCount}}' <container-name>
```

A restart count that continually increases is evidence to investigate, not a success signal.

## 11. Record Release Traceability

```bash
docker image inspect registry.example.com/mern-api:v2.3.0
docker image inspect --format='{{json .RepoDigests}}' registry.example.com/mern-api:v2.3.0
docker compose -f docker-compose.prod.yml images
```

Record the image tag, digest, Git revision, deployment time, and previous release.

## 12. Deploy a Specific Version

```bash
docker compose -f docker-compose.prod.yml pull api
docker compose -f docker-compose.prod.yml up -d --no-deps api
```

For a rollback, change the approved image reference to the known-good version or digest, then run the same controlled deployment procedure.

## 13. Roll Back and Validate

```bash
docker compose -f docker-compose.prod.yml pull api
docker compose -f docker-compose.prod.yml up -d --no-deps api
docker compose -f docker-compose.prod.yml ps api
curl -i http://localhost/health
curl -i http://localhost/ready
```

Before rollback, review database compatibility and configuration compatibility.

## 14. Stop and Remove the Stack Carefully

```bash
docker compose -f docker-compose.prod.yml stop
docker compose -f docker-compose.prod.yml down
```

Avoid deleting volumes during ordinary cleanup:

```bash
# Destructive for named volumes; use only with explicit approval.
docker compose -f docker-compose.prod.yml down -v
```

## 15. Build and Scan an Assessment Image

```bash
docker build --pull -t mern-api:assessment-v1 ./backend
docker image inspect mern-api:assessment-v1
trivy image --severity HIGH,CRITICAL mern-api:assessment-v1
```

## 16. Production Troubleshooting Order

```text
1. Confirm impact and affected route.
2. Check public proxy logs.
3. Check API logs.
4. Check API health and readiness.
5. Check service state and restart count.
6. Check network membership and DNS.
7. Check database logs and connectivity.
8. Check CPU, memory, disk, and volume state.
9. Compare current and previous image/configuration.
10. Remediate or roll back, then validate.
```
