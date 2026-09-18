# Day 41 - Commands: CI/CD Fundamentals

Replace paths, branch names, image names, and ports with values from your project.

## 1. Inspect the Project

```bash
git status
git log -1 --oneline
node --version
npm --version
```

PowerShell:

```powershell
git status
git log -1 --oneline
node --version
npm --version
```

Find package scripts:

```bash
npm run
```

Inspect the backend package file:

```bash
cat backend/package.json
```

PowerShell:

```powershell
Get-Content backend/package.json
```

## 2. Install Reproducibly

Use the lockfile-aware install in CI:

```bash
cd backend
npm ci
```

Do not silently replace `npm ci` with `npm install` in CI. Investigate lockfile changes and dependency errors instead.

## 3. Run Local CI Checks

```bash
npm run lint
npm test
npm run build
```

If the project uses a different test command, record the actual command in the workflow and documentation.

## 4. Get the Git Commit SHA

Linux/macOS/Git Bash:

```bash
git rev-parse HEAD
git rev-parse --short HEAD
```

PowerShell:

```powershell
git rev-parse HEAD
git rev-parse --short HEAD
```

Set a local image tag:

PowerShell:

```powershell
$GitSha = git rev-parse --short HEAD
docker build -t "mern-api:$GitSha" .\backend
```

Bash:

```bash
GIT_SHA=$(git rev-parse --short HEAD)
docker build -t "mern-api:$GIT_SHA" ./backend
```

## 5. Inspect the CI Image

```bash
docker image inspect mern-api:<git-sha>
docker history --no-trunc mern-api:<git-sha>
docker run --rm mern-api:<git-sha> id
```

Check that the image does not contain credentials, development files, or an unexpected root runtime user.

## 6. Run the Image for a Health Test

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

Stop a test container if it was started detached:

```bash
docker stop mern-api-ci
docker logs --tail 100 mern-api-ci
```

## 7. Test a Compose-Based Service

```bash
docker compose config
docker compose up -d mongodb api
docker compose ps
docker compose logs --tail 100 api
```

Check service health:

```bash
docker inspect --format='{{json .State.Health}}' <container-name>
```

PowerShell uses the same Docker commands, but quoting may need adjustment depending on the shell.

## 8. Validate the CI Workflow Locally

Review the workflow file:

```bash
cat .github/workflows/ci.yml
```

PowerShell:

```powershell
Get-Content .github/workflows/ci.yml
```

A local workflow runner such as `act` can be used if it is approved for the project:

```bash
act pull_request
```

Treat local workflow emulation as useful feedback, not as a replacement for the hosted runner.

## 9. Review Git Changes

```bash
git diff -- .github/workflows/ci.yml backend/package.json
git status --short
```

Confirm that no `.env` files, tokens, private keys, or generated secrets were added.

## 10. Record Artifact Metadata

```bash
docker image inspect --format='{{.Id}}' mern-api:<git-sha>
docker image inspect --format='{{json .RepoDigests}}' mern-api:<git-sha>
docker images mern-api
```

Record:

```text
Git SHA:
Image tag:
Image ID:
Image digest:
Pipeline run:
Test result:
Scan result:
```

## 11. Check for Accidental Secrets

Search tracked files carefully and review matches manually:

```bash
git grep -n -i -E 'password|secret|token|api[_-]?key|private[_-]?key'
```

Do not paste secret values into terminal output or CI logs. If a real credential was committed, revoke or rotate it immediately.

## 12. Clean Up Test Resources

```bash
docker stop mern-api-ci
docker compose down
```

Do not use `docker compose down -v` unless deleting test volumes is intentional.
