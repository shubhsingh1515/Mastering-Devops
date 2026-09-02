# Day 27 - Commands: Docker Registry and Deployment Workflow

## 1. Build a Versioned Image

```bash
docker build -t mern-api:1.0 .
```

## 2. Add a Commit-Based Tag

```bash
docker tag mern-api:1.0 mern-api:8f31c2a
```

A commit-based tag helps connect the image to a source revision.

## 3. Add a Registry-Qualified Tag

```bash
docker tag \
  mern-api:1.0 \
  registry.example.com/team/mern-api:1.0
```

Tagging adds another reference. It does not rebuild the image.

## 4. List Local Images

```bash
docker images
```

## 5. Inspect Image Metadata

```bash
docker image inspect mern-api:1.0
```

## 6. Log In to a Registry

```bash
docker login registry.example.com
```

Use a secure credential mechanism. Never commit registry passwords or tokens to source control.

## 7. Push an Image

```bash
docker push registry.example.com/team/mern-api:1.0
```

## 8. Pull an Image

```bash
docker pull registry.example.com/team/mern-api:1.0
```

## 9. Run a Pulled Image

```bash
docker run -d \
  --name mern-api \
  -p 3000:3000 \
  registry.example.com/team/mern-api:1.0
```

## 10. Test the Deployed Container

```bash
curl http://localhost:3000/health
```

## 11. Show Image History

```bash
docker history mern-api:1.0
```

This helps inspect the layers that make up the image.

## 12. Remove a Local Tag

```bash
docker rmi mern-api:1.0
```

Removing one tag does not necessarily remove the underlying image if another tag still references it.

## 13. Stop and Remove a Test Container

```bash
docker stop mern-api
docker rm mern-api
```

## 14. Pull an Exact Digest

```bash
docker pull registry.example.com/team/mern-api@sha256:<digest>
```

A digest identifies exact image content more strongly than a mutable tag.

## 15. Run an Exact Digest

```bash
docker run -d \
  --name mern-api \
  -p 3000:3000 \
  registry.example.com/team/mern-api@sha256:<digest>
```

## 16. Deploy a Previous Version for Rollback

```bash
docker pull registry.example.com/team/mern-api:1.6.4
docker stop mern-api
docker rm mern-api
docker run -d \
  --name mern-api \
  -p 3000:3000 \
  registry.example.com/team/mern-api:1.6.4
```

In production, use the rollback mechanism provided by your orchestrator where available.

## 17. Inspect Registry-Qualified References

```bash
docker image inspect registry.example.com/team/mern-api:1.0
```

## 18. Build and Tag Multiple References

```bash
docker build \
  -t registry.example.com/team/mern-api:1.4.2 \
  -t registry.example.com/team/mern-api:8f31c2a \
  .
```

## 19. Push Multiple References

```bash
docker push registry.example.com/team/mern-api:1.4.2
docker push registry.example.com/team/mern-api:8f31c2a
```

## 20. Local Deployment Workflow

```bash
docker build -t mern-api:1.0 .
docker tag mern-api:1.0 registry.example.com/team/mern-api:1.0
docker images
docker login registry.example.com
docker push registry.example.com/team/mern-api:1.0
docker pull registry.example.com/team/mern-api:1.0
docker run -d --name mern-api -p 3000:3000 registry.example.com/team/mern-api:1.0
curl http://localhost:3000/health
```

## Troubleshooting Flow

```bash
docker images
docker image inspect <image>
docker pull <registry>/<repository>:<tag>
docker ps -a
docker logs <container>
docker inspect <container>
```

Check that:

- the registry hostname is correct
- the repository path is correct
- you are authenticated
- your account can pull or push the repository
- the image tag exists
- the deployment uses the intended tag or digest
- the health endpoint responds after startup
