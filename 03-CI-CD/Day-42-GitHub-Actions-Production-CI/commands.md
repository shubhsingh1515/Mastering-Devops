# Day 42 - Commands: GitHub Actions Production CI

Replace paths, branches, image names, and ports with values from your project.

---

## 1. Inspect Repository and Runtime

```bash
git status
git branch --show-current
git log -1 --oneline
node --version
npm --version
```

PowerShell uses the same commands:

```powershell
git status
git branch --show-current
git log -1 --oneline
node --version
npm --version
```

List package scripts:

```bash
npm run
```

PowerShell:

```powershell
npm run
Get-Content package.json
```

---

## 2. Install Dependencies Like CI

```bash
npm ci
```

Check that a lockfile exists:

```bash
ls package-lock.json
```

PowerShell:

```powershell
Test-Path package-lock.json
```

Use `npm install` only when intentionally changing dependencies or regenerating the lockfile. CI should normally use the committed lockfile with `npm ci`.

---

## 3. Run the Local Quality Gates

```bash
npm run lint
npm test
npm run build
```

Run one command and stop on failure in Bash:

```bash
npm run lint && npm test && npm run build
```

PowerShell:

```powershell
npm run lint
if ($LASTEXITCODE -eq 0) { npm test }
if ($LASTEXITCODE -eq 0) { npm run build }
```

---

## 4. Inspect the Workflow

```bash
cat .github/workflows/ci.yml
```

PowerShell:

```powershell
Get-Content .github/workflows/ci.yml
```

Search for the important controls:

```bash
grep -n -E 'pull_request|push:|npm ci|needs:|github.sha|cache: npm' .github/workflows/ci.yml
```

PowerShell:

```powershell
Select-String -Path .github/workflows/ci.yml -Pattern 'pull_request|push:|npm ci|needs:|github.sha|cache: npm'
```

---

## 5. Find the Git Commit SHA

```bash
git rev-parse HEAD
git rev-parse --short HEAD
```

PowerShell:

```powershell
git rev-parse HEAD
git rev-parse --short HEAD
```

GitHub Actions exposes the full commit SHA as:

```text
${{ github.sha }}
```

---

## 6. Build a SHA-Tagged Docker Image

Bash:

```bash
GIT_SHA=$(git rev-parse --short HEAD)
docker build -t "mern-api:$GIT_SHA" ./backend
```

PowerShell:

```powershell
$GitSha = git rev-parse --short HEAD
docker build -t "mern-api:$GitSha" .\backend
```

Inspect the image:

```bash
docker image inspect mern-api:<git-sha>
docker history --no-trunc mern-api:<git-sha>
docker images mern-api
```

---

## 7. Run a Local Image Health Check

```bash
docker run --rm --name mern-api-ci -p 3000:3000 mern-api:<git-sha>
```

In another terminal:

```bash
curl -i http://localhost:3000/health
```

PowerShell:

```powershell
Invoke-WebRequest http://localhost:3000/health
```

For a detached container:

```bash
docker run -d --name mern-api-ci -p 3000:3000 mern-api:<git-sha>
docker logs --tail 100 mern-api-ci
docker stop mern-api-ci
docker rm mern-api-ci
```

---

## 8. Validate Docker and Compose Configuration

```bash
docker version
docker build --check ./backend
docker compose config
docker compose ps
```

Start only the services required for a local check:

```bash
docker compose up -d mongodb api
docker compose logs --tail 100 api
docker compose down
```

Do not use `docker compose down -v` unless deleting test volumes is intentional.

---

## 9. Check Image Metadata

```bash
docker image inspect --format='{{.Id}}' mern-api:<git-sha>
docker image inspect --format='{{json .Config.Labels}}' mern-api:<git-sha>
docker image inspect --format='{{json .RepoDigests}}' mern-api:<git-sha>
```

Review that the image does not contain credentials, local `.env` files, unnecessary development files, or an unexpected runtime user.

---

## 10. Optional Local Workflow Emulation

If the project approves a local GitHub Actions emulator such as `act`, run:

```bash
act pull_request
```

Treat local emulation as useful feedback, not a replacement for the hosted GitHub runner. The hosted runner remains the source of truth for the repository check.

---

## 11. Review Changes and Secrets

```bash
git diff -- .github/workflows/ci.yml
git status --short
git grep -n -i -E 'password|secret|token|api[_-]?key|private[_-]?key'
```

PowerShell:

```powershell
git diff -- .github/workflows/ci.yml
git status --short
Select-String -Path .github\workflows\ci.yml,package.json -Pattern 'password|secret|token|api[_-]?key|private[_-]?key'
```

Review matches manually. Never print real credentials into terminal output or CI logs. If a real credential was committed, revoke or rotate it immediately.

---

## 12. Record CI Evidence

Record the following after a successful run:

```text
Workflow name:
Run number:
Commit SHA:
Runner:
Node version:
Cache enabled:
Lint result:
Test result:
Build result:
Docker tag:
Artifact name:
```

For a failed run, record the failed job, failed step, error message, and whether dependent jobs were skipped.

---

## 13. Clean Up Local Resources

```bash
docker stop mern-api-ci
docker rm mern-api-ci
docker compose down
```

Commands may report that a resource does not exist. Confirm that no required development service is being stopped before cleanup.
