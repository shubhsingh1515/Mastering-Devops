# Day 25 - Commands: Docker Compose and Multi-Container MERN

Run these commands from the directory containing `compose.yaml`.

## 1. Validate the Compose File

```bash
docker compose config
```

Resolves and validates the Compose configuration without starting containers.

## 2. Build Service Images

```bash
docker compose build
```

Builds images for services that use `build`.

## 3. Build and Start the Stack

```bash
docker compose up -d --build
```

Builds changed images, creates the network and containers, and starts them in detached mode.

## 4. Start Without Rebuilding

```bash
docker compose up -d
```

Starts the stack using the currently available images.

## 5. Show Service Status

```bash
docker compose ps
docker compose ps -a
```

The second command also shows stopped containers.

## 6. View All Logs

```bash
docker compose logs
```

## 7. Follow One Service's Logs

```bash
docker compose logs -f api
docker compose logs -f mongodb
```

Press `Ctrl+C` to stop following logs without necessarily stopping the stack.

## 8. Execute a Command in a Running Service

```bash
docker compose exec api sh
```

Use this to inspect files, environment variables, or installed tools inside the API container.

## 9. Check Runtime Environment

```bash
docker compose exec api printenv MONGO_URI
```

The expected hostname is the service name:

```text
mongodb://mongodb:27017/mern
```

## 10. Test the API Health Endpoint

```bash
curl http://localhost:3000/health
```

The host uses the published API port. A container-to-container request uses the internal service name and port.

## 11. List Networks

```bash
docker network ls
```

## 12. Inspect the Compose Network

```bash
docker compose config
docker network inspect <project>_default
```

The exact network name can vary with the Compose project name.

## 13. List and Inspect Volumes

```bash
docker volume ls
docker volume inspect <project>_mongo-data
```

## 14. Check Container Health

```bash
docker compose ps
docker inspect <container-id-or-name>
```

Look for the health status reported by the container's health check.

## 15. Restart One Service

```bash
docker compose restart api
```

## 16. Stop the Stack

```bash
docker compose stop
```

Stops containers without removing them.

## 17. Remove the Stack

```bash
docker compose down
```

Removes Compose-created containers and network. Named volumes normally remain.

## 18. Remove the Stack and Its Volumes

```bash
docker compose down --volumes
```

Use with care because this can delete local MongoDB data.

## 19. Rebuild One Service

```bash
docker compose build api
docker compose up -d api
```

## 20. Inspect the Resolved Configuration

```bash
docker compose config --services
docker compose config --volumes
```

## Troubleshooting Flow

```bash
docker compose config
docker compose ps -a
docker compose logs mongodb
docker compose logs api
docker compose exec api printenv MONGO_URI
docker network ls
docker volume ls
```

Check that the URI uses `mongodb`, not `localhost`, and that the API and MongoDB services are on the same Compose network.
