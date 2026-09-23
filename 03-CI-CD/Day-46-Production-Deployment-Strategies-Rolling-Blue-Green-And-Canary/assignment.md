# Day 46 - Assignment: Production Deployment Strategies

## Objective

Understand the difference between rolling, blue-green, and canary deployment strategies and apply them to a Dockerized MERN application in production.

This assignment focuses on zero-downtime release patterns, safe rollback, and compatibility concerns during deployment.

---

## Scenario

Your MERN API is running in production with multiple instances behind Nginx or a load balancer. You are preparing a new release of the application and need to choose a strategy that keeps the application available while reducing the risk of a bad release.

The deployment question is no longer:

> "How do we push a new image?"

It is:

> "How do we do it without breaking production users, and how do we recover if it fails?"

---

## Part 1: Explain the Strategies

Write short but practical answers to these questions:

1. What is a rolling deployment?
2. What is a blue-green deployment?
3. What is a canary deployment?
4. Why is downtime a serious issue in production?
5. What is zero-downtime deployment?
6. Why does a deployment strategy matter even when the container starts successfully?
7. What is the main difference between rolling and blue-green?
8. What is the main difference between canary and rolling?

Keep your answers specific to real production environments.

---

## Part 2: Compare Their Trade-offs

Create a comparison table like this:

| Strategy | How it works | Best for | Benefits | Risks |
|----------|--------------|----------|----------|-------|
| Rolling | Gradually replaces old instances | Small-medium production environments | Low overhead, gradual rollout | Mixed versions during rollout |
| Blue-Green | Runs two full environments and switches traffic | Fast rollback needs | Very fast revert | More infrastructure cost |
| Canary | Sends small traffic to new version first | Risk-sensitive releases | Limits blast radius | Requires observability and automation |

Then answer:

- Which strategy is best for a low-risk application with limited infrastructure?
- Which strategy is best when rollback speed matters most?
- Which strategy is best when the release is high-risk?

---

## Part 3: Version Compatibility

Imagine a production database currently contains the field:

```text
users.name
```

You deploy a new app version that expects:

```text
users.display_name
```

Explain what can go wrong during a rolling deployment.

Then explain how the following pattern reduces risk:

```text
Expand schema -> Deploy compatible version -> Migrate data -> Remove old dependency
```

Your answer should explain:

- why mixed versions can fail
- why the migration must be backward compatible
- why destructive DB changes are dangerous during deployment

---

## Part 4: Health Checks and Safety Gates

For each strategy, describe how health checks should be used:

1. Rolling deployment
2. Blue-green deployment
3. Canary deployment

Your answer must include:

- what should happen before traffic is routed to the new instance
- which signal should be used to decide whether to continue
- what to do when metrics worsen

---

## Part 5: Design a Rolling Deployment for MERN

Your app has 3 API instances behind Nginx.

Initial state:

```text
api-1 -> v1
api-2 -> v1
api-3 -> v1
```

Target state:

```text
api-1 -> v2
api-2 -> v2
api-3 -> v2
```

Write the rollout sequence in steps.

Include:

- health check requirement
- how traffic is shifted
- what happens if the new instance fails
- what metrics you monitor during rollout
- how rollback occurs

---

## Part 6: Failure Drill

Simulate this production scenario:

```text
v1 error rate = 0.3%
v2 error rate = 8.2%
```

At only 20% traffic exposure, what should you do?

Write a step-by-step incident response:

1. Stop rollout
2. Keep majority traffic on v1
3. Inspect logs and metrics
4. Remove or isolate v2
5. Restore traffic to v1
6. Investigate the root cause

Then explain why continuing to increase traffic would be a bad decision.

---

## Part 7: Rollback Decision Policy

Create a rollback policy for your MERN deployment.

Your policy should include:

```text
- health checks failed
- error rate increased beyond threshold
- latency crossed acceptable levels
- smoke tests failed
- database compatibility issue detected
- required approval not completed
```

Explain why rollback must be designed before deployment, not after failure.

---

## Part 8: MERN Production-Safe Notes

List 5 practical deployment considerations for a Dockerized MERN application:

- Nginx load balancing
- MongoDB compatibility
- health endpoint
- image tag strategy
- observability and alerting

Explain why each one matters.

---

## Part 9: Interview Practice

Prepare a strong answer to this question:

> How do you achieve zero-downtime deployment for a Dockerized MERN application?

Your answer should mention:

- multiple API instances
- load balancer or reverse proxy
- health checks before traffic routing
- rolling/blue-green/canary strategy selection
- rollback capability
- backward-compatible database migrations

---

## Part 10: Final Reflection

Write a short paragraph answering:

> Which deployment strategy would you choose for a small MERN project in early production, and why?

Then explain a case where you would choose blue-green instead, and a case where canary would be appropriate.

---

## Deliverable

Create a file named:

```text
DEPLOYMENT.md
```

with the following sections:

```text
Deployment strategy:
Initial version:
Target version:
Health check:
Rollback plan:
Database compatibility strategy:
Monitoring during deployment:
Failure criteria:
```

This file should be practical enough to show your production deployment thought process.
