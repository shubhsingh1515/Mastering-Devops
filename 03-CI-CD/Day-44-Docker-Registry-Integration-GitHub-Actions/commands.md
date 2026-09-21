# Day 44 - Commands: Docker Registry Integration

Replace registry hostnames, repository names, paths, ports, and image tags with values from your project.

---

## 1. Inspect Repository and Commit

```bash
git status
git branch --show-current
git log -1 --oneline
git rev-parse HEAD
git rev-parse --short HEAD
```

PowerShell:

```powershell
git status
git branch --show-current
git log -1 --oneline
git rev-parse HEAD
git rev-parse --short HEAD
```

## 2. Run Local Quality Gates

```bash
npm ci
npm run lint
npm test
npm run build
```

Bash chain:

```bash
npm ci && npm run lint && npm test && npm run build
```

PowerShell:

```powershell
npm ci
if ($LASTEXITCODE -eq 0) { npm run lint }
if ($LASTEXITCODE -eq 0) { npm test }
if ($LASTEXITCODE -eq 0) { npm run build }
```

## 3. Define the Image Reference

Bash:

```bash
REGISTRY=registry.example.com
IMAGE_NAME=mern-api
GIT_SHA=$(git rev-parse --short HEAD)
IMAGE="$REGISTRY/$IMAGE_NAME:$GIT_SHA"
echo "$IMAGE"
```

PowerShell:

```powershell
$Registry = "registry.example.com"
$ImageName = "mern-api"
$GitSha = git rev-parse --short HEAD
$Image = "$Registry/$ImageName`:$GitSha"
$Image
```

## 4. Build and Inspect the Image

```bash
docker build -t "$IMAGE" ./backend
docker image inspect "$IMAGE"
docker history --no-trunc "$IMAGE"
docker images "$REGISTRY/$IMAGE_NAME"
```

PowerShell build:

```powershell
docker build -t $Image .\backend
```

## 5. Scan the Image

Example with Trivy:

```bash
trivy image "$IMAGE"
trivy image --exit-code 1 --severity HIGH,CRITICAL "$IMAGE"
```

Use the scanner and policy approved by your project. The sequence is:

```text
Build -> Scan -> Pass -> Login -> Push
```

## 6. Log In and Push

Interactive login:

```bash
docker login registry.example.com
```

Non-interactive login:

```bash
echo "$REGISTRY_PASSWORD" | docker login "$REGISTRY" \\
  --username "$REGISTRY_USERNAME" \\
  --password-stdin
```

Push:

```bash
docker push "$IMAGE"
docker image inspect --format='{{json .RepoDigests}}' "$IMAGE"
```

Never place real passwords or tokens in shell history, source files, workflow YAML, or command output.

## 7. Inspect the Workflow

```bash
cat .github/workflows/build-and-publish.yml
grep -n -E 'docker build|docker login|docker push|github.sha|secrets.|needs:|scan|permissions:' .github/workflows/*.yml
```

PowerShell:

```powershell
Get-Content .github/workflows/build-and-publish.yml
Select-String -Path .github/workflows/*.yml -Pattern 'docker build|docker login|docker push|github.sha|secrets.|needs:|scan|permissions:'
```

## 8. Run an Image Health Check

```bash
docker run -d --name mern-api-ci -p 3000:3000 "$IMAGE"
curl -i http://localhost:3000/health
docker logs --tail 100 mern-api-ci
docker stop mern-api-ci
docker rm mern-api-ci
```

PowerShell:

```powershell
Invoke-WebRequest http://localhost:3000/health
```

## 9. Diagnose Push Failures

For `denied: requested access to the resource is denied`, check:

```text
Registry URL -> Repository path -> Login -> Token validity -> Push permission
```

For `manifest unknown`, compare:

```text
Git SHA -> CI image tag -> Registry tag -> Deployment image tag
```

Do not rebuild until you know the failure is related to the image itself. Authentication and authorization failures do not require a new build.

## 10. Review Changes and Secret Safety

```bash
git diff -- .github/workflows
git status --short
git diff --cached -- .github/workflows
git grep -n -I -E 'password|token|secret|BEGIN .* PRIVATE KEY' -- . ':!package-lock.json'
```

Treat any real credential exposed in a file or log as compromised and rotate it immediately.
