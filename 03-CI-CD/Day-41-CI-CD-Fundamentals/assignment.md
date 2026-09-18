# Day 41 - Assignment: Build Your First CI Pipeline

## Objective

Create a CI pipeline for a MERN backend that validates code before a Docker image is produced. The pipeline should install dependencies, run quality checks, execute tests, build the application, and create an image tagged with the Git commit SHA.

Do not deploy to production in this assignment. The goal is to establish reliable CI gates first.

---

## Scenario

Assume the backend has this structure:

```text
backend/
+-- package.json
+-- package-lock.json
+-- src/
+-- tests/
+-- Dockerfile
```

The desired flow is:

```text
Git push
   |
   v
npm ci
   |
   v
npm run lint
   |
   v
npm test
   |
   v
npm run build
   |
   v
docker build -t mern-api:<git-sha> ./backend
```

Every failed required step must prevent the image stage from running.

---

## Part 1: Pipeline Design

Draw or describe the pipeline stages and answer:

1. What event starts the pipeline?
2. Which branch or pull request events should run CI?
3. Which steps are required gates?
4. What happens when lint fails?
5. What happens when tests fail?
6. Which step creates the artifact?
7. How will the artifact be identified later?

Use this outline:

```text
Trigger:
Source revision:
Runner/runtime:
Install command:
Lint command:
Unit test command:
Integration test command:
Build command:
Docker tag:
Required gates:
```

---

## Part 2: Backend CI Preparation

Verify that the backend provides appropriate scripts in `package.json`:

```json
{
  "scripts": {
    "lint": "...",
    "test": "...",
    "build": "..."
  }
}
```

Run the commands locally before adding them to CI:

```bash
npm ci
npm run lint
npm test
npm run build
```

If a command does not exist, document whether you will add it or omit that stage temporarily. Do not hide a missing check by naming a command that does not run meaningful validation.

---

## Part 3: Create a CI Workflow

Create a workflow in:

```text
.github/workflows/ci.yml
```

The workflow should:

- Run on pull requests and pushes to the main development branch.
- Check out the repository.
- Set up a documented Node.js version.
- Install dependencies with `npm ci`.
- Run linting.
- Run unit tests.
- Run integration tests where practical.
- Build the application.
- Build the backend Docker image only after earlier gates pass.
- Tag the image with the Git commit SHA.
- Avoid storing credentials in the workflow file.

A conceptual job is:

```text
jobs:
  ci:
    checkout
    setup node
    npm ci
    npm run lint
    npm test
    npm run build
    docker build -t mern-api:<sha> ./backend
```

Adapt paths and commands to the actual repository layout.

---

## Part 4: Failure Tests

Prove that pipeline gates work. In a temporary branch, introduce each failure separately:

### Lint failure

```text
Lint FAIL -> tests/build/image stages do not publish an artifact
```

### Test failure

```text
Lint PASS -> tests FAIL -> build/image stages do not publish an artifact
```

### Build failure

```text
Lint PASS -> tests PASS -> build FAIL -> image publication does not proceed
```

Record the observed job status and the useful error message. Restore the branch after each controlled test.

---

## Part 5: Image Traceability

Use the commit SHA as the image tag. Record:

```text
Git commit SHA:
Image name:
Image tag:
Full image reference:
Dockerfile path:
Build timestamp:
```

The desired format is similar to:

```text
mern-api:abc123def456
```

Explain why `latest` alone is insufficient for production rollback and incident investigation.

---

## Part 6: Health Test

Add or verify an API endpoint:

```text
GET /health
```

It should return HTTP 200 and a small response such as:

```json
{
  "status": "ok"
}
```

Run the application in a test environment and verify it:

```bash
curl -i http://localhost:<port>/health
```

PowerShell alternative:

```powershell
Invoke-WebRequest http://localhost:<port>/health
```

Document whether this is a liveness check, readiness check, or both. Explain what dependency failures should do to readiness.

---

## Part 7: Secrets and Pull Requests

Document how the pipeline handles:

- Registry credentials.
- Database credentials.
- Deployment credentials.
- Pull requests from forks.
- Production-only secrets.

The solution must state that secrets are stored in a CI/CD secret store or external secret manager, are limited to trusted jobs, and are not printed in logs.

---

## Part 8: Required Deliverables

Submit:

```text
.github/workflows/ci.yml
Pipeline diagram or written stage description
Failure-test notes
Image traceability record
Health endpoint test result
Secrets and permissions explanation
```

---

## Part 9: Completion Checklist

```text
[ ] CI runs on the intended Git events
[ ] Node.js version is explicit
[ ] npm ci uses the lockfile
[ ] Lint is a required gate
[ ] Unit tests are a required gate
[ ] Integration tests are included where practical
[ ] Application build is validated
[ ] Docker image builds after earlier gates
[ ] Image is tagged with the Git SHA
[ ] Failed gates stop downstream work
[ ] Secrets are not committed
[ ] Fork pull requests receive limited permissions
[ ] /health returns HTTP 200
[ ] Pipeline result and artifact reference are recorded
```

## Reflection

Answer in a short paragraph:

> How does your pipeline reduce the risk of a broken change reaching production, and what additional stages would you add before enabling deployment?
