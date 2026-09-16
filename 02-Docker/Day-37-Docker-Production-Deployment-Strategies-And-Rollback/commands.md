# Day 37 - Commands: Docker Production Deployment Strategies & Rollback

## 1. Build a Versioned Image

```bash
docker build -t mern-api:v1 ./backend
docker build -t mern-api:v2 ./backend
```

## 2. List Local Images

```bash
docker images
```

## 3. Run a Versioned Image for Validation

```bash
docker run --rm -p 3000:3000 mern-api:v1
```

## 4. Test the Health Endpoint

```bash
curl http://localhost:3000/health
```

PowerShell:

```powershell
Invoke-WebRequest http://localhost:3000/health
```

## 5. Tag an Image for a Registry

```bash
docker tag mern-api:v2 registry.example.com/mern-api:v2
```

## 6. Push the Image to a Registry

```bash
docker push registry.example.com/mern-api:v2
```

## 7. Pull the Exact Image Version

```bash
docker pull registry.example.com/mern-api:v2
```

## 8. Start or Recreate the Stack with the New Version

```bash
docker compose up -d --build
```

or if updating configuration manually:

```bash
docker compose up -d
```

## 9. Check Service State

```bash
docker compose ps
```

## 10. View Logs

```bash
docker compose logs --tail 100 api
docker compose logs --tail 100 nginx
docker compose logs -f api
```

## 11. Inspect Container Health and Metadata

```bash
docker inspect <api-container>
docker inspect --format='{{json .State.Health}}' <api-container>
```

## 12. Check Resource Usage

```bash
docker stats --no-stream
```

## 13. Roll Back to the Previous Version

After updating the service image ref:

```bash
docker compose up -d
```

Or if you have a specific old image tag:

```bash
docker compose up -d --no-deps api
```

## 14. Validate the Rollback

```bash
curl http://localhost/api/health
```

## 15. Inspect Runtime Configuration

```bash
docker compose config
```

This helps verify the resolved image tag and environment settings.

## 16. Remove Old Images Carefully

Use only when you are sure they are no longer required for rollback:

```bash
docker rmi mern-api:v1
```

Do not remove previous versions too early if rollback still matters.

## 17. Create a Simple Deployment Script Pattern

```bash
#!/bin/bash
VERSION="$1"

if [ -z "$VERSION" ]; then
  echo "Usage: ./deploy.sh <version>"
  exit 1
fi

IMAGE="mern-api:${VERSION}"

echo "Deploying ${IMAGE}"
echo "Pulling image..."
docker pull "$IMAGE"

echo "Starting service..."
docker compose up -d

if curl -fsS http://localhost/api/health >/dev/null; then
  echo "Deployment successful."
else
  echo "Deployment failed. Recommended action: rollback to previous known-good version."
  exit 1
fi
```

## 18. Useful Deployment Safety Checks

```bash
docker compose ps
docker compose logs --tail 50 api
docker inspect --format='{{.State.Status}}' <container>
```

## 19. Practical Advice

- keep previous image versions around
- deploy exact known artifacts
- validate health before calling the release successful
- preserve logs and metadata during incident response
- treat database compatibility as a separate deployment concern
