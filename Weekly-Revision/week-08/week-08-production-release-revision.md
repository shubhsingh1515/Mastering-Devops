# DevOps Mentorship Program - Week 8 Sunday Revision

## Phase 3: CI/CD -> Production Operations

### Day 50 - Weekly DevOps Revision, Quiz and Practical Assessment

**Type:** Sunday revision, quiz and practical assessment  
**Level:** Intermediate -> Professional  
**Duration:** 25-30 minutes  
**Focus:** Docker artifacts, registry publishing, staging promotion, deployment strategies, rollback, feature flags, observability and production incident response

Today is a revision day. There is no new isolated topic. The goal is to connect Days 44-49 into one production-grade MERN delivery system.

This week you progressed from publishing a Docker image to safely operating a release in production:

```text
Day 44 -> Docker registry and image publishing
Day 45 -> Staging, smoke tests and promotion
Day 46 -> Rolling, blue-green and canary deployments
Day 47 -> Rollback, feature flags and release safety
Day 48 -> CI/CD observability
Day 49 -> Production logging
Day 50 -> Weekly revision and assessment
```

---

## 1. Learning Objectives

By the end of this revision, you should be able to:

- Explain the complete CI/CD flow for a production MERN application.
- Build once and promote the same Docker artifact through environments.
- Explain why registry tags should identify an immutable release.
- Define the purpose of staging, health checks and smoke tests.
- Compare rolling, blue-green and canary deployment strategies.
- Choose between rollback and a feature-flag disable.
- Connect metrics, logs and traces during an incident.
- Explain why normal CPU or memory does not prove application health.
- Investigate a failed canary deployment using business and technical signals.
- Create a release runbook that another engineer can execute.
- Answer end-to-end production release questions in an interview.

---

## 2. Week-at-a-Glance

The production pipeline now looks like this:

```text
Developer
    |
    v
Git
    |
    v
GitHub Actions
    |
    +-- npm ci
    +-- Lint
    +-- Tests
    +-- Build
    +-- Docker build
    +-- Security scan
           |
           v
        Registry
           |
           v
        Staging
           |
      +----+----+
      |         |
      v         v
   Health     Smoke
    check      tests
      |         |
      +----+----+
           |
           v
       Promotion
           |
           v
       Production
           |
      +----+----------------+
      |                     |
      v                     v
Deployment              Observability
strategy                    |
      |                +----+----+
      v                v    v    v
Rolling /            Logs Metrics Traces
Blue-green /             |
Canary                   v
                       Alerts
                          |
                   +------+------+
                   v             v
                Healthy       Unhealthy
                   |             |
                Continue      Stop /
                              Rollback /
                            Disable flag
```

The key production question is not only:

> Did the deployment command succeed?

It is:

> Is the new version healthy for users, dependencies and the business?

---

## 3. Build Once, Promote the Same Artifact

CI produces a versioned image, for example:

```text
registry.example.com/mern-api:9f72c1
```

The registry stores that image. Staging and production should deploy the same digest or immutable image that was validated in CI and staging.

Preferred flow:

```text
CI
  |
  v
ONE image
  |
  v
Registry
  |
  +--> Staging
  |
  +--> Production
```

Avoid rebuilding for each environment:

```text
CI       -> build v2
Staging  -> rebuild v2
Production -> rebuild v2
```

A rebuild can change dependency resolution, base layers, generated files or build timestamps. Even if the source commit is identical, the resulting artifact may differ. Building once preserves artifact consistency and makes rollback traceable.

### Tagging guidance

Use a commit SHA or another immutable identifier for deployment:

```text
mern-api:9f72c1
```

A human-readable tag may also be useful:

```text
mern-api:release-2026-09-27
```

Do not use a mutable tag such as `latest` as the only production reference. Record the image digest where the platform supports it.

---

## 4. Staging and Promotion

Staging answers:

> Does this deployable artifact work in a production-like environment?

A production promotion should require evidence:

```text
CI checks       PASS
Image scan      PASS or accepted exception
Staging deploy  PASS
Health checks   PASS
Smoke tests     PASS
Promotion       APPROVED
```

A failed staging smoke test should block production. This is a release gate, not an optional report.

### Health versus smoke tests

A health or readiness check verifies that an instance can safely receive traffic. A smoke test verifies a small set of important user-facing behaviors.

```text
Container starts
      |
      v
Readiness passes
      |
      v
Smoke test passes
      |
      v
Promotion can be considered
```

A container being `Up` only proves that its process is running. If `/health` returns HTTP 500, the instance should not receive production traffic.

---

## 5. Deployment Strategies

### Rolling deployment

A rolling deployment replaces instances gradually:

```text
v1 v1 v1
  |
  v
v2 v1 v1
  |
  v
v2 v2 v1
  |
  v
v2 v2 v2
```

**Strengths:** simple, efficient and widely supported.  
**Risks:** old and new versions coexist; compatibility and database migrations need care.

### Blue-green deployment

Two environments exist:

```text
BLUE  -> v1 -> receives traffic
GREEN -> v2 -> validated without primary traffic
```

Traffic switches when green is ready:

```text
BLUE  -> v1
GREEN -> v2
          ^
          |
      switch traffic
```

**Strengths:** fast rollback by switching traffic back.  
**Risks:** requires capacity for two environments and careful handling of shared data.

### Canary deployment

A small percentage of traffic reaches the new version:

```text
95% -> v1
 5% -> v2
```

If signals remain healthy, increase exposure:

```text
90/10 -> 50/50 -> 0/100
```

**Strengths:** limits blast radius and supports evidence-based rollout.  
**Risks:** needs reliable routing, representative traffic and strong monitoring.

Choose the strategy based on risk, capacity, compatibility, observability and rollback speed.

---

## 6. Release Safety and Recovery

A production release does not end when a container starts:

```text
Deployment
    |
    v
Health
    |
    v
Metrics
    |
    v
Logs
    |
    v
Business signals
    |
    v
Release decision
```

Use the smallest safe recovery mechanism:

| Problem | Preferred first response |
|---|---|
| Isolated feature is broken | Disable the feature flag |
| Application release is broadly broken | Stop rollout or roll back the image |
| Infrastructure or dependency is failing | Recover the infrastructure or dependency |
| Unknown impact during a canary | Pause exposure and investigate |

A feature flag is useful only when it is isolated, tested and operationally safe to disable. A flag should not become a substitute for removing broken code permanently.

---

## 7. Observability Review

The three pillars answer different questions:

| Signal | Question | Examples |
|---|---|---|
| Metrics | What is changing and how much? | 5xx rate, p95 latency, CPU, memory, request rate, checkout success |
| Logs | What happened? | `MongoServerSelectionError`, payment timeout, failed operation |
| Traces | Where did the request spend time? | Nginx, API, MongoDB and external provider spans |

A useful investigation connects them:

```text
Metric alert
    |
    v
Establish scope
    |
    v
Correlate with deployment
    |
    v
Search structured logs
    |
    v
Follow request or trace ID
    |
    v
Check dependencies
    |
    v
Mitigate and preserve evidence
```

### Structured production log

```json
{
  "level": "error",
  "service": "mern-api",
  "version": "8f31c2a",
  "requestId": "7fa92c",
  "route": "/api/orders",
  "status": 500,
  "durationMs": 842,
  "errorType": "MongoServerSelectionError"
}
```

Never log passwords, JWTs, API keys, database passwords, credit-card data or entire application objects.

### Business signals matter

Normal infrastructure metrics do not prove that users are successful:

```text
CPU       = 49%
Memory    = 54%
Checkout  = 81% success
```

The checkout metric is a severe business regression even though CPU and memory look normal.

---

## 8. Review Questions and Answers

### Q1. Why should production deploy the same Docker image that passed staging instead of rebuilding it?

To guarantee that production runs the exact artifact that was validated. Rebuilding can create a different artifact and weakens traceability and rollback confidence.

### Q2. A container is `Up`, but `/health` returns HTTP 500. Should the instance receive production traffic?

No. `Up` only proves that the process is running. A failed health or readiness check indicates that the instance is not ready to serve traffic.

### Q3. A canary has 95% v1 and 5% v2. v2 has a dramatically higher 5xx rate. What should happen?

Pause or reverse the canary rollout, remove v2 traffic and investigate before increasing exposure.

### Q4. The new checkout feature is broken, but the rest of the application is healthy. It is behind a feature flag. What is the preferred immediate recovery?

Disable the feature flag if it is safely isolated and doing so restores the previous behavior.

### Q5. The API 5xx rate jumps after deployment. Should you inspect only CPU?

No. Establish scope with metrics, correlate with deployment timing, inspect structured logs and request IDs, and check dependencies such as MongoDB and payment services.

---

## 9. Interview Drill

### Question 1: Explain your CI/CD pipeline for a production MERN application.

A strong structure is:

```text
Git push
    |
    v
CI
    |
    +-- Install dependencies
    +-- Lint and tests
    +-- Build
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
Smoke tests
    |
    v
Production
    |
    v
Monitor and decide
```

Mention immutable image tagging, approval gates where appropriate, deployment strategy, health checks and rollback.

### Question 2: How would you achieve zero-downtime deployment?

Mention multiple instances, a load balancer or reverse proxy, readiness checks, a rolling, blue-green or canary strategy, monitoring, backward-compatible changes and a tested rollback path.

### Question 3: A deployment succeeds but users report failures. What do you do?

```text
Measure
  |
  v
Scope the impact
  |
  v
Correlate with deployment
  |
  v
Inspect logs and traces
  |
  v
Check dependencies
  |
  v
Mitigate
  |
  v
Roll back or disable a flag if necessary
  |
  v
Root-cause analysis
```

Do not start by restarting everything. First preserve evidence and identify the smallest safe recovery.

---

## 10. Practical Assessment: Canary Incident

You deploy `mern-api:9f72c1` using a canary strategy.

Initial state:

```text
v1 -> 100%
```

After deployment:

```text
v1 -> 90%
v2 -> 10%
```

Five minutes later:

| Signal | Before | After |
|---|---:|---:|
| 5xx rate | 0.3% | 5.8% |
| p95 latency | 290 ms | 1.9 s |
| CPU | 45% | 49% |
| Memory | 52% | 54% |
| Checkout success | 99% | 81% |

Application logs show:

```text
event=checkout_failed
requestId=83ad91
errorType=PaymentTimeout
durationMs=5000
```

### Assessment answers

**1. Should rollout continue?**  
No. Error rate, latency and checkout success all show significant degradation.

**2. What should happen immediately?**

```text
Stop rollout
    |
    v
Reduce or remove v2 traffic
    |
    v
Restore v1 if necessary
```

**3. What should you investigate?**

- Payment integration behavior.
- API timeout configuration.
- Network connectivity.
- Recent code changes in v2.
- External payment provider health.
- v2-specific logs and traces.
- Whether only checkout or other flows are affected.

**4. Why is CPU not enough to dismiss the problem?**

Because normal resource utilization does not mean the application is healthy. A payment timeout can create severe business impact without exhausting CPU or memory.

**5. What could provide faster recovery if checkout is behind a feature flag?**

Disable the checkout feature if the flag safely restores the known-good behavior. Continue with rollback if the failure is broader than the feature.

---

## 11. Practical Assessment: Design Your MERN Release

Design this flow without referring to the notes:

```text
Developer
    |
    v
Git
    |
    v
CI
    |
    +-- Tests
    +-- Docker build
    +-- Security scan
    |
    v
Registry
    |
    v
Staging
    |
    +-- Health checks
    +-- Smoke tests
    |
    v
Canary / Rolling / Blue-green
    |
    v
Metrics + logs + traces
    |
    v
Healthy?
  +---+---+
  |       |
 Yes      No
  |       |
  v       v
Continue  Roll back /
rollout   disable flag
```

Create a `RELEASE-RUNBOOK.md` containing:

1. Build process.
2. Image naming and tagging.
3. Registry location.
4. Staging deployment.
5. Health check.
6. Smoke tests.
7. Deployment strategy.
8. Release metrics.
9. Rollback procedure.
10. Feature-flag procedure.
11. Incident investigation steps.

The runbook should be executable by another engineer without relying on undocumented tribal knowledge.

---

## 12. Monthly Cumulative Project Milestone

The MERN DevOps project should now have:

- Git-based workflow.
- Continuous integration.
- Automated tests.
- Docker packaging.
- Image scanning.
- Registry publishing.
- Versioned artifacts.
- Staging deployment.
- Smoke tests.
- Promotion gates.
- A deployment strategy.
- Feature flags where appropriate.
- Rollback capability.
- Health and readiness checks.
- Metrics.
- Structured logs.
- Incident investigation guidance.

The monthly goal is a reproducible production delivery system:

```text
MERN application
       |
       v
      Git
       |
       v
       CI
       |
  +----+----+
  v    v    v
Test Build Scan
  +----+----+
       |
       v
   Registry
       |
       v
   Staging
       |
  +----+----+
  v         v
Health    Smoke
  +----+----+
       |
       v
  Production
       |
       v
 Canary / Rolling / Blue-green
       |
       v
 Observe
       |
  +----+----+
  v         v
Healthy  Unhealthy
  |         |
  v         v
Release  Recover
```

**Project milestone:** the deployment must be reproducible from Git and have a documented, testable rollback path.

---

## 13. Weekly Master Summary

The progression this week was:

```text
Day 44 -> Artifact delivery
   |
Day 45 -> Environment promotion
   |
Day 46 -> Deployment strategies
   |
Day 47 -> Release safety
   |
Day 48 -> Observability
   |
Day 49 -> Production logging
```

You have moved from:

> How do I deploy a Docker container?

Toward:

> How do I safely operate and release a production system?

That is the central DevOps transition: connecting tools into a controlled system with evidence, recovery and accountability.

---

## 14. Final Interview Challenge

Answer this aloud without looking at the lesson:

> You are responsible for deploying a new version of a Dockerized MERN application. Explain your production release process from Git commit through successful production verification, and explain what you do if the release fails.

Your answer should naturally include:

```text
Git
  |
  v
CI
  |
  v
Tests
  |
  v
Docker build
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
Smoke tests
  |
  v
Canary / Rolling / Blue-green
  |
  v
Health checks
  |
  v
Metrics + logs + business KPIs
  |
  v
Continue
  OR
Rollback / disable feature
```

A strong answer describes the decision points, not only the tools.

---

## 15. Weekly Assessment Scoring

| Area | Weight |
|---|---:|
| CI/CD pipeline understanding | 20% |
| Docker and registry | 15% |
| Staging and promotion | 15% |
| Deployment strategies | 15% |
| Rollback and release safety | 15% |
| Observability and logging | 10% |
| Incident troubleshooting | 10% |
| **Total** | **100%** |

| Score | Guidance |
|---|---|
| 80% or higher | Ready to progress confidently |
| 60-79% | Review weak areas before the next project milestone |
| Below 60% | Repeat the week's practical exercises before moving deeper into production operations |

---

## Day 51 Preview

The normal syllabus resumes with centralized logging and alerting architecture:

```text
Containers
    |
    v
stdout / stderr
    |
    v
Log collector
    |
    v
Central log store
    |
    v
Search
    |
    v
Alerts
    |
    v
Incident response
```

The focus will be log pipeline design, alert quality, retention, security, alert fatigue and MERN production troubleshooting.
