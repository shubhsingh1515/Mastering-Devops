# Day 34 - Commands: Compose Resilience and Troubleshooting

## 1. Validate Compose

```bash
docker compose config
docker compose config --services
docker compose config --quiet
```

## 2. Start the Stack

```bash
docker compose up -d --build
```

## 3. Check Service and Health State

```bash
docker compose ps
docker compose ps -a
docker inspect <container>
```

Look for `Up`, `Restarting`, `Exited`, and health states such as `starting`, `healthy`, or `unhealthy`.

## 4. Show Health Details

PowerShell:

```powershell
docker inspect --format='{{json .State.Health}}' <container>
```

## 5. Read Service Logs

```bash
docker compose logs --tail 100 api
docker compose logs --tail 100 mongodb
docker compose logs --tail 100 nginx
docker compose logs -f api
```

## 6. Inspect Restart Count and Exit State

```bash
docker inspect --format='{{.RestartCount}} {{.State.Status}} {{.State.ExitCode}}' <container>
```

## 7. Inspect Networks

```bash
docker network ls
docker network inspect <project>_mern-network
```

## 8. Test Internal DNS

```bash
docker compose exec api getent hosts mongodb
docker compose exec nginx getent hosts api
```

The diagnostic executable must exist in the selected image.

## 9. Test the API From Inside the Network

```bash
docker compose exec nginx wget -qO- http://api:3000/health
```

If the Nginx image does not contain `wget`, use another available diagnostic method.

## 10. Test the Public Path

```bash
curl -f http://localhost/api/health
```

## 11. Test MongoDB Health Manually

```bash
docker compose exec mongodb mongosh --quiet --eval "db.adminCommand({ ping: 1 })"
```

Confirm that `mongosh` is installed and compatible with the image.

## 12. Inspect Resource Usage

```bash
docker stats --no-stream
docker system df
```

## 13. Follow Restart-Loop Logs

```bash
docker compose logs -f --tail 100 api
```

In another terminal:

```bash
docker compose ps -a
```

## 14. Restart One Service After a Configuration Fix

```bash
docker compose up -d api
```

## 15. Rebuild Only the API

```bash
docker compose up -d --build api
```

## 16. Stop the Stack

```bash
docker compose down
```

## 17. Stop and Remove Volumes

Use with care because this removes persistent Compose-managed data:

```bash
docker compose down -v
```

## 18. Test a Broken Health Endpoint

Temporarily configure the health check to use:

```text
/incorrect-health
```

Then rebuild or recreate the service:

```bash
docker compose up -d --build api
docker compose ps
docker inspect --format='{{json .State.Health}}' <api-container>
```

Restore `/health` and repeat the check.

## 19. Test a Wrong MongoDB Host

Temporarily set:

```dotenv
MONGO_URI=mongodb://localhost:27017/mern
```

Then run:

```bash
docker compose up -d api
docker compose logs --tail 100 api
```

Restore:

```dotenv
MONGO_URI=mongodb://mongodb:27017/mern
```

## 20. Check the Public API With PowerShell

```powershell
Invoke-WebRequest http://localhost/api/health -UseBasicParsing
```

A failed response should be investigated alongside Compose logs and health state.
