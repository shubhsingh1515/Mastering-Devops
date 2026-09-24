# Day 47 - Assignment: Safe Releases and Feature Flags

## Objective

Design a release-safety process for a Dockerized MERN application. Your work should show how deployment, feature release, monitoring and rollback fit together.

The key question is:

> How can a team deploy code safely, release a risky feature gradually and recover quickly when production evidence shows a problem?

---

## Part 1: Core Concepts

Write practical answers to the following:

1. What is the difference between deployment and feature release?
2. What is a feature flag?
3. Why should a new high-risk feature default to disabled?
4. What is an application rollback?
5. What is a feature rollback?
6. Why is disabling a feature often faster than reverting an image?
7. What is progressive or gradual rollout?
8. How does a canary deployment differ from a feature flag?
9. Why can a passing health endpoint be insufficient?
10. What does reducing blast radius mean?

Use examples from the MERN checkout scenario rather than dictionary definitions only.

---

## Part 2: Design the Feature Flag

Define the following flag:

```text
NEW_CHECKOUT_ENABLED=false
```

Document:

- the safe default
- the owner of the flag
- the environments where it exists
- the users or groups that can receive it
- the audit information recorded for every change
- the fallback behavior if the flag service is unavailable
- the date or release in which the flag should be removed

Explain why a feature flag must not become a permanent hidden branch in the codebase.

---

## Part 3: Create a Release Plan

Your current production release is `mern-api:v1`. The new release is `mern-api:v2`, containing a new checkout flow.

Write a staged release plan with these phases:

```text
1. Build and scan the immutable image.
2. Deploy v2 with the feature disabled.
3. Run health checks and smoke tests.
4. Enable the feature for internal users.
5. Observe technical and business metrics.
6. Enable 1%, then 10%, then 50%, then 100% if healthy.
7. Stop or recover if release criteria fail.
```

For every phase, state:

- who or what performs the action
- what evidence is required to continue
- what action is taken if the evidence is negative

---

## Part 4: Monitoring Design

Create a table with at least eight signals:

| Signal | Type | Example threshold | Window | Action |
|---|---|---|---|---|
| 5xx rate | Technical | More than 5% | 3 minutes | Stop rollout and investigate |
| Checkout success | Business | Below agreed baseline | 5 minutes | Disable feature |

Include both technical and business metrics. At minimum, discuss:

- 4xx and 5xx error rate
- p95 latency
- availability
- CPU and memory
- container restarts
- database errors
- payment failures
- checkout success rate
- order creation failures
- cart abandonment

Explain why one metric alone should not normally decide a release.

---

## Part 5: Canary Incident Drill

At 10% exposure, you observe:

```text
v1 5xx rate = 0.3%
v2 5xx rate = 6.7%

v1 latency = 180 ms
v2 latency = 620 ms
```

Write a step-by-step response that includes:

1. stopping further exposure
2. keeping the majority of traffic on v1
3. checking logs, traces and metrics
4. deciding whether v2 should be isolated
5. restoring traffic to v1 if necessary
6. preserving evidence
7. opening a root-cause investigation
8. defining what must be true before retrying the release

Explain why increasing traffic to 25%, 50% or 100% would be unsafe.

---

## Part 6: Feature-Level Incident Drill

The infrastructure is healthy:

```text
CPU: normal
Memory: normal
Containers: healthy
API error rate: normal
Checkout success: 98% -> 61%
```

The new checkout flow is controlled by a feature flag. Write the immediate response and the follow-up investigation.

Your answer must distinguish:

- user-impact reduction
- feature rollback
- application rollback
- infrastructure recovery
- root-cause analysis

State why disabling the flag is the smallest safe recovery action in this scenario.

---

## Part 7: Database Compatibility

A v2 migration renames:

```text
users.name -> users.display_name
```

and removes the old field. Explain why restoring v1 may fail after the migration.

Design an expand-and-contract sequence:

```text
Expand -> Compatible code -> Backfill -> Switch behavior -> Verify -> Contract
```

Your answer should state:

- why both fields may need to coexist
- how old and new application versions can be supported
- when the old field can safely be removed
- why an emergency rollback should not make destructive schema changes

---

## Part 8: Automated Rollback Policy

Write a policy for this rule:

```text
IF v2 5xx rate > 5% for 3 consecutive minutes
AND request volume is above the minimum sample size
THEN stop the rollout and notify the on-call engineer
```

Discuss:

- why the time window matters
- why minimum traffic matters
- how to avoid flapping
- when a human approval is required
- when automatic rollback may not solve the problem
- how to pause or override the automation safely

---

## Part 9: Practical MERN Exercise

In a disposable MERN environment:

1. Start v2 with `NEW_CHECKOUT_ENABLED=false`.
2. Confirm the old checkout behavior.
3. Enable the flag in staging.
4. Test the new checkout behavior using test payment data.
5. Record health, latency and checkout results.
6. Disable the flag without rebuilding the image.
7. Confirm that the old behavior returns.
8. Record the exact recovery time.

Deliver a short evidence log containing commands, observations and results. Never use production customer or payment data in this exercise.

---

## Part 10: Release-Safety Document

Create `RELEASE-SAFETY.md` containing:

```text
Deployment strategy
Feature-flag strategy
Health checks
Smoke tests
Progressive rollout stages
Monitoring metrics
Rollback thresholds
Feature rollback procedure
Application rollback procedure
Database compatibility strategy
Incident contacts and approvals
```

Keep the document operational. Another engineer should be able to use it during a release or incident without guessing the next action.

---

## Part 11: Interview Practice

Prepare a strong answer to:

> A new MERN checkout feature is deployed, but users report payment failures. What do you do?

Your answer should mention:

- scope and severity
- logs, metrics and business KPIs
- feature-flag disablement
- stopping a canary or gradual rollout
- application rollback when required
- payment provider and database investigation
- backward-compatible schema changes
- staging validation before redeployment

---

## Deliverables

Submit:

1. Written answers for Parts 1-8.
2. The practical evidence log from Part 9.
3. `RELEASE-SAFETY.md` from Part 10.
4. A concise interview answer from Part 11.

A strong submission makes the recovery path explicit and chooses the smallest safe action for each failure type.
