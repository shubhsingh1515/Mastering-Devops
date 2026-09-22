# DevOps Mentorship Program - Day 45

## Phase 3: CI/CD

### CI/CD Environments: Staging, Smoke Tests & Promotion

**Level:** Intermediate -> Professional  
**Focus:** Environment separation, staging deployment, smoke tests, promotion gates, and safe MERN production releases

Yesterday, Day 44, your pipeline learned how to produce and publish a versioned Docker artifact from CI.

The flow looked like this:

```text
Git
  |
  v
CI
  |
  v
Test
  |
  v
Docker Build
  |
  v
Security Scan
  |
  v
Registry
  |
  v
mern-api:<git-sha>
```

Today we answer the next production question:

> How do we safely take that exact image from the registry and promote it toward production?

The flow becomes:

```text
Registry
   |
   v
Staging
   |
   v
Smoke Tests
   |
   v
Promotion Gate
   |
   v
Production
```

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain why staging exists.
- Distinguish development, staging, and production.
- Explain why the same artifact should move between environments.
- Understand deployment vs promotion.
- Design basic smoke tests.
- Explain environment-specific configuration.
- Understand deployment gates and approvals.
- Troubleshoot a failed staging deployment.
- Explain staging and promotion in DevOps interviews.
- Apply the workflow to your MERN project.

---

## 2. Why Do We Need Environments?

Imagine you deploy every Git commit directly to production:

```text
Developer
   |
   v
Git push
   |
   v
Production
```

A configuration mistake, database issue, missing secret, or a broken API can immediately affect real users.

Instead, a safer release flow is:

```text
Developer
   |
   v
CI
   |
   v
Staging
   |
   v
Validation
   |
   v
Production
```

Staging acts as a production-like environment where you test the real deployable artifact before exposing it to live users.

This separation is important because CI is not a real deployment proof. A build can pass, but the runtime environment may still fail due to:

- incorrect environment variables
- database connection issues
- missing network access
- container port mapping problems
- configuration drift
- secret mismatches
- infrastructure policy differences

Staging catches these problems before they hit production.

---

## 3. The Three Core Environments

A common model is:

```text
Development
    |
    v
Staging
    |
    v
Production
```

### Development

Optimized for:

- fast iteration
- debugging
- local testing
- feature work and experimentation

This is where developers build and debug the code quickly.

### Staging

Optimized for:

- production-like validation
- integration testing
- smoke testing
- release verification

This environment should resemble production closely enough to catch deployment problems.

### Production

Optimized for:

- real users
- availability
- reliability
- security
- observability

This is where the application is made available to end users and business-critical traffic.

The environments should differ where necessary, but the application artifact should remain consistent.

---

## 4. Same Artifact, Different Configuration

This is one of the most important ideas in modern delivery.

Suppose CI builds a single image:

```text
registry.example.com/mern-api:8f31c2a
```

Then:

- Staging deploys that same image with staging configuration.
- Production deploys that same image with production configuration.

Not:

```text
mern-api:8f31c2a-staging
```

followed by:

```text
mern-api:8f31c2a-production
```

and definitely not a rebuild.

Think of it like this:

```text
                 SAME IMAGE
                     |
           ┌─────────┴─────────┐
           ▼                   ▼
       Staging             Production
           │                   │
    staging config      production config
```

The artifact stays the same. The runtime environment configuration changes.

This is the practical meaning of:

> Build once, deploy many.

It improves:

- traceability
- release confidence
- rollback simplicity
- environment consistency

---

## 5. MERN Example

Suppose the image contains:

```text
mern-api:8f31c2a
```

In staging:

```text
NODE_ENV=staging
MONGO_URI=mongodb://staging-mongodb:27017/mern
```

In production:

```text
NODE_ENV=production
MONGO_URI=mongodb://production-mongodb:27017/mern
```

The application image is identical. Only the runtime configuration differs.

This is a key principle:

- the code and image are fixed
- the environment variables and infrastructure are environment-specific

The same Docker image is not a problem as long as runtime configuration is handled correctly.

---

## 6. What Is Promotion?

Promotion means moving an already-built artifact from one environment to another.

For example:

```text
Image
  |
  v
Staging
  |
  v
Validation
  |
  v
Production
```

The image is not rebuilt.

Conceptually:

```text
Registry
   |
   +--> Staging
   |      |
   |      v
   |   Smoke tests
   |      |
   |      v
   +--> Production
```

The production deployment consumes the exact artifact that passed staging.

This matters because it preserves the exact version that was tested.

---

## 7. What Is a Smoke Test?

A smoke test is a quick validation that checks whether the deployment is fundamentally working.

The goal is not to test every business flow.

The goal is to answer:

> Is this deployment alive enough to continue?

For a MERN application, a smoke test might check:

- `GET /`
- `GET /health`
- `GET /api/health`

These are lightweight checks that confirm the app is responding and dependencies are functioning at a basic level.

---

## 8. MERN Smoke Test Example

Suppose your API exposes:

```text
GET /health
```

Expected output:

```http
HTTP/1.1 200 OK
```

and body:

```json
{
  "status": "ok"
}
```

After staging deployment, you run:

```bash
curl https://staging.example.com/api/health
```

Expected response:

```text
200
```

If you receive:

```text
502
```

then promotion should stop.

That is exactly the purpose of a smoke test: to catch a failed deployment early before exposing it to production.

---

## 9. Smoke Tests vs Full Tests

Do not confuse the two.

### CI tests

These run before artifact publication:

- unit tests
- integration tests
- linting
- build validation

These answer: "Did the code pass the quality gates?"

### Smoke tests

These run after deployment:

- is the service responding?
- is the health endpoint healthy?
- does the deployed application start correctly?

These answer: "Does the deployed artifact actually work in the target environment?"

### End-to-end tests

These validate broader user journeys:

```text
Browser
  |
  v
Frontend
  |
  v
Nginx
  |
  v
API
  |
  v
MongoDB
```

A production pipeline may eventually use all three layers:

1. CI validation
2. deployment smoke tests
3. broad end-to-end testing

---

## 10. Deployment Pipeline Evolution

The pipeline now looks like this:

```text
Git
  |
  v
CI
  |
  +-- Lint
  +-- Unit Tests
  +-- Integration Tests
  +-- Build
  +-- Security Scan
  |
  v
Docker Image
  |
  v
Registry
  |
  v
Staging Deployment
  |
  v
Smoke Tests
  |
  v
Promotion Gate
  |
  v
Production
```

This is becoming a proper CD pipeline.

It is no longer just a validation workflow. It is an automated release mechanism that moves the validated artifact through environments.

---

## 11. Promotion Gate

A promotion gate is a quality barrier that decides whether the artifact can continue.

Example:

```text
Staging deployed ✓
Health check ✓
Smoke tests ✓
Then promotion is allowed.
```

But if:

```text
Staging deployed ✓
Health check ❌
```

then:

```text
Promotion blocked
```

This is a quality gate.

A gate can include:

- health checks
- smoke tests
- vulnerability thresholds
- manual approval
- database schema validation
- environment readiness checks

---

## 12. Manual Approval

Some organizations require a human to review before production:

```text
Staging
  |
  v
Automated validation
  |
  v
Manual approval
  |
  v
Production
```

Others rely entirely on automation:

```text
Staging
  |
  v
Automated validation
  |
  v
Production
```

The correct choice depends on:

- risk
- compliance requirements
- team maturity
- release frequency
- business requirements

Interviewers generally want you to show that you understand the trade-off.

A strong answer is not: "Always require manual approval." The correct answer is:

> It depends on the risk, compliance, and release process.

---

## 13. Environment-Specific Secrets

Staging and production should not share the same secrets unnecessarily.

For example:

```text
Staging:
  MONGO_URI -> staging database

Production:
  MONGO_URI -> production database
```

Likewise:

```text
Staging:
  STRIPE_KEY -> test account

Production:
  STRIPE_KEY -> live account
```

The important principle is:

```text
Environment
   |
   v
Configuration
   |
   v
Secrets
```

not:

```text
Docker image
   |
   v
All environment secrets
```

A Docker image should not embed real production secrets. Secrets should be injected at runtime from controlled secret stores or deployment platforms.

---

## 14. Environment Isolation

Your production database should not accidentally be used by staging.

Bad:

```text
Staging -> Production MongoDB
```

Better:

```text
Staging -> Staging MongoDB

Production -> Production MongoDB
```

This prevents staging tests from modifying real user data.

Isolation also reduces the blast radius of mistakes or malicious actions in non-production environments.

---

## 15. A Realistic Staging Failure

Suppose the deployment succeeds:

- Docker pull ✓
- container starts ✓
- Nginx starts ✓

But the smoke test fails:

```bash
curl https://staging.example.com/api/health
```

returns:

```text
502 Bad Gateway
```

What should you do?

Do not immediately redeploy blindly.

Instead, investigate:

```text
Nginx logs
   |
   v
API logs
   |
   v
Container status
   |
   v
Network
   |
   v
API port
   |
   v
Health endpoint
```

This is directly connected to your Docker troubleshooting knowledge.

---

## 16. Staging Troubleshooting Example

Imagine the issue is:

```text
Nginx: upstream connection refused
```

Then you inspect:

```bash
docker compose ps
```

You might discover:

```text
API: Restarting
```

Then:

```bash
docker compose logs api
```

and find:

```text
MongoServerSelectionError
```

Now inspect the configuration:

```text
MONGO_URI=mongodb://localhost:27017/mern
```

This is wrong because:

```text
localhost
```

inside the API container refers to the API container itself, not the MongoDB service.

The correct service name is usually:

```text
mongodb://mongodb:27017/mern
```

This is why staging is valuable: it catches deployment and configuration mistakes before production is affected.

---

## 17. Promotion vs Deployment

These terms are related but different.

### Deployment

Deployment means putting an artifact into an environment.

```text
Image
  |
  v
Staging
```

### Promotion

Promotion means allowing that already validated artifact to advance.

```text
Staging
  |
  v
Production
```

A useful mental model is:

- Deployment = place it
- Promotion = advance it

This distinction matters in interviews and release procedures.

---

## 18. Release Candidate

You might call the image:

```text
mern-api:8f31c2a
```

a release candidate after it has passed CI and is being validated in staging.

Then the flow becomes:

```text
CI
  |
  v
Artifact
  |
  v
Staging
  |
  v
Validation
  |
  v
Release
```

The important point is that the artifact identity remains stable.

The image tag or digest should remain the same from staging to production promotion unless there is an explicit versioning event.

---

## 19. Interview Preparation

### Q1. Why do you need staging if CI already runs tests?

A strong answer:

> CI validates code and build behavior in a controlled environment, while staging validates the actual deployable artifact and its interaction with deployment configuration, networking, infrastructure, and dependent services in a production-like environment.

This distinction is critical.

### Q2. Should you rebuild the image for production?

No. Prefer promoting the exact image that passed staging so production receives the same artifact that was validated.

### Q3. What is a smoke test?

A smoke test is a lightweight post-deployment test that verifies the service is fundamentally operational before release continues.

### Q4. What should happen if staging smoke tests fail?

The promotion should stop, and the failure should be investigated before production deployment.

### Q5. Why separate staging and production secrets?

To isolate environments and prevent test workloads or compromised staging systems from accessing production resources.

---

## 20. Senior Interview Scenario

> The staging deployment is healthy, but production fails immediately after promotion. What could be different?

A strong answer would include:

- production configuration
- production secrets
- network rules
- database endpoint
- DNS
- TLS
- resource limits
- IAM permissions
- external services
- database schema
- environment variables

Notice that the artifact does not have to be different for the environments to behave differently.

That is exactly why environment configuration must be controlled carefully.

---

## 21. Review Questions From Earlier Topics

### Q1 — Registry

Why should production pull the image built by CI rather than rebuild it?

Answer:

> To guarantee production uses the exact artifact that passed the earlier validation stages.

### Q2 — Docker Networking

Inside Docker, why is this usually wrong for API → MongoDB?

```text
mongodb://localhost:27017
```

Answer:

> `localhost` refers to the API container itself; the MongoDB service should normally be addressed by its service name.

### Q3 — Security

Why shouldn't staging use production database credentials?

Answer:

> Environment isolation reduces the risk of accidental or malicious access to real production data.

### Q4 — Health Checks

What's the difference between a container being Up and an application being ready?

Answer:

> Up indicates that the container or process is running. Readiness requires the application to be capable of serving traffic correctly.

### Q5 — Rollback

Why does promoting the same image make rollback easier?

Answer:

> You can redeploy a previously validated artifact without rebuilding it.

---

## 22. Today's Quiz

### Q1

Why does staging exist?

A. To store Git repositories  
B. To validate deployments in a production-like environment  
C. To replace CI  
D. To store Docker credentials

### Q2

Should production normally use a rebuilt image?

A. Yes  
B. No, promote the validated artifact

### Q3

What is a smoke test?

A. Full performance testing  
B. A lightweight test verifying basic deployment functionality  
C. A Docker build  
D. A database backup

### Q4

If staging smoke tests fail, what should happen?

A. Deploy to production anyway  
B. Stop promotion and investigate  
C. Delete the registry  
D. Disable health checks

### Q5

What should differ between staging and production?

A. Necessarily the application image  
B. Environment-specific configuration and infrastructure  
C. Source code  
D. Dockerfile contents for every release

### Q6

Why isolate staging databases from production databases?

A. To prevent test workloads from affecting real production data  
B. To make Docker slower  
C. To eliminate backups  
D. To disable MongoDB

### Q7

Deployment means:

A. Putting an artifact into an environment  
B. Writing source code  
C. Deleting an image  
D. Running Git

### Q8

Promotion means:

A. Advancing a validated artifact toward another environment  
B. Rebuilding source code  
C. Changing MongoDB schemas  
D. Restarting Nginx

### Q9

A staging API returns 502. What should you investigate first?

A. Nginx/upstream connectivity and API health  
B. Delete production  
C. Change React CSS  
D. Rebuild MongoDB

### Q10

Why is the same Docker image useful across staging and production?

A. It improves reproducibility and confidence that production uses the validated artifact  
B. It removes configuration  
C. It eliminates testing  
D. It forces identical databases

### Answer Key

```text
1 -> B
2 -> B
3 -> B
4 -> B
5 -> B
6 -> A
7 -> A
8 -> A
9 -> A
10 -> A
```

### Score

```text
9-10  Excellent
7-8   Good
5-6   Review staging/promotion
<5    Revisit today's lesson
```

---

## 23. Practical Exercise — Deploy to Staging

Extend your current MERN pipeline.

Target flow:

```text
Git
  |
  v
CI
  |
  v
Docker image
  |
  v
Registry
  |
  v
Staging
```

Deploy:

```text
registry.example.com/mern-api:${GITHUB_SHA}
```

to your staging environment.

Then verify:

```bash
curl https://staging.example.com/api/health
```

Expected:

```text
HTTP 200
```

Then test one real API endpoint, for example:

```bash
curl https://staging.example.com/api/products
```

You are not trying to perform full end-to-end testing yet. You are establishing the promotion gate.

---

## 24. Practical Failure Drill

Simulate a deployment problem.

Change the staging environment's API configuration to:

```text
MONGO_URI=mongodb://localhost:27017/mern
```

Then deploy.

Expected:

```text
API health fails
```

Now troubleshoot:

```text
Nginx
  |
  v
API
  |
  v
MongoDB
  |
  v
Configuration
```

Correct it to:

```text
MONGO_URI=mongodb://mongodb:27017/mern
```

Redeploy or restart the affected service.

Then confirm:

```text
Health ✓
Smoke test ✓
Promotion allowed
```

This is an excellent interview scenario because it combines:

- Docker
- networking
- configuration
- CI/CD
- troubleshooting

---

## 25. Monthly Cumulative Project — Staging Milestone

Your MERN project should now support:

```text
Git
  |
  v
CI
  |
  +-- Lint
  +-- Tests
  +-- Build
  +-- Docker Build
  +-- Security Scan
  |
  v
Registry
  |
  v
Staging
  |
  v
Health Check
  |
  v
Smoke Tests
```

Add to your deployment documentation:

```text
STAGING.md
```

Document:

- [ ] Staging URL
- [ ] Image reference
- [ ] Environment configuration
- [ ] Database configuration
- [ ] Health endpoint
- [ ] Smoke tests
- [ ] Failure procedure
- [ ] Promotion criteria

Promotion should require:

```text
CI ✓
Image scan ✓
Deployment ✓
Health ✓
Smoke tests ✓
```

Then:

```text
PROMOTE
   |
   v
Production
```

---

## 26. Interview Challenge

Answer this aloud:

> Explain how you would safely promote a Dockerized MERN application from staging to production.

A strong answer:

> CI builds and validates the Docker image, scans it, and publishes it to a registry using a traceable immutable or tightly versioned reference. Staging deploys that exact image with staging-specific configuration. After deployment, I run health checks and smoke tests against the actual deployed service. If the gates pass, the same image is promoted to production with production-specific configuration and secrets. If production health checks fail, I use the previous known-good artifact for rollback.

That is the complete production story.

---

## 27. Day 45 Summary

Today's most important concept:

> Staging validates the deployable artifact in a production-like environment before promotion to production.

Your architecture is now:

```text
                  GIT
                   |
                   v
                  CI
                   |
        ┌──────────┼──────────┐
        ▼          ▼          ▼
      Test       Build      Scan
        │          │          │
        └──────────┼──────────┘
                   ▼
                Registry
                   │
                   ▼
                Staging
                   │
            ┌──────┴──────┐
            ▼             ▼
         Health        Smoke Tests
            │             │
            └──────┬──────┘
                   ▼
              Promotion
                   │
                   ▼
              Production
```

### Interview takeaway

- CI asks: "Is this change good?"
- Staging asks: "Does this deployable artifact work in an environment like production?"
- Promotion asks: "Have we gathered enough evidence to release this artifact?"

---

## 28. Next Lesson Preview

Day 46 will continue with production deployment strategies, including:

- rolling deployment
- blue-green deployment
- canary deployment
- zero-downtime releases
- rollback strategies

We will apply these to the MERN application and focus heavily on DevOps interview scenarios and trade-offs.

---

## 29. Final Revision Checklist

Before moving to the next lesson, make sure you can explain:

- [ ] Why staging exists
- [ ] What deployment and promotion mean
- [ ] Why the same artifact should be reused
- [ ] What a smoke test is
- [ ] Why environment-specific secrets matter
- [ ] How to troubleshoot a failed staging health check
- [ ] Why promotion gates stop bad releases
- [ ] How this applies to a MERN pipeline

If you can explain all of those clearly, your understanding of CI/CD production flow is becoming strong.
