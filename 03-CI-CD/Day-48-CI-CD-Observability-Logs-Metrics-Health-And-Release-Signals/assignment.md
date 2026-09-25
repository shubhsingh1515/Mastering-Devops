# Day 48 - Assignment: CI/CD Observability and Release Signals

## Objective

Design an observability and release-monitoring strategy for a Dockerized MERN application. Your work should show how logs, metrics, traces, health checks and business signals guide a deployment decision.

Use a disposable or staging environment. Never use real customer, payment or production secrets in exercises.

---

## Part 1: Core Concepts

Answer with MERN examples:

1. What do logs tell you?
2. What do metrics tell you?
3. What do traces tell you?
4. Why is deployment success different from application health?
5. What is the difference between liveness and readiness?
6. Why can a simple `{"status":"ok"}` health response be misleading?
7. What is error rate?
8. What do p95 and p99 latency represent?
9. What is saturation?
10. Why are business metrics required in addition to CPU and memory?

---

## Part 2: Observability Design

Create a release dashboard with at least these panels:

- availability
- request rate
- 4xx rate
- 5xx rate
- p95 and p99 latency
- CPU and memory
- container restarts
- MongoDB latency and errors
- checkout success rate
- payment completion rate
- order creation failures

For each panel, document its source, aggregation, baseline and alert threshold.

---

## Part 3: Health and Readiness

Design `/health` and `/ready` behavior for your API. State:

- which endpoint proves process liveness
- which dependencies readiness checks
- what status code is returned when MongoDB is unavailable
- how dependency-check timeouts are bounded
- how the load balancer or orchestrator uses readiness
- how you prevent the check from creating significant database load

---

## Part 4: Structured Logging

Define a JSON log format containing:

```text
 timestamp
 level
 service
 environment
 release_version
 request_id
 method
 route
 status_code
 duration_ms
 error_code
```

List at least five fields that must not be logged in plain text, such as passwords, access tokens and payment credentials. Explain how redaction works.

---

## Part 5: Release Health Gate

Write release criteria for a 5-minute observation window. Include:

```text
5xx rate
p95 latency
readiness
MongoDB errors
checkout success
minimum request sample
```

For every criterion, state the action when it fails. Explain why a minimum sample size and consecutive-failure window reduce noisy decisions.

---

## Part 6: Incident Drill

A new release produces the following result:

```text
Before       After
5xx 0.3%     5xx 4.7%
p95 280 ms   p95 1.4 s
CPU 45%      CPU 47%
Memory 52%   Memory 54%
Checkout 98.8% Checkout 82.1%
```

Write a response that includes:

1. how you confirm the deployment correlation
2. how you stop further rollout
3. how you keep the previous version available
4. which logs, traces and metrics you inspect
5. how you check MongoDB and external payment latency
6. when feature disablement is enough
7. when application rollback is justified
8. what evidence you preserve
9. what must be true before retrying the release

Explain why normal CPU and memory do not prove this release is healthy.

---

## Part 7: Practical Exercise

In a disposable MERN environment:

1. Start a release with a version identifier.
2. Call `/health` and `/ready`.
3. Generate safe test traffic.
4. Capture request rate, status codes and latency.
5. Add a deployment marker to your notes or dashboard.
6. Find one request by its correlation ID.
7. Simulate a dependency failure and observe readiness.
8. Record the recovery action and recovery time.

Submit an evidence log with commands, timestamps, observations and results.

---

## Part 8: Reflection

Answer these questions in your own words:

1. Why is business health different from infrastructure health?
2. Why should p95 often be more useful than average latency?
3. What does a correlation ID make possible?
4. Why should release decisions compare against a baseline?
5. Why should an operator avoid restarting containers as the first response?
6. Which signal would make you pause a rollout immediately?
