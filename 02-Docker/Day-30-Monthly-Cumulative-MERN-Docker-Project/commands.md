# Day 30 - Commands: Monthly MERN Docker Project

## 1. Validate the Compose File

```bash
docker compose config
docker compose config --services
docker compose config --volumes
```

## 2. Build and Start the Stack

```bash
docker compose up -d --build
```

## 3. Check Service Status

```bash
docker compose ps
docker compose ps -a
docker ps -a
```

## 4. View Logs

```bash
docker compose logs
docker compose logs --tail 100 api
docker compose logs -f api
docker logs -t <container>
```

## 5. Test the Health Endpoint

```bash
curl http://localhost:3000/health
```

## 6. Inspect Runtime Configuration

```bash
docker compose exec api printenv MONGO_URI
docker inspect <api-container>
```

Avoid sharing inspection output if it contains secrets.

## 7. Verify Service DNS

```bash
docker compose exec api getent hosts mongodb
```

The API should resolve `mongodb` through the Compose network.

## 8. Inspect Networks

```bash
docker network ls
docker network inspect <project>_app-network
```

## 9. Inspect Volumes

```bash
docker volume ls
docker volume inspect <project>_mongo-data
```

## 10. Monitor Resources

```bash
docker stats
docker stats --no-stream
docker top <api-container>
docker system df
docker system df -v
```

## 11. Test MongoDB Persistence

```bash
docker compose rm -sf mongodb
docker compose up -d mongodb
docker compose ps
```

Then verify that a known test document still exists.

## 12. Inspect Container Identity

```bash
docker compose exec api whoami
docker compose exec api id
```

The application should run as a non-root user where practical.

## 13. Build a Versioned API Image

```bash
docker build -t mern-api:abc123 ./backend
```

## 14. Tag an Image for a Registry

```bash
docker tag \
  mern-api:abc123 \
  registry.example.com/team/mern-api:abc123
```

## 15. Push and Pull an Image

```bash
docker login registry.example.com
docker push registry.example.com/team/mern-api:abc123
docker pull registry.example.com/team/mern-api:abc123
```

Use only repositories where you are authorized.

## 16. Run a Registry Image

```bash
docker run -d \
  --name mern-api-staging \
  -p 3000:3000 \
  registry.example.com/team/mern-api:abc123
```

## 17. Investigate Exit Code 137

```bash
docker ps -a
docker inspect <container>
docker logs --tail 100 <container>
docker stats --no-stream
```

On a Linux host:

```bash
free -h
df -h
```

## 18. Run a Hardened Test Container

```bash
docker run -d \
  --name mern-api-hardened \
  --read-only \
  --cap-drop=ALL \
  --tmpfs /tmp \
  --memory=512m \
  --cpus=1 \
  -p 3000:3000 \
  mern-api:secure
```

Test compatibility before using these controls in production.

## 19. Check and Run the Health Script

Linux:

```bash
chmod +x health-check.sh
./health-check.sh
```

## 20. Controlled Cleanup

```bash
docker compose stop
docker compose down
docker system df
```

Use this destructive command only after confirming the data impact:

```bash
docker compose down --volumes
```

It can delete local MongoDB data.
