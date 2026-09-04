# Day 28 - Commands: Docker Logging, Monitoring & Resource Management

## 1. List Running Containers

```bash
docker ps
```

## 2. List All Containers

```bash
docker ps -a
```

Use this when a container may have stopped or entered a restart loop.

## 3. View Container Logs

```bash
docker logs mern-api
```

## 4. Follow Logs in Real Time

```bash
docker logs -f mern-api
```

## 5. Limit Log Output

```bash
docker logs --tail 100 mern-api
```

## 6. Add Log Timestamps

```bash
docker logs -t mern-api
```

## 7. Combine Log Options

```bash
docker logs -f --tail 50 -t mern-api
```

## 8. View Compose Service Logs

```bash
docker compose logs api
docker compose logs -f api
docker compose logs --tail 100 mongodb
```

## 9. Show Live Resource Usage

```bash
docker stats
```

## 10. Monitor One Container

```bash
docker stats mern-api
```

## 11. Get a Single Resource Snapshot

```bash
docker stats --no-stream
```

## 12. Inspect Processes in a Container

```bash
docker top mern-api
```

## 13. Inspect Container Configuration

```bash
docker inspect mern-api
```

Review restart policy, health status, limits, mounts, networks, and environment configuration. Avoid exposing inspection output if it contains secrets.

## 14. Enter a Running Container

```bash
docker exec -it mern-api sh
```

This only works when the image contains a shell.

## 15. Test the API Health Endpoint

```bash
curl http://localhost:3000/health
```

## 16. Check Docker Disk Usage

```bash
docker system df
docker system df -v
```

## 17. Inspect Docker Resources

```bash
docker images
docker ps -a
docker volume ls
docker builder du
```

## 18. Configure a Restart Policy at Runtime

```bash
docker run \
  --restart=unless-stopped \
  myapp
```

## 19. Inspect Restart Policy

```bash
docker inspect --format='{{json .HostConfig.RestartPolicy}}' mern-api
```

## 20. Run a Resource-Limited Container

```bash
docker run -d \
  --name mern-api-limited \
  --memory=512m \
  --cpus=1 \
  -p 3000:3000 \
  myapp
```

## 21. Stop and Remove a Test Container

```bash
docker stop mern-api-limited
docker rm mern-api-limited
```

## 22. Compose Status and Logs

```bash
docker compose ps
docker compose logs --tail 50 api
docker compose config
```

## 23. Dangerous Cleanup Commands

```bash
docker system prune
docker system prune --volumes
```

Review what will be removed and protect required images, volumes, and rollback artifacts before cleanup.

## 24. Production Incident Flow

```bash
docker ps -a
docker logs --tail 100 mern-api
docker inspect mern-api
docker stats --no-stream
docker system df
curl http://localhost:3000/health
```

On a Linux host, also check:

```bash
free -h
top
df -h
```

## 25. Compose Health Check Example

```yaml
services:
  api:
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s
```

Verify that `wget` exists in the image before using this command.
