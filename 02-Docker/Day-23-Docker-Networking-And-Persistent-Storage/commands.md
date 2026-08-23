# Day 23 — Commands: Docker Networking & Persistent Storage

## 1. Create a Docker Network

```bash
docker network create mern-network
```

## 2. List Networks

```bash
docker network ls
```

## 3. Inspect a Network

```bash
docker network inspect mern-network
```

## 4. Run MongoDB on the Network

```bash
docker run -d \
  --name mongodb \
  --network mern-network \
  mongo
```

## 5. Run the API on the Same Network

```bash
docker run -d \
  --name mern-api \
  --network mern-network \
  -p 3000:3000 \
  -e MONGO_URI=mongodb://mongodb:27017/mydb \
  mern-api:1.1
```

## 6. Check Running Containers

```bash
docker ps
```

## 7. Inspect Container Configuration

```bash
docker inspect mern-api
docker inspect mongodb
```

## 8. Access a Running Container

```bash
docker exec -it mern-api sh
docker exec -it mongodb sh
```

## 9. Test DNS Resolution Within a Container

```bash
getent hosts mongodb
```

## 10. View Container Logs

```bash
docker logs mongodb
docker logs mern-api
```

## 11. Create a Persistent Volume

```bash
docker volume create mongo-data
```

## 12. List Volumes

```bash
docker volume ls
```

## 13. Inspect a Volume

```bash
docker volume inspect mongo-data
```

## 14. Run MongoDB with Persistent Storage

```bash
docker run -d \
  --name mongodb \
  --network mern-network \
  -v mongo-data:/data/db \
  mongo
```

## 15. Remove a Container Without Losing the Volume

```bash
docker rm mongodb
```

If MongoDB was using a named volume, the data generally stays in the volume.

## 16. Connect to MongoDB Using the Service Name

```text
mongodb://mongodb:27017/mydb
```

## 17. Avoid the Common Mistake

```text
mongodb://localhost:27017/mydb
```

This is usually wrong when MongoDB is in another container.

## 18. Host vs Container Port Mapping

```bash
docker run -p 3000:3000 myapp
```

- `3000` on host
- `3000` in container

## 19. Use Host Networking (Advanced)

```bash
docker run --network host myapp
```

This is not the normal choice for a standard MERN architecture.

---

## Quick Troubleshooting Flow

```bash
docker ps
docker network inspect mern-network
docker logs mongodb
docker logs mern-api
docker exec -it mern-api sh
getent hosts mongodb
```

---

## Typical Production Pattern

```bash
docker network create mern-network
docker volume create mongo-data

docker run -d --name mongodb --network mern-network -v mongo-data:/data/db mongo
docker run -d --name mern-api --network mern-network -p 3000:3000 -e MONGO_URI=mongodb://mongodb:27017/mydb mern-api:1.1
```
