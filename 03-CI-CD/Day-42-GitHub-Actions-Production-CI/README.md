# DevOps Mentorship Program - Day 42

## Phase 3: CI/CD

### GitHub Actions: Designing a Production CI Pipeline

**Level:** Intermediate -> Professional  
**Focus:** GitHub Actions, runners, workflows, jobs, steps, caching, artifacts, secrets, dependencies, and a MERN production CI pipeline

Day 41 introduced the CI/CD lifecycle. Today we turn that theory into an actual CI pipeline design using GitHub Actions.

The pipeline we are building is intentionally focused on continuous integration. It validates code, creates a repeatable build, and prepares a Docker artifact. It does not deploy to production yet.

```text
Git push or pull request
          |
          v
   GitHub Actions workflow
          |
          v
        Runner
          |
          +-- Checkout
          +-- Setup Node.js
          +-- npm ci
          +-- Lint
          +-- Test
          +-- Build
          +-- Docker build
          +-- Security checks
          |
          v
   Versioned CI artifact
```

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain what GitHub Actions is.
- Distinguish a workflow, job, step, and runner.
- Design a basic CI pipeline for a Node.js or MERN application.
- Trigger CI for pull requests and pushes to the main branch.
- Explain why `npm ci` is preferred in CI when a lockfile exists.
- Configure dependency caching to improve pipeline speed.
- Explain the difference between a cache and an artifact.
- Use environment variables and secrets appropriately.
- Make jobs depend on earlier quality gates.
- Prevent image publication or deployment when tests fail.
- Tag Docker images with the Git commit SHA for traceability.
- Separate validation in CI from controlled deployment in CD.
- Explain a production-quality GitHub Actions pipeline in an interview.

---

## 2. GitHub Actions Mental Model

GitHub Actions is a programmable automation engine attached to a GitHub repository. Repository events, such as a push or pull request, can trigger workflows.

```text
Developer runs git push
          |
          v
       GitHub event
          |
          v
        Workflow
          |
          v
          Job
          |
          v
         Steps
          |
          v
   Commands run on a runner
```

A typical Node.js CI execution might be:

```text
Checkout source code
          |
          v
Install the required Node.js version
          |
          v
Install locked dependencies with npm ci
          |
          v
Run linting
          |
          v
Run tests
          |
          v
Build the application
```

The workflow file describes what should happen. GitHub provides or connects the runner that executes the commands.

---

## 3. The Four Terms You Must Know

These terms frequently appear in interviews and in every GitHub Actions workflow.

### 3.1 Workflow

A **workflow** is the complete automation definition stored as a YAML file in `.github/workflows/`.

Examples:

```text
.github/workflows/ci.yml
.github/workflows/deploy.yml
.github/workflows/security.yml
```

A workflow defines:

- Its name.
- The events that trigger it.
- The jobs it contains.
- Permissions, variables, and optional concurrency rules.

### 3.2 Job

A **job** is a logical unit of work executed on a runner.

Examples:

```text
lint
unit-tests
build
security
publish-image
```

Jobs run independently by default. A job can depend on another job with `needs`.

### 3.3 Step

A **step** is one operation inside a job. A step may use a reusable action or run a shell command.

Examples:

```yaml
- uses: actions/checkout@v4

- name: Install dependencies
  run: npm ci
```

Steps within one job execute in order and share the job workspace.

### 3.4 Runner

A **runner** is the machine or execution environment that runs a job.

```yaml
runs-on: ubuntu-latest
```

This requests a GitHub-hosted Ubuntu runner. Other choices include Windows, macOS, or a self-hosted runner managed by your organization.

A runner provides the operating system, shell, tools, workspace, and network environment required by the job.

---

## 4. Where Workflows Live

GitHub Actions workflows must be stored in this repository location:

```text
.github/
└── workflows/
    └── ci.yml
```

The filename can be different, but it must end in `.yml` or `.yaml` and be inside `.github/workflows/`.

A repository can have multiple workflows. For example:

```text
.github/workflows/
├── ci.yml
├── security.yml
└── deploy.yml
```

A useful separation is:

- `ci.yml`: lint, test, build, and validation.
- `security.yml`: dependency and container scanning.
- `deploy.yml`: release and environment deployment.

---

## 5. A Minimal MERN CI Workflow

The following workflow validates a Node.js application on pull requests and pushes to `main`.

```yaml
name: MERN CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run lint
        run: npm run lint

      - name: Run tests
        run: npm test
```

The execution flow is:

```text
Pull request or push to main
            |
            v
     Ubuntu runner starts
            |
            v
       Source is checked out
            |
            v
       Node.js 22 is installed
            |
            v
       npm ci installs dependencies
            |
            v
       Lint and tests execute
```

This is a validation workflow. It does not publish an image or deploy an application.

---

## 6. The `on` Trigger

The `on` section specifies when GitHub Actions should start a workflow.

### Push to the main branch

```yaml
on:
  push:
    branches:
      - main
```

This runs when commits are pushed to `main`.

### Pull requests targeting main

```yaml
on:
  pull_request:
    branches:
      - main
```

This runs when a pull request targets `main`. It gives the repository a quality gate before merging.

### Manual execution

```yaml
on:
  workflow_dispatch:
```

This adds a manual **Run workflow** option in the GitHub Actions interface.

### Scheduled execution

```yaml
on:
  schedule:
    - cron: '0 3 * * 1'
```

This example runs every Monday at 03:00 UTC. Schedules are useful for recurring security or maintenance checks.

A common production arrangement is:

```text
Pull request
    |
    v
CI validation only

Merge to main
    |
    v
Build and release workflow
```

---

## 7. Pull Request CI as a Quality Gate

Suppose a developer changes a feature branch:

```text
feature/order-discount
          |
          v
       Pull request
          |
          v
         main
```

The pull request workflow should run:

```text
Lint
  |
  v
Unit tests
  |
  v
Integration tests
  |
  v
Application build
```

If a test fails:

```text
Pull request
     |
     v
  CI failed
     |
     v
Do not merge until fixed
```

If all required checks pass:

```text
Pull request
     |
     v
  CI passed
     |
     v
Review and merge
```

This is the practical value of CI: bad code is identified before it reaches a shared or production environment.

For a MERN application, an accidental change such as:

```javascript
return items.reduce((sum, item) => sum + item.price);
```

instead of:

```javascript
return items.reduce((sum, item) => sum + item.price, 0);
```

should be detected by tests. The failed check should prevent the pull request from being merged and should prevent downstream publishing or deployment jobs from running.

---

## 8. `npm ci` Versus `npm install`

For CI, prefer:

```bash
npm ci
```

when the repository contains a `package-lock.json` file.

The dependency model is:

```text
package.json + package-lock.json
              |
              v
            npm ci
              |
              v
      Reproducible dependency tree
```

`npm ci` is designed for automated environments. It performs a clean installation based on the lockfile and normally fails if the lockfile and `package.json` do not agree.

Use `npm install` primarily when developing locally or when intentionally changing the dependency tree. If the dependency manifest changes, regenerate and commit the lockfile.

A deterministic CI installation helps ensure that:

- Local and CI dependency versions match.
- Builds are repeatable.
- Unexpected dependency upgrades do not enter the pipeline silently.
- A clean runner does not inherit packages from a previous run.

---

## 9. Dependency Caching

Installing hundreds of packages on every run can make CI slow. GitHub Actions can cache npm's download cache.

With `actions/setup-node`, the usual configuration is:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: 22
    cache: npm
```

The cache speeds up future installations, but it must not be required for correctness.

```text
First run
   |
   +-- Download packages
   +-- Populate cache

Later run
   |
   +-- Restore cache
   +-- Install more quickly
```

A correct pipeline must still work when:

- The cache is empty.
- The cache expires.
- The cache key changes.
- The cache is temporarily unavailable.

Caching improves speed. It does not replace `npm ci`, tests, or build validation.

---

## 10. Cache Versus Artifact

These concepts are easy to confuse.

### Cache

A cache is temporary reusable data intended to make future workflow runs faster.

Examples:

```text
npm package download cache
Docker build cache
Compiled dependency cache
```

Mental model:

```text
Cache = Help the next run do less repeated work.
```

### Artifact

An artifact is a useful output produced by a workflow and saved for later inspection or use.

Examples:

```text
Build directory
Coverage report
Test-results.xml
Debug logs
Docker metadata
```

Mental model:

```text
Artifact = Preserve the result of this run.
```

A cache can expire without affecting correctness. An artifact may be needed for auditability, debugging, release promotion, or another job.

---

## 11. Uploading a Build Artifact

For a frontend build that creates `dist/`, an artifact can be uploaded like this:

```yaml
- name: Build application
  run: npm run build

- name: Upload build artifact
  uses: actions/upload-artifact@v4
  with:
    name: frontend-dist
    path: dist/
    if-no-files-found: error
```

A test report can also be uploaded:

```yaml
- name: Upload test results
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: test-results/
```

For a containerized production architecture, the final release artifact is often a Docker image rather than a raw `dist/` directory. Both are useful, but they serve different release designs.

---

## 12. Environment Variables and Secrets

Workflows often need configuration values. Separate non-sensitive configuration from credentials.

### Variables

A variable is normally safe to expose to the workflow logs or repository configuration when it is not confidential.

Examples:

```text
NODE_ENV=production
APP_PORT=3000
IMAGE_NAME=mern-api
```

### Secrets

A secret contains sensitive information and must be stored in GitHub's encrypted secrets or environment secret mechanism.

Examples:

```text
Registry token
Cloud credentials
MongoDB password
Deployment key
```

Do not write credentials directly in a workflow:

```yaml
# Never do this
password: myProductionPassword
```

Use a secret reference instead:

```yaml
- name: Login to container registry
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.REGISTRY_USERNAME }}
    password: ${{ secrets.REGISTRY_TOKEN }}
```

The source repository should contain the workflow logic, not the secret values.

```text
Configuration
      |
      +-- Non-sensitive -> variables
      |
      +-- Sensitive ----> secrets
```

Secrets should also be scoped as narrowly as practical. A job should receive only the credentials it needs.

---

## 13. Jobs and Dependencies

Jobs run independently by default. Use `needs` to create an explicit dependency graph.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: npm run build

  publish:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Publish trusted artifact"
```

The execution graph is:

```text
test
  |
  v
build
  |
  v
publish
```

If `test` fails:

```text
test fails
    |
    v
build is skipped
    |
    v
publish is skipped
```

This is a critical release safety mechanism. The workflow should publish only artifacts that have passed the required quality gates.

Independent jobs can run in parallel:

```text
          +-- Lint --------+
          |                |
          +-- Unit tests --+--> Docker build
          |                |
          +-- Security ----+
```

Parallelism can reduce wall-clock time, but only use it when the jobs do not depend on each other's outputs.

---

## 14. Build and Docker Image Tagging

A Docker image should be traceable to the source commit that produced it.

GitHub exposes the commit SHA through:

```text
${{ github.sha }}
```

A Docker build can use it as a tag:

```yaml
- name: Build Docker image
  run: docker build -t mern-api:${{ github.sha }} ./backend
```

The relationship becomes:

```text
Git commit: abc123
      |
      v
Docker image: mern-api:abc123
```

A registry can then contain:

```text
mern-api:abc123
mern-api:def456
mern-api:789abc
```

The commit SHA provides:

- Traceability from image to source code.
- A stable release reference.
- A reproducible rollback target.
- A way to compare deployed images with Git history.

Avoid relying only on `latest`. A mutable tag does not clearly identify which commit is running.

---

## 15. A Complete CI Foundation Workflow

This example combines pull request validation, dependency caching, build validation, and Docker image construction.

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
    name: Validate application
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run lint
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Build application
        run: npm run build

  docker-build:
    name: Build Docker image
    needs: validate
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Build image tagged with commit SHA
        run: docker build -t "${{ env.IMAGE_NAME }}:${{ github.sha }}" ./backend

      - name: Display image tag
        run: echo "Built ${{ env.IMAGE_NAME }}:${{ github.sha }}"
```

Important behavior:

- Pull requests and pushes to `main` trigger validation.
- Dependencies are installed with `npm ci`.
- The Node version is controlled.
- Lint, tests, and build must pass first.
- The Docker job depends on `validate`.
- A failed validation job prevents Docker build execution.
- The image is tagged with the commit SHA.
- No deployment is performed.

If the frontend and backend are separate applications, repeat the appropriate setup inside each job or use a matrix strategy. Do not assume that a single root `package.json` exists unless the repository actually uses that structure.

---

## 16. Separating CI from CD

A useful production design separates validation from deployment.

### CI workflow

```text
Pull request or push
          |
          v
Lint -> Test -> Build -> Docker build -> Scan
```

### CD workflow

```text
Trusted image
      |
      v
Registry -> Staging -> Health check -> Approval -> Production
```

This separation means that a pull request can be validated without deploying to production.

A typical policy is:

```text
Pull request
    |
    v
Run CI checks only

Merge to main
    |
    v
Build and publish a versioned image

Release approval
    |
    v
Deploy to production
```

Separating CI and CD makes responsibilities clearer and reduces accidental production changes.

---

## 17. Environment Protection

Production should have stronger controls than staging.

```text
Staging
   |
   +-- Automatic deployment

Production
   |
   +-- Required approval
   +-- Restricted secrets
   +-- Protected branch or environment
```

GitHub environments can be configured with:

- Required reviewers.
- Environment-specific secrets.
- Deployment branch restrictions.
- Approval rules.

The goal is to combine automated validation with controlled release:

```text
Automated checks
       +
Controlled production promotion
```

Avoid a design where every commit automatically reaches production before the organization has the required tests, monitoring, rollback, and approval controls.

---

## 18. Production Pipeline Design

A mature CI/CD architecture may look like this:

```text
                         Pull request
                              |
                              v
                         GitHub Actions
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
            Lint            Tests            Build
              |               |               |
              +---------------+---------------+
                              |
                              v
                         Security scan
                              |
                              v
                         Docker build
                              |
                              v
                       Image vulnerability scan
                              |
                              v
                           Registry
                              |
                              v
                           Staging
                              |
                              v
                        Health check
                              |
                              v
                           Approval
                              |
                              v
                         Production
```

The important rule is that an artifact should move forward only after the required gates pass.

---

## 19. Improving a Slow Pipeline

Suppose a CI pipeline takes 20 minutes and the target is 8 minutes. Start by measuring rather than guessing.

```text
Measure duration
       |
       v
Find the slowest stages
       |
       v
Parallelize independent jobs
       |
       v
Cache dependencies
       |
       v
Avoid unnecessary work
       |
       v
Optimize tests and Docker builds
```

Before:

```text
Lint
  |
  v
Unit tests
  |
  v
Integration tests
  |
  v
Docker build
```

Possible design:

```text
          +-- Lint -----------+
          |                   |
          +-- Unit tests -----+---> Docker build
          |                   |
          +-- Integration ----+
```

Other practical improvements include:

- Cache npm downloads.
- Avoid rebuilding unchanged services.
- Split frontend and backend jobs where appropriate.
- Run independent test suites in parallel.
- Upload and reuse build artifacts.
- Use Docker layer caching.
- Run the smallest relevant test set on pull requests when policy allows.
- Keep full regression checks for protected branches or scheduled workflows.
- Measure each stage before and after an optimization.

Adding more runners is not automatically the answer. The bottleneck may be dependency installation, serial jobs, slow integration tests, Docker layers, or an inefficient test setup.

---

## 20. Production Interview Challenge

Be able to explain this pipeline without looking at your notes:

```text
Developer
    |
    v
Pull request
    |
    v
GitHub Actions
    |
    v
Runner
    |
    v
npm ci
    |
    v
Lint
    |
    v
Tests
    |
    v
Build
    |
    v
Docker image
    |
    v
Git SHA tag
```

Then answer:

> If tests fail, why should the Docker image not be pushed?

A strong answer is:

> The pipeline should publish only artifacts that have passed the required quality gates. Publishing a failed build creates ambiguity about which artifacts are trusted and increases the chance that bad code reaches a deployment environment.

---

## 21. Day 42 Summary

The central mental model is:

```text
Git event
    |
    v
Workflow
    |
    v
Runner
    |
    v
Job
    |
    v
Steps
    |
    v
Quality gates
    |
    v
Artifact
```

The MERN CI pipeline is:

```text
             Git
              |
              v
        GitHub Actions
              |
       +------+------+------+
       v      v      v      |
     Lint    Test   Build   |
       |      |      |      |
       +------+------+------+
              |
              v
        Docker build
              |
              v
       mern-api:<SHA>
```

Remember these distinctions:

> **Workflow = automation definition. Job = unit of work. Step = individual operation. Runner = machine that executes it.**

> **Cache makes CI faster. Artifacts carry useful outputs forward.**

> **A failed quality gate should stop untrusted artifacts from moving toward deployment.**

---

## 22. Next Lesson

The next lesson continues the CI/CD sequence with Docker image publishing and registry integration from GitHub Actions:

```text
GitHub Actions
      |
      v
Docker build
      |
      v
Image scan
      |
      v
Registry login
      |
      v
Docker push
      |
      v
Versioned image
```

Today's pipeline answers:

```text
Does the code work?
```

The next pipeline will also answer:

```text
Can we produce and publish a trusted production artifact?
```
