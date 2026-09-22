# Day 45 - Assignment: Staging, Smoke Tests, and Promotion

## Objective

Build a clear understanding of how a validated Docker image moves from registry to staging and then toward production using smoke tests, health checks, and promotion gates.

This assignment focuses on environment separation, artifact promotion, and release safety.

---

## Scenario

Your MERN application is now building and publishing a Docker image from CI. The next step is to deploy that image to a staging environment and verify that it works before release.

The target flow is:

```text
Git
  |
  v
CI
  |
  v
Docker build + scan
  |
  v
Registry
  |
  v
Staging deployment
  |
  v
Health check + smoke tests
  |
  v
Promotion gate
  |
  v
Production
```

Your goal is not to skip validation; your goal is to validate the actual deployed artifact in a production-like setting.

---

## Part 1: Understand the Concepts

Write short answers to the following:

1. Why do we need a staging environment if CI already runs tests?
2. What is the difference between development, staging, and production?
3. Why should the same Docker image be deployed to both staging and production?
4. What is promotion in a deployment pipeline?
5. What is a smoke test?
6. Why are staging secrets different from production secrets?
7. Why is environment isolation important?
8. What should happen if a staging health check fails?

Keep your answers practical and real-world, not theoretical only.

---

## Part 2: Draw the Release Flow

Create a simple architecture diagram for your MERN deployment process.

Your diagram should include:

```text
Git
CI
Registry
Staging
Smoke tests
Promotion gate
Production
```

Then label where the build artifact is created and where the runtime configuration changes.

Explain in 4-5 lines why the artifact should remain the same while configuration changes between environments.

---

## Part 3: Define the Smoke Test

A good smoke test should answer the question:

> Is the deployed service ready enough to keep moving?

For your MERN app, identify at least 3 smoke checks.

Example checks:

```text
GET /health
GET /api/health
GET /api/products
```

Record:

- the endpoint
- expected HTTP status
- expected response type
- what it proves

If your application has a different health route, use the actual one from your project.

---

## Part 4: Review the Configuration Model

List the configuration values that should be different between staging and production.

Example:

```text
NODE_ENV
MONGO_URI
PORT
STRIPE_KEY
API_BASE_URL
```

For each item, explain:

- whether it should be environment-specific
- whether it is a secret or non-secret
- which environment should own it

Then explain why you must not hard-code production values into the Docker image.

---

## Part 5: Diagnose a Failure

Use this scenario:

```text
Staging deploy succeeds
Nginx is up
API container is running
curl /api/health returns 502
```

Write a troubleshooting checklist for this issue.

Your checklist should include:

1. Check container status
2. Check Docker logs
3. Check Nginx upstream or reverse proxy config
4. Check API port binding
5. Check MongoDB connectivity
6. Check environment variables such as MONGO_URI
7. Confirm service-to-service address names inside Docker

Then explain the likely root cause if the API is configured with:

```text
mongodb://localhost:27017/mern
```

instead of:

```text
mongodb://mongodb:27017/mern
```

---

## Part 6: Promotion Decision

Write a policy for when promotion to production is allowed.

Use this template:

```text
Promotion is allowed only if:
- CI passed
- image scan passed
- staging deployment succeeded
- health checks passed
- smoke tests passed
- required approval is complete
```

If any gate fails, the release should be blocked and investigated.

Explain why a failed staging smoke test is safer than a failed production release.

---

## Part 7: Apply It to Your MERN Project

Document the staging release pipeline for your MERN project.

Add a short document or section with:

```text
Staging URL:
Image reference:
Tag strategy:
Database configuration:
Environment variables:
Health endpoint:
Smoke tests:
Rollback plan:
Promotion criteria:
```

If your project is not yet deployed to staging, write the intended configuration and explain how you would deploy it.

---

## Part 8: Interview Response Practice

Prepare a concise but strong answer to this question:

> Explain how you would safely promote a Dockerized MERN application from staging to production.

Your answer should include:

- CI validation
- image publication
- staging deployment
- smoke tests and health checks
- same-image promotion
- production-specific configuration
- rollback strategy

Keep it under 150 words if possible, but make sure it is complete.

---

## Deliverables

Submit or create:

- a diagram of the staging-to-production flow
- a smoke test list for your project
- a short troubleshooting write-up
- a promotion policy
- a 1-paragraph interview answer

The purpose is to show that you understand not only CI but also real deployment safety.

---

## Bonus Reflection

Answer this question in 3-5 sentences:

> Why does promoting the same validated image make rollback easier and improve confidence?

---

## Evaluation Criteria

Your work should show:

- understanding of environment separation
- correct use of staging and production concepts
- recognition of smoke tests as deployment gates
- awareness of configuration and secret differences
- understanding of deployment vs promotion
- practical failure-handling logic

A strong answer should read like an engineering decision, not just a list of definitions.
