# DevOps Mentorship Program - Day 41

## Phase 3: CI/CD

### CI/CD Fundamentals: From Git Commit to Production

**Level:** Intermediate -> Professional   
**Focus:** Continuous Integration, Continuous Delivery, Continuous Deployment, pipeline gates, artifacts, and a production-style MERN pipeline

Day 41 begins the CI/CD phase. Docker taught us how to package and run the application. CI/CD teaches us how to validate, package, promote, and release it repeatedly and safely.

The delivery journey is:

```text
MERN source code
      |
      v
Git commit and push
      |
      v
CI runner
      |
      +-- Install dependencies
      +-- Lint
      +-- Unit tests
      +-- Integration tests
      +-- Build
      +-- Docker build
      +-- Security scan
      |
      v
Versioned artifact
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
Monitoring and rollback
```

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain Continuous Integration, Continuous Delivery, and Continuous Deployment.
- Describe a CI/CD pipeline from Git push to production.
- Explain why CI/CD improves feedback, repeatability, and release safety.
- Design a basic CI pipeline for a MERN backend.
- Explain pipeline stages, gates, jobs, runners, and artifacts.
- Explain why tests should run before Docker image publication or deployment.
- Tag a Docker image with the Git commit SHA for traceability.
- Store credentials in a secret store instead of source code.
- Explain the difference between unit, integration, and end-to-end tests.
- Answer common CI/CD interview questions clearly.

---

## 2. What Problem Does CI/CD Solve?

Without automation, a release may depend on a developer remembering a long sequence of manual steps:

```text
Developer
   |
   +-- Run a few local tests
   +-- Build a Docker image
   +-- SSH to a server
   +-- Copy or pull files
   +-- Restart services
   +-- Hope the deployment works
```

This process is difficult to repeat, difficult to audit, and easy to perform differently each time.

With CI/CD:

```text
Git push
   |
   v
Checkout
   |
   v
Install -> Lint -> Test -> Build -> Scan
   |
   v
Versioned artifact
   |
   v
Staging -> Health check -> Production
```

The goal is to make delivery:

- **Repeatable:** the same pipeline performs the same steps.
- **Automated:** machines perform routine validation and packaging.
- **Observable:** logs and reports explain what happened.
- **Traceable:** a release points back to an exact commit and artifact.
- **Safer:** failed gates prevent bad changes from moving forward.

CI/CD does not mean that every change must deploy directly to production. It means that the path from change to release is controlled and increasingly automated.

---

## 3. Continuous Integration

**Continuous Integration (CI)** means that developers frequently integrate changes into a shared codebase and automated systems validate those changes.

A basic CI pipeline is:

```text
Git push
   |
   v
Checkout source
   |
   v
Install from lockfile
   |
   v
Lint
   |
   v
Unit tests
   |
   v
Integration tests
   |
   v
Build application
```

If a required step fails, later steps should normally stop:

```text
Tests fail
    |
    v
Pipeline fails
    |
    +-- No image publication
    +-- No staging deployment
    +-- No production deployment
```

CI gives developers fast feedback while the change is still fresh in their minds.

---

## 4. Continuous Delivery and Continuous Deployment

**Continuous Delivery** extends CI by keeping validated software in a deployable state. The artifact is built, tested, and ready to release, but production deployment may still require a manual approval.

```text
Code -> Test -> Build -> Package -> Ready to release
```

**Continuous Deployment** goes one step further. When all automated gates pass, the system automatically deploys the change to production.

```text
Code -> Test -> Build -> Scan -> Deploy -> Production
```

The interview distinction is:

```text
Continuous Integration
    Validate integrated changes.

Continuous Delivery
    Keep validated software ready to deploy.

Continuous Deployment
    Automatically deploy passing changes to production.
```

Delivery and deployment are related, but they are not synonyms.

---

## 5. The MERN CI/CD Pipeline

For a MERN project, a production-style pipeline can look like this:

```text
Developer
    |
    v
Git push or pull request
    |
    v
CI runner
    |
    +-- Checkout
    +-- npm ci
    +-- npm run lint
    +-- npm test
    +-- Integration tests
    +-- npm run build
    +-- docker build
    +-- Image scan
    |
    v
mern-api:<git-sha>
    |
    v
Container registry
    |
    v
Staging deployment
    |
    v
GET /health and smoke tests
    |
    v
Production deployment
    |
    v
Monitoring and rollback
```

Every important stage is a gate. A failed lint, test, build, or security check should prevent the artifact from being promoted according to the project's release policy.

---

## 6. Pipeline Stages and Gates

A pipeline stage is a meaningful section of work. A gate is a condition that must pass before the pipeline continues.

```text
Checkout
   |
   v
Install dependencies
   |
   v
Lint -------- FAIL -> STOP
   |
   v
Unit tests --- FAIL -> STOP
   |
   v
Integration tests
   |
   v
Build -------- FAIL -> STOP
   |
   v
Docker build
   |
   v
Security scan - FAIL -> STOP
   |
   v
Publish artifact
   |
   v
Deploy
```

This is safer than sending every Git push directly to production because each transition has an explicit quality check.

Common gates include:

- Lint and formatting checks.
- Unit and integration test results.
- Build success.
- Dependency or image vulnerability thresholds.
- Required code review approvals.
- Branch protection checks.
- Manual approval for production.
- Health and smoke checks after deployment.

---

## 7. Git Is the Trigger

A pipeline commonly begins with a push or pull request:

```bash
git add .
git commit -m "Add order endpoint"
git push origin feature/order-endpoint
```

The CI platform receives the repository event and starts a workflow. A typical branch flow is:

```text
feature branch
      |
      v
Pull request
      |
      v
CI checks and review
      |
      v
Protected main branch
      |
      v
Release pipeline
```

Branch protection can require passing checks, code review, and a clean merge before changes reach the release branch.

Pull requests from untrusted forks deserve additional care. They should receive limited permissions and should not automatically receive production credentials.

---

## 8. Docker Images as Artifacts

An artifact is a build output that can be consumed by a later process. Examples include:

```text
Docker image
Frontend build directory
npm package
Test report
Security report
```

For this project, the primary deployment artifact can be:

```text
mern-api:<commit-sha>
```

CI creates the artifact, a registry stores it, and CD deploys the same artifact. Building again in each environment can produce a different result, so the preferred model is:

```text
Build once
   |
   v
Scan and test
   |
   v
Store artifact
   |
   v
Promote the exact artifact
```

### Why use the Git SHA?

If the commit is `abc123def`, CI can create:

```text
mern-api:abc123def
```

This makes it possible to answer:

- What source code is running?
- Which pipeline produced the image?
- Which artifact should be rolled back?
- Which commit introduced a problem?

A mutable tag such as `latest` is convenient for experiments but is not enough for reliable production traceability.

---

## 9. Test Pyramid in CI

A MERN pipeline should use tests at different levels:

```text
              E2E tests
             /         \
            /           \
       Integration tests
          /             \
         /               \
             Unit tests
```

Unit tests are usually numerous and fast. Integration tests validate boundaries such as the API and MongoDB. End-to-end tests validate a complete user path through the browser, frontend, API, and database.

Example for `POST /api/orders`:

```text
Unit test:
    calculateOrderTotal()

Integration test:
    API -> MongoDB

End-to-end test:
    Browser -> Nginx -> React -> API -> MongoDB
```

The lower tests provide quick feedback. The higher tests provide broader confidence but generally cost more time and are more sensitive to environment problems.

---

## 10. Secrets and Permissions

Pipelines may need registry credentials, cloud credentials, deployment keys, or API keys. These values must not be committed to the repository, placed in a Dockerfile, or printed in logs.

Use a CI platform secret store or an external secret manager:

```text
Secret store
     |
     v
Job with minimum required permissions
     |
     v
Temporary credential use
```

Good practices include:

- Give each job only the permissions it needs.
- Expose deployment credentials only to trusted branches or environments.
- Do not pass production secrets to untrusted pull request code.
- Mask secrets in logs.
- Rotate credentials when exposure is suspected.
- Prefer short-lived identity or federation where supported.

---

## 11. Failed Pipeline Example

Suppose a developer pushes `feat/order-discounts` and the tests report:

```text
Expected: 100
Received: 110
```

The correct result is:

```text
Checkout       PASS
Install        PASS
Lint           PASS
Unit tests     FAIL
Build          SKIPPED
Docker build   SKIPPED
Scan           SKIPPED
Deploy         SKIPPED
```

This is a successful safety behavior. The pipeline detected the defect and prevented an unverified artifact from moving forward.

---

## 12. When Tests Pass Locally but Fail in CI

Investigate differences instead of simply rerunning the pipeline:

```text
1. Runtime and operating-system versions
2. Node.js version
3. Lockfile and npm installation behavior
4. Environment variables and missing secrets
5. Database or service dependencies
6. Filesystem and path assumptions
7. Timezone and locale
8. Test ordering and shared state
9. Network access and service readiness
10. CI logs and the exact failing command
```

A useful technique is to reproduce the CI runtime with the same container image or tool versions used by the runner.

---

## 13. Health Validation

A pipeline should not stop at `docker run` or a successful process start. It should validate that the application responds correctly.

For an API exposing `GET /health`:

```text
Start application
      |
      v
GET /health
      |
      v
HTTP 200 and expected response
      |
      v
Health gate passes
```

A liveness check answers whether the process is alive. A readiness check answers whether the instance is prepared to receive traffic, including required dependencies where appropriate.

```json
{
  "status": "ok"
}
```

Health checks connect Day 41 to earlier lessons on deployment, service dependencies, and rollback.

---

## 14. CI/CD and Earlier Docker Lessons

The concepts now connect into one delivery system:

```text
Day 34: Health checks
Day 38: Registry and image lifecycle
Day 39: Image security and vulnerability management
Day 40: Production readiness and rollback
Day 41: CI/CD automation
```

The resulting architecture is:

```text
Code
  |
  v
Git
  |
  v
CI: lint, test, build, scan
  |
  v
Versioned Docker image
  |
  v
Registry
  |
  v
Staging and health validation
  |
  v
Production
  |
  v
Monitoring and rollback
```

The central idea is:

> CI/CD turns a collection of manual DevOps commands into a repeatable software delivery system.

---

## 15. Day 41 Summary

Remember these points:

- CI validates integrated code changes.
- Continuous Delivery keeps software deployable.
- Continuous Deployment automatically releases passing changes.
- Tests and quality gates should run before image publication and deployment.
- A Docker image is a deployable artifact when it is versioned, tested, scanned, and traceable.
- Tagging images with a Git SHA connects production behavior to source code.
- Secrets belong in a secret-management system, not in Git or image layers.
- The same tested artifact should be promoted between environments.
- Health checks prove more than process startup.
- A failed pipeline should stop the appropriate downstream stages.

### Next lesson

Day 42 goes deeper into GitHub Actions-style workflow design: jobs, steps, runners, caching, artifacts, environment variables, and secrets.
