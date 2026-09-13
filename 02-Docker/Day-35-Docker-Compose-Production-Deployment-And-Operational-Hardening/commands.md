# Day 35 - Commands: Production Compose Operations

## 1. Validate the Resolved Compose File

```bash
docker compose config
docker compose config --services
docker compose config --quiet
```

## 2. Start or Rebuild the Stack

```bash
docker compose up -d --build
```

## 3. Check Service and Health State

```bash
docker compose ps
docker compose ps -a
docker inspect <container>
```

## 4. Inspect Health Details

PowerShell:

```powershell
docker inspect --format='{{json .State.Health}}' <container>
```

## 5. View Logs

```bash
docker compose logs --tail 100 api
docker compose logs --tail 100 nginx
docker compose logs --tail 100 mongodb
docker compose logs -f api
```

## 6. Inspect Restart and Exit State

```bash
docker inspect --format='{{.RestartCount}} {{.State.Status}} {{.State.ExitCode}}' <container>
```

## 7. Inspect Ports

```bash
docker compose port nginx 80
docker port <container>
```

Confirm that only intended services publish host ports.

## 8. Inspect Networks

```bash
docker network ls
docker network inspect <project>_mern-network
```

## 9. Test Internal Service Discovery

```bash
docker compose exec nginx getent hosts api
docker compose exec api getent hosts mongodb
```

The `getent` command must exist in the image.

## 10. Test the API Through Nginx

```bash
curl -f http://localhost/api/health
```

PowerShell:

```powershell
Invoke-WebRequest http://localhost/api/health -UseBasicParsing
```

## 11. Test the API From Inside the Network

```bash
docker compose exec nginx wget -qO- http://api:3000/health
```

Use an available image-specific tool if `wget` is absent.

## 12. Test MongoDB Connectivity

```bash
docker compose exec mongodb mongosh --quiet --eval "db.adminCommand({ ping: 1 })"
```

Confirm that `mongosh` is installed in the selected MongoDB image.

## 13. Check Resource Usage

```bash
docker stats --no-stream
docker system df
```

## 14. Test Graceful Shutdown

```bash
docker stop <api-container>
docker logs <api-container>
```

Look for the application's shutdown handler and resource cleanup messages.

## 15. Recreate One Service After a Targeted Fix

```bash
docker compose up -d api
```

## 16. Rebuild One Service

```bash
docker compose up -d --build api
```

## 17. Stop the Stack

```bash
docker compose down
```

## 18. Stop and Remove Volumes

Use with care because this removes persistent Compose-managed data:

```bash
docker compose down -v
```

## 19. Follow a Failure

```bash
docker compose logs -f --tail 100 api
```

In another terminal:

```bash
docker compose ps -a
docker inspect <api-container>
```

## 20. Check the Resolved Environment Safely

```bash
docker compose config
```

Redact secrets before sharing output. Do not casually run a full environment dump in production.
