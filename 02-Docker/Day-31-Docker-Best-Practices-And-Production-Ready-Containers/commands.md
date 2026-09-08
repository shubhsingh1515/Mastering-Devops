# Day 31 - Commands: Production-Ready Docker Containers

## 1. Build a Versioned Image

```bash
docker build -t mern-api:1.4.2 .
```

## 2. Build with Plain Progress Output

```bash
docker build --progress=plain -t mern-api:1.4.2 .
```

This makes cache reuse and layer execution easier to inspect.

## 3. List Images

```bash
docker images
```

## 4. Inspect Image History

```bash
docker history mern-api:1.4.2
```

Look for unexpectedly large layers and commands that may expose secrets.

## 5. Inspect Image Metadata

```bash
docker image inspect mern-api:1.4.2
```

## 6. Run as a Non-Root User

```bash
docker run -d \
  --name mern-api-secure \
  -p 3000:3000 \
  mern-api:1.4.2

docker exec mern-api-secure whoami
docker exec mern-api-secure id
```

## 7. Test the Health Endpoint

```bash
curl http://localhost:3000/health
```

## 8. View Logs

```bash
docker logs mern-api-secure
docker logs -f mern-api-secure
docker logs --tail 100 -t mern-api-secure
```

## 9. Apply Runtime Hardening

```bash
docker run -d \
  --name mern-api-hardened \
  --read-only \
  --cap-drop=ALL \
  --tmpfs /tmp \
  --memory=512m \
  --cpus=1 \
  -p 3000:3000 \
  mern-api:1.4.2
```

Test these settings against the application before production use.

## 10. Monitor Resources

```bash
docker stats
docker stats --no-stream mern-api-secure
docker top mern-api-secure
```

## 11. Inspect Configuration and Health

```bash
docker inspect mern-api-secure
docker inspect --format='{{json .State.Health}}' mern-api-secure
```

## 12. Stop and Observe Graceful Shutdown

```bash
docker stop mern-api-secure
docker logs mern-api-secure
```

## 13. Configure a Restart Policy

```bash
docker run -d \
  --restart=unless-stopped \
  --name mern-api-restart \
  mern-api:1.4.2
```

## 14. Apply CPU and Memory Limits

```bash
docker run -d \
  --memory=512m \
  --cpus=1 \
  --name mern-api-limited \
  mern-api:1.4.2
```

## 15. Build a Frontend Multi-Stage Image

```bash
docker build -t mern-frontend:1.0 ./frontend
docker history mern-frontend:1.0
docker images mern-frontend:1.0
```

The runtime stage should contain the generated static assets and runtime server, not the complete build toolchain.

## 16. Compose Validation

```bash
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs -f api
```

## 17. Verify Service Configuration

```bash
docker compose exec api whoami
docker compose exec api printenv NODE_ENV
docker compose exec api printenv MONGO_URI
```

Do not expose command output containing credentials.

## 18. Inspect Network and Storage

```bash
docker network ls
docker network inspect <project>_app-network
docker volume ls
docker volume inspect <project>_mongo-data
```

## 19. Test Service-Name DNS

```bash
docker compose exec api getent hosts mongodb
```

## 20. Controlled Cleanup

```bash
docker stop mern-api-secure
docker rm mern-api-secure
```

Avoid manually changing production containers. Rebuild from the Dockerfile when the runtime needs to change.
