# Day 38 - Commands: Docker Registry & Image Lifecycle

## 1. Build a Versioned Image

```bash
docker build -t mern-api:1.0.0 ./backend
```

Build using a Git commit identifier:

```bash
docker build -t mern-api:a83f29c ./backend
```

## 2. List Local Images

```bash
docker image ls
```

Filter for one repository:

```bash
docker image ls mern-api
```

## 3. Inspect an Image

```bash
docker image inspect mern-api:1.0.0
```

Inspect selected metadata:

```bash
docker image inspect --format='{{.Id}} {{.Created}}' mern-api:1.0.0
```

## 4. View Image History

```bash
docker history mern-api:1.0.0
```

## 5. Run the Image Locally

```bash
docker run --rm -p 3000:3000 mern-api:1.0.0
```

## 6. Check the Health Endpoint

Linux/macOS:

```bash
curl http://localhost:3000/health
```

PowerShell:

```powershell
Invoke-WebRequest http://localhost:3000/health
```

## 7. Tag for a Registry

```bash
docker tag mern-api:1.0.0 registry.example.com/mern-api:1.0.0
docker tag mern-api:1.0.0 registry.example.com/mern-api:a83f29c
```

## 8. Authenticate to a Registry

Use the registry's documented authentication method:

```bash
docker login registry.example.com
```

Do not place the password in a Dockerfile, Compose file, or Git repository.

## 9. Push an Image

```bash
docker push registry.example.com/mern-api:1.0.0
docker push registry.example.com/mern-api:a83f29c
```

## 10. Pull an Exact Tag

```bash
docker pull registry.example.com/mern-api:1.0.0
```

## 11. Pull by Digest

```bash
docker pull registry.example.com/mern-api@sha256:0123456789abcdef...
```

Use a real digest copied from your registry. The example digest is illustrative only.

## 12. Inspect the Registry-Tagged Image

```bash
docker image inspect registry.example.com/mern-api:1.0.0
```

## 13. Run the Pulled Image

```bash
docker run --rm -p 3000:3000 registry.example.com/mern-api:1.0.0
```

## 14. Use the Image in Production Compose

```yaml
services:
  api:
    image: registry.example.com/mern-api:1.0.0
```

Then pull and start the service:

```bash
docker compose pull api
docker compose up -d api
```

## 15. Check Compose State and Logs

```bash
docker compose ps
docker compose logs --tail 100 api
docker compose logs -f api
```

## 16. Record the Image Digest Used Locally

```bash
docker image inspect --format='{{json .RepoDigests}}' registry.example.com/mern-api:1.0.0
```

## 17. Check Container Health and Metadata

```bash
docker inspect <container-name>
docker inspect --format='{{json .State.Health}}' <container-name>
```

## 18. Verify the Resolved Compose Configuration

```bash
docker compose config
```

Check that the resolved `image` value is the intended version or digest.

## 19. Simulate a Versioned Deployment

```bash
docker compose pull api
docker compose up -d --no-deps api
docker compose ps api
```

## 20. Roll Back to a Known-Good Version

Update the Compose image reference to the previous version, then run:

```bash
docker compose pull api
docker compose up -d --no-deps api
```

Validate the rollback:

```bash
curl http://localhost:3000/health
```

## 21. Remove a Local Image Carefully

Only remove an image when it is no longer needed locally:

```bash
docker image rm mern-api:1.0.0
```

Do not delete the only available rollback artifact during an incident.

## 22. Useful Pull-Failure Checks

```bash
docker info
docker context show
docker image ls
docker compose config
docker compose logs --tail 100 api
```

Then check the registry hostname, repository path, tag, credentials, permissions, and network access.

## 23. Practical Release Naming

A release can have both a human-readable version and a source identifier:

```bash
docker tag mern-api:build registry.example.com/mern-api:v1.5.0
docker tag mern-api:build registry.example.com/mern-api:a83f29c
docker push registry.example.com/mern-api:v1.5.0
docker push registry.example.com/mern-api:a83f29c
```
