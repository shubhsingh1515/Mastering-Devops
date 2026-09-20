# DevOps Mentorship Program - Week 7 Sunday Revision

## Phase 3: CI/CD

### Day 43 - Weekly CI/CD Revision and Practical Assessment

**Type:** Sunday revision, quiz, and practical assessment  
**Level:** Intermediate to Professional   
**Focus:** CI/CD fundamentals, GitHub Actions, Docker artifact flow, and MERN production delivery

Today consolidates Days 41 and 42. Day 41 introduced CI/CD fundamentals, delivery models, pipeline gates, artifacts, and release safety. Day 42 applied those ideas through GitHub Actions workflows, runners, jobs, steps, caching, secrets, and a SHA-tagged Docker build.

The goal is not to introduce another isolated tool. The goal is to connect the concepts into one explainable delivery system and prove that the pipeline stops unsafe changes.

---

## Weekly Progression

```text
Docker production practices
          |
          v
CI/CD fundamentals
          |
          v
Continuous Integration
          |
          v
Continuous Delivery
          |
          v
Continuous Deployment
          |
          v
GitHub Actions
          |
          v
Workflows / Jobs / Steps / Runners
          |
          v
Caching / Artifacts / Secrets
          |
          v
MERN CI pipeline
          |
          v
SHA-tagged Docker artifact
```

The central principle is:

> **CI validates the change. CD safely moves a trusted artifact toward production.**

---

## 1. Today's Objectives

By the end of this revision, you should be able to:

- Explain Continuous Integration, Continuous Delivery, and Continuous Deployment.
- Design a basic GitHub Actions pipeline for a Node.js or MERN application.
- Explain the relationship between a workflow, job, step, and runner.
- Explain why `npm ci` is preferred in CI when a lockfile exists.
- Distinguish a dependency cache from a workflow artifact.
- Explain CI secrets, variables, permissions, and security boundaries.
- Build a Docker image from a CI workflow.
- Tag an image with the Git commit SHA.
- Explain why failed tests must block release artifacts.
- Investigate why a pipeline passes locally but fails in CI.
- Explain health checks, rollback, and artifact traceability.
- Complete the practical pipeline assessment.

---

## 2. Weekly Mental Model

A production-oriented CI/CD system for the MERN project is becoming:

```text
Developer
    |
    v
Git push or pull request
    |
    v
GitHub Actions workflow
    |
    v
Runner
    |
    +-- npm ci
    +-- Lint
    +-- Unit tests
    +-- Integration tests
    +-- Application build
    |
    v
Docker build
    |
    v
Image scan
    |
    v
Container registry
    |
    v
Staging
    |
    v
Health and smoke checks
    |
    v
Production approval
    |
    v
Production
    |
    v
Rollback if required
```

Every stage has a purpose:

- **Git event:** identifies when validation should start.
- **Runner:** provides the execution environment.
- **Validation steps:** check code quality and behavior.
- **Docker build:** packages the application into a deployable artifact.
- **Image scan:** checks the artifact for known vulnerabilities.
- **Registry:** stores the versioned image.
- **Staging:** provides a controlled environment for release verification.
- **Health checks:** confirm that the deployed application is actually ready.
- **Rollback:** returns to a previous known-good artifact when required.

---

## 3. Revision: CI, Continuous Delivery, and Continuous Deployment

### Continuous Integration

Continuous Integration means that developers frequently integrate changes into a shared repository and automated checks validate those changes.

```text
Code change
    |
    v
Integrate
    |
    v
Lint
    |
    v
Test
    |
    v
Build
```

CI answers:

> **Is this change technically acceptable?**

A CI pipeline should provide fast feedback while the change is still easy to understand and fix.

### Continuous Delivery

Continuous Delivery extends CI by keeping the software in a deployable state.

```text
Validated source
       |
       v
Build artifact
       |
       v
Test and scan artifact
       |
       v
Ready to release
```

Continuous Delivery does not necessarily deploy automatically to production. A release may require a reviewer, change window, or business approval.

Continuous Delivery answers:

> **Is this software ready to release?**

### Continuous Deployment

Continuous Deployment automatically releases a validated artifact when all required gates pass.

```text
Validated artifact
       |
       v
Automated deployment
       |
       v
Production
```

Continuous Deployment answers:

> **Can passing changes automatically reach production?**

### Interview trap

Do not describe CI/CD as simply:

> Every commit automatically deploys to production.

A stronger explanation distinguishes the three models:

```text
Continuous Integration
    Validate integrated changes.

Continuous Delivery
    Keep validated software ready to deploy.

Continuous Deployment
    Automatically release passing changes.
```

The correct model depends on the application's risk, testing confidence, approval process, and operational maturity.

---

## 4. Revision: GitHub Actions Structure

Remember the hierarchy:

```text
Workflow
    |
    v
Jobs
    |
    v
Steps
    |
    v
Commands or reusable actions
```

The runner executes the jobs:

```text
Runner
    |
    +-- Job 1
    +-- Job 2
    +-- Job 3
```

A minimal example is:

```yaml
name: MERN CI

on:
  pull_request:
    branches:
      - main
  push:
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

      - name: Run tests
        run: npm test
```

The mental model is:

```text
Workflow = complete automation definition
Job      = unit of work
Step     = individual operation
Runner   = execution environment
```

A workflow can contain multiple jobs. Jobs may run in parallel or may be linked with `needs`.

---

## 5. Revision: MERN CI Pipeline

For a MERN backend, the validation flow is:

```text
Git push or pull request
          |
          v
Checkout source
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
```

The Docker stage should happen after the required validation gates:

```text
Validation passes
        |
        v
Docker build
        |
        v
mern-api:<git-sha>
```

This creates a traceable artifact. If the image tag is `mern-api:abc123`, the team can identify the source commit that produced it.

A production-quality pipeline should not publish an image merely because the Dockerfile is syntactically valid. The image should represent code that passed the project's required quality checks.

---

## 6. Revision: Why `npm ci`?

A Node.js project commonly contains:

```text
package.json
package-lock.json
```

CI should generally use:

```bash
npm ci
```

The lockfile records the resolved dependency tree:

```text
package.json + package-lock.json
              |
              v
            npm ci
              |
              v
      Reproducible dependencies
```

`npm ci` is designed for clean, automated installations. It uses the lockfile, avoids silently selecting a different dependency tree, and helps a fresh runner behave consistently.

Use `npm install` when intentionally changing dependencies or regenerating the lockfile. After changing dependencies, review and commit the resulting lockfile so CI has a known input.

Interview answer:

> `npm ci` is designed for clean, reproducible dependency installation in automated environments using the committed lockfile.

---

## 7. Revision: Cache Versus Artifact

### Cache

A cache is reusable data intended to make future runs faster.

```text
Cache
  |
  v
Reduce repeated work on a later run
```

Example:

```text
npm dependency download cache
```

A cache can expire or be unavailable. The pipeline must still work without it.

### Artifact

An artifact is a useful output preserved from a workflow run.

```text
Artifact
    |
    v
Preserve or transfer a result from this run
```

Examples:

```text
Coverage report
test-results.xml
Frontend build directory
Docker image metadata
```

Remember:

```text
Cache
    = performance optimization

Artifact
    = build output or useful deliverable
```

A Docker image in a registry is commonly the release artifact. A test report uploaded to GitHub is a workflow artifact that supports debugging and audit evidence.

---

## 8. Revision: Secrets and Security Boundaries

A pipeline may eventually need:

```text
Registry token
Cloud credentials
Deployment credentials
Database credentials
```

Never put real values directly in source code:

```yaml
# Do not do this
password: MyProductionPassword
```

Use a secret store:

```text
Encrypted secret store
          |
          v
Trusted CI job
          |
          v
Required command
```

Use least privilege:

- A lint job normally needs no production credentials.
- A pull request from a fork should not receive deployment secrets.
- An image publishing job should receive only the registry permissions it needs.
- Production deployment secrets should be scoped to a protected environment.
- Secrets must not be printed into logs or baked into Docker image layers.

A basic read-only workflow permission can be expressed as:

```yaml
permissions:
  contents: read
```

Publishing and deployment workflows require additional permissions only when their operations genuinely need them.

---

## 9. Revision: Job Dependencies and Quality Gates

Suppose a workflow contains:

```text
test
build
deploy
```

The safe relationship is:

```text
test
  |
  v
build
  |
  v
deploy
```

In GitHub Actions:

```yaml
jobs:
  test:
    ...

  build:
    needs: test
    ...

  deploy:
    needs: build
    ...
```

If the test job fails:

```text
test failed
    |
    v
build blocked
    |
    v
deploy blocked
```

This turns the pipeline into explicit quality gates instead of a list of unrelated commands.

Independent jobs can still run in parallel:

```text
          +-- Lint --------+
          |                |
          +-- Unit tests --+--> Docker build
          |                |
          +-- Security ----+
```

The Docker job should depend on every required upstream check. Parallelism improves speed, while `needs` protects the release path.

---

## 10. Revision: The Docker Artifact

The pipeline should produce a traceable image:

```text
mern-api:abc123
```

where:

```text
abc123 -> Git commit abc123
```

The release identity is then:

```text
Production
    |
    v
mern-api:abc123
    |
    v
Git commit abc123
    |
    v
Exact source revision
```

This is much stronger than:

```text
Production
    |
    v
latest
```

The `latest` tag is mutable and may point to a different image later. A commit SHA or release version makes rollback and incident investigation more reliable. The image digest should also be recorded when the image is published.

---

## 11. Practical Incident: Failed Tests

You are on call. A developer has pushed a change and GitHub Actions reports:

```text
Checkout       passed
npm ci         passed
Lint           passed
Unit tests     failed
Docker build   skipped
Deploy         skipped
```

The developer says:

> Can you deploy the Docker image anyway? It worked locally.

Do not deploy it. The pipeline has correctly stopped an artifact that did not pass the required quality gate.

Use this sequence:

```text
Test failure
     |
     v
Read the CI logs
     |
     v
Understand the root cause
     |
     v
Reproduce locally with the same runtime
     |
     v
Fix the change or test
     |
     v
Push again
     |
     v
Run CI again
```

A local success does not override a reproducible CI failure. CI may expose runtime, dependency, environment, service-readiness, filesystem, or test-isolation problems that are absent on the developer's machine.

The pipeline is doing exactly what it should: stopping an untrusted artifact before publication or deployment.

---

## 12. Troubleshooting: Local Success but CI Failure

Interview question:

> A developer says the application works locally, but CI is failing. What do you investigate?

A strong investigation includes:

```text
1. CI logs and the first meaningful error
2. Node.js and npm versions
3. Dependency installation and lockfile state
4. Environment variables and defaults
5. Database and service dependencies
6. Filesystem paths and case sensitivity
7. Timezone and locale assumptions
8. Test isolation and ordering
9. Network access and service readiness
10. Differences between local and CI runtime
```

The investigation should be evidence-based:

1. Identify the first failing step rather than only the final job status.
2. Compare the CI runtime with the local runtime.
3. Confirm that the lockfile and working tree are the same revision.
4. Check whether required services are running and ready.
5. Reproduce with the same Node version, environment, and container configuration.
6. Fix the root cause and rerun the same focused check.

A strong answer is:

> I would inspect the CI logs, compare Node and dependency versions, verify environment variables and service readiness, check filesystem and timezone assumptions, and reproduce the failure using the same runtime or container configuration instead of simply rerunning the pipeline.

---

## 13. Review Questions From Earlier Topics

### Docker networking

Why does this commonly fail inside the API container?

```text
mongodb://localhost:27017
```

**Answer:** Inside the API container, `localhost` refers to that API container. In Docker Compose, MongoDB should normally be reached through its service name, such as `mongodb://mongodb:27017`.

### Docker security

Why should secrets not be baked into Docker images?

**Answer:** Images are distributed artifacts. Secrets included during a build may remain in image layers or metadata and can be exposed to anyone who can access or inspect the image.

### Image versioning

Why tag images with the Git SHA?

**Answer:** It creates direct traceability between the deployed image and the source revision that produced it.

### Health checks

Why should CD validate application health after deployment?

**Answer:** A container can be running while the application is unhealthy, unable to connect to its dependencies, or not ready to receive traffic.

### Rollback

Why keep the previous known-good image?

**Answer:** It enables a fast, reproducible rollback without rebuilding the old release during an incident.

---

## 14. Weekly Quiz

### Q1

What is the primary purpose of CI?

A. Automatically delete servers  
B. Validate integrated code changes  
C. Back up MongoDB  
D. Configure DNS

### Q2

Continuous Delivery means:

A. Every commit automatically reaches production  
B. Software is kept in a deployable state  
C. Docker is mandatory  
D. Developers do not need tests

### Q3

A GitHub Actions runner is:

A. The Git repository  
B. The machine or environment executing workflow jobs  
C. A Docker registry  
D. A database

### Q4

Which command is generally preferred in CI when a Node lockfile exists?

A. `npm ci`  
B. `npm remove`  
C. `npm publish`  
D. `npm init`

### Q5

What is the purpose of caching?

A. Make repeated CI runs faster  
B. Store production secrets  
C. Replace testing  
D. Deploy MongoDB

### Q6

What is an artifact?

A. A useful output produced by the pipeline  
B. A Git password  
C. A Docker network  
D. A runner

### Q7

If the test job fails, what should happen to a dependent deployment job?

A. Deploy anyway  
B. It should be blocked  
C. Delete the repository  
D. Restart MongoDB

### Q8

Why use a Git SHA as a Docker image tag?

A. Traceability  
B. Encryption  
C. Database persistence  
D. Networking

### Q9

Where should CI/CD credentials normally be stored?

A. Source code  
B. Dockerfile  
C. CI/CD secret store or appropriate secret manager  
D. README

### Q10

Which is the best high-level CI pipeline?

A.

```text
Push -> Production
```

B.

```text
Push -> Test -> Build -> Scan -> Artifact
```

C.

```text
Push -> Delete old image
```

D.

```text
Push -> Restart MongoDB
```

### Answer Key

```text
1 -> B
2 -> B
3 -> B
4 -> A
5 -> A
6 -> A
7 -> B
8 -> A
9 -> C
10 -> B
```

### Score Guide

```text
9-10  Excellent
7-8   Good
5-6   Review CI/CD fundamentals
0-4   Revisit Days 41-42
```

---

## 15. Practical Assessment: Build the Pipeline

The goal is to produce:

```text
.github/
└── workflows/
    └── ci.yml
```

It should implement:

```text
Pull request or push to main
          |
          v
       Checkout
          |
          v
      Setup Node
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
      Docker build
          |
          v
     mern-api:<SHA>
```

### Requirements

```text
[ ] Controlled Node.js version
[ ] npm ci
[ ] Lint
[ ] Unit tests
[ ] Application build
[ ] Docker image build
[ ] Git SHA image tag
[ ] Failed tests block downstream work
```

### Bonus requirements

```text
[ ] npm dependency cache
[ ] Test result artifact
[ ] Coverage artifact
[ ] Separate validation and Docker jobs
[ ] Read-only workflow permissions
```

### Suggested job structure

```yaml
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
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
      - run: docker build -t "mern-api:${{ github.sha }}" ./backend
```

Adapt the working directory and commands to the actual repository layout. Do not copy a path that does not exist in your project.

---

## 16. Practical Failure Test

Create a deliberate test failure and push it to a branch or draft pull request.

Expected result:

```text
Checkout       passed
Install        passed
Lint           passed
Tests          failed
Build          blocked
Docker         blocked
```

Then fix the test and run CI again.

Expected result:

```text
Checkout       passed
Install        passed
Lint           passed
Tests          passed
Build          passed
Docker         passed
```

This exercise teaches one of the most important CI/CD principles:

> **A pipeline is not valuable because it always passes. It is valuable because it reliably stops bad changes.**

Record the failed job, failed step, error message, and skipped dependent jobs as evidence.

---

## 17. Monthly Cumulative Project - CI/CD Milestone

The cumulative MERN project now contains a real CI foundation.

### Current architecture

```text
Developer
    |
    v
Pull request
    |
    v
GitHub Actions
    |
    +-- Lint
    +-- Tests
    +-- Build
    +-- Docker build
             |
             v
       mern-api:<SHA>
```

### Next project milestone

Add:

```text
Docker image
      |
      v
Security scan
      |
      v
Registry
      |
      v
Staging
```

The longer-term architecture becomes:

```text
Git
 |
 v
CI
 +-- Test
 +-- Build
 +-- Scan
 +-- Package
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
Production
       |
       v
Monitoring
       |
       v
Rollback
```

The next milestone should publish only the same artifact that passed validation and scanning. Avoid rebuilding a different image for each environment.

---

## 18. Interview Practical

Answer this aloud:

> Design a CI/CD pipeline for a Dockerized MERN application.

Your answer should include:

```text
Git trigger
    |
    v
CI runner
    |
    v
Dependency installation
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
Security scan
    |
    v
Registry
    |
    v
Staging
    |
    v
Health validation
    |
    v
Production
    |
    v
Rollback
```

Then explain:

- Where secrets live.
- How images are versioned.
- How failed tests stop deployment.
- How health checks prove readiness.
- How rollback selects a known-good artifact.
- How the deployed image maps back to source code.

A strong answer demonstrates system design, not merely familiarity with GitHub Actions syntax.

---

## 19. Day 43 Takeaways

The most important concepts from this week's CI/CD lessons are:

```text
Continuous Integration
    = Validate changes

Continuous Delivery
    = Keep releases deployable

Continuous Deployment
    = Automatically release passing changes
```

And:

```text
Workflow
    |
    v
Job
    |
    v
Step
    |
    v
Runner
```

For the MERN project:

```text
Git
 |
 v
GitHub Actions
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
Docker
 |
 v
SHA-tagged image
```

### Interview takeaway

A strong DevOps engineer does not merely automate commands. They create quality gates that prevent bad software from becoming a production incident.

---

## 20. Next Lesson

Tomorrow the course resumes new material with Docker registry integration from GitHub Actions:

```text
CI
 |
 v
Docker build
 |
 v
Image scan
 |
 v
Registry authentication
 |
 v
Docker push
 |
 v
Versioned production artifact
```

This connects the CI pipeline from Days 41-43 to the Docker registry and deployment architecture already established in the project.
