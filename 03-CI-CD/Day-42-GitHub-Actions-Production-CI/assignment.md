# Day 42 - Assignment: Build a GitHub Actions CI Pipeline

## Objective

Create a GitHub Actions workflow for a MERN application that validates code before a Docker image can be built. The workflow must run on pull requests and pushes to `main`, install dependencies reproducibly, run quality checks, build the application, and tag the Docker image with the Git commit SHA.

Do not deploy to production. This assignment is about creating a reliable CI foundation.

---

## Scenario

Assume the project has a backend with this structure:

```text
backend/
+-- package.json
+-- package-lock.json
+-- src/
+-- tests/
+-- Dockerfile
```

The desired pipeline is:

```text
Pull request or push to main
            |
            v
         Checkout
            |
            v
        Setup Node.js
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
 docker build -t mern-api:<sha> ./backend
```

Every required failure must stop downstream work.

---

## Part 1: Inspect the Application

Record the actual values used by your project:

```text
Repository layout:
Node.js version:
Package manager:
Package lockfile:
Lint command:
Test command:
Build command:
Backend directory:
Dockerfile path:
Application port:
Health endpoint:
```

Verify that `package.json` contains meaningful scripts for the commands you plan to run. Do not add a workflow command that does not exist or does not perform a real check.

---

## Part 2: Run Checks Locally

From the correct application directory, run:

```bash
npm ci
npm run lint
npm test
npm run build
```

Document the result of every command. If the project uses different scripts, record the actual commands and explain why.

---

## Part 3: Create the Workflow

Create:

```text
.github/workflows/ci.yml
```

The workflow must:

- Run for pull requests targeting `main`.
- Run for pushes to `main`.
- Use a controlled Node.js version.
- Use `actions/checkout`.
- Use `actions/setup-node`.
- Install with `npm ci`.
- Enable npm dependency caching.
- Run linting and tests.
- Run the application build.
- Build the Docker image only after validation passes.
- Tag the image with `${{ github.sha }}`.
- Avoid hard-coded credentials.
- Use `needs` so Docker construction depends on validation.

Start with this structure and adapt paths to your project:

```yaml
name: MERN CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

permissions:
  contents: read

env:
  NODE_VERSION: 22
  IMAGE_NAME: mern-api

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: npm

      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build

  docker-build:
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t "${{ env.IMAGE_NAME }}:${{ github.sha }}" ./backend
```

---

## Part 4: Add an Artifact

Upload a build directory or test report when your application produces one:

```yaml
- name: Upload test or build artifact
  uses: actions/upload-artifact@v4
  with:
    name: ci-output
    path: dist/
    if-no-files-found: error
```

Explain whether your uploaded artifact is intended for debugging, later jobs, release promotion, or audit evidence.

---

## Part 5: Prove Job Dependencies

Make one test fail temporarily. Push the change to a test branch or open a draft pull request.

Record whether the result matches this expectation:

```text
validate       failed
Docker build   skipped
Publish        not present
Deploy         not present
```

Restore the test and verify that the Docker job runs after validation succeeds.

---

## Part 6: Trace the Image

For a successful local Docker build, record:

```text
Git commit SHA:
Short SHA:
Image name:
Image tag:
Full image reference:
Dockerfile path:
Image ID:
```

Explain why `mern-api:latest` alone is weaker than `mern-api:<commit-sha>` for rollback and incident investigation.

---

## Part 7: Secrets and Trust Boundaries

Write a short policy explaining:

- Which values are variables and which are secrets.
- Where registry credentials will be stored later.
- Why production secrets must not be available to pull request jobs from forks.
- Why credentials must not be echoed into logs.
- Which jobs need permission to publish or deploy.

---

## Part 8: Required Deliverables

Submit:

```text
[ ] .github/workflows/ci.yml
[ ] Local command results
[ ] Successful CI run link or screenshot
[ ] Failed-test run evidence
[ ] Image SHA and tag record
[ ] Cache and artifact explanation
[ ] Secrets and permissions explanation
[ ] Short reflection on what would be moved into CD
```

## Success Criteria

The assignment is complete when a pull request runs CI, a failed validation prevents Docker construction, a successful validation builds the SHA-tagged image, and the workflow contains no hard-coded credentials.
