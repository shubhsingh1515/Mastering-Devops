# DevOps Mentorship Program - Day 47

## Phase 3: CI/CD

### Safe Releases: Rollback, Feature Flags and Release Safety

**Level:** Intermediate -> Professional   
**Focus:** Rollbacks, feature flags, progressive releases, monitoring and MERN production incidents

Yesterday, Day 46, you learned three deployment strategies:

- Rolling
- Blue-green
- Canary

Today we add the safety mechanisms that make those strategies useful in production:

```text
Deployment
    |
    v
Health checks
    |
    v
Metrics
    |
    v
Feature flags
    |
    v
Gradual release
    |
    v
Rollback or disable feature
```

The central idea is:

> Deployment and feature release do not have to be the same event.

Deploying code means putting an artifact into a production environment. Releasing a feature means allowing users to exercise that behavior. A safe system can do those two things independently.

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain application rollback versus feature rollback.
- Explain why feature flags reduce deployment risk.
- Design a safe MERN feature rollout.
- Identify when to stop or roll back a deployment.
- Compare automated and manual rollback.
- Explain backward-compatible database releases.
- Connect deployment metrics to release decisions.
- Answer production rollback questions in interviews.

---

## 2. Deployment Is Not Feature Release

Suppose the registry contains:

```text
registry.example.com/mern-api:v2
```

and v2 contains a new checkout system. You do not have to expose checkout to every user the moment v2 starts.

A safer sequence is:

```text
Deploy v2
    |
    v
Feature OFF
    |
    v
Validate infrastructure
    |
    v
Enable for a small group
    |
    v
Monitor
    |
    v
Increase exposure or recover
```

Deployment is the act of making code available. Release is the act of making functionality available. Separating them gives the team time to validate infrastructure and observe the new path before the blast radius becomes large.

---

## 3. Feature Flags

A feature flag is a runtime decision that controls whether a behavior is active.

```javascript
if (featureFlags.newCheckout) {
  return newCheckout(request);
}

return existingCheckout(request);
```

Conceptually:

```text
User request
     |
     v
Feature flag evaluation
   /                 \\
OFF                   ON
 |                     |
v1 behavior       v2 behavior
```

A useful feature flag should have:

- a safe default, usually disabled for a new high-risk feature
- a clear owner
- a documented removal date
- an audit trail for changes
- predictable behavior when the flag service is unavailable
- targeting rules for environments, users or percentages

Do not treat flags as a permanent replacement for good testing. They are a release-control mechanism, not a substitute for code quality.

### Environment variable example

```text
NEW_CHECKOUT_ENABLED=false
```

A basic MERN API can read this setting at startup:

```javascript
const newCheckoutEnabled = process.env.NEW_CHECKOUT_ENABLED === 'true';
```

For a production rollout, a runtime configuration service is often preferable because it can change the flag without rebuilding an image. Whatever mechanism is used, protect it with authentication, authorization, validation and an audit log.

---

## 4. MERN Checkout Rollout

Without a flag:

```text
Deploy v2
    |
    v
100% of users receive checkout
    |
    v
Bug affects everyone
    |
    v
Emergency rollback
```

With a flag:

```text
Deploy v2
    |
    v
newCheckout = false
    |
    v
Validate the deployed application
    |
    v
Enable for internal users
    |
    v
Enable for 1%
    |
    v
Enable for 10%
    |
    v
Enable for 50%
    |
    v
Enable for 100%
```

At every step, compare the new path with the baseline. For checkout, useful business signals include:

- checkout success rate
- payment authorization failures
- order creation failures
- cart abandonment
- duplicate order reports
- refund or cancellation rate

A service can be technically healthy while the business feature is broken. A passing `/health` endpoint only proves that the health check passed.

---

## 5. Canary Deployment Plus Feature Flags

Canary and feature flags control different dimensions of risk.

- Canary controls which traffic reaches the application version.
- A feature flag controls which behavior is active for selected users.

They can be combined:

```text
5% of traffic -> v2
newCheckout   -> OFF
```

First validate that v2 can serve traffic. Then enable checkout for a controlled group within that traffic:

```text
5% of traffic -> v2
internal users -> new checkout ON
```

This creates two safety layers. If the application version is unstable, stop the canary. If only checkout is defective, disable the checkout flag while leaving v2 available for other functionality.

---

## 6. What to Monitor

Never make a release decision from a single HTTP 200 check. Monitor technical and business signals together.

### Technical signals

- 4xx and 5xx error rate
- p50, p95 and p99 latency
- availability and request volume
- CPU, memory and container restarts
- database connection pool usage
- MongoDB query latency and errors
- logs and traces
- dependency failures

### Business signals

- checkout success rate
- payment failure rate
- order creation rate
- cart abandonment
- login success rate
- revenue or conversion rate

Compare v2 with the known-good baseline. A threshold should include a time window and enough traffic to avoid reacting to one isolated request.

---

## 7. When to Roll Back

Example:

```text
v1 5xx rate = 0.4%
v2 5xx rate = 7.5%
```

That is a strong rollback signal, especially if it is sustained and user-facing. A release decision should consider:

```text
Error rate
    +
Latency
    +
Availability
    +
Business impact
    +
Trend and duration
```

A healthy API with a 300% increase in checkout failures is not a successful release. Stop the rollout, reduce exposure and choose the smallest safe recovery action.

---

## 8. Two Levels of Application Recovery

### Level 1: Feature rollback

```text
newCheckout = true
        |
        v
newCheckout = false
```

The feature is disabled while the application image continues running. This is usually the fastest response when the defect is isolated behind a flag.

### Level 2: Application rollback

```text
mern-api:v2
        |
        v
mern-api:v1
```

Return traffic to the previous known-good image when the problem affects the application more broadly, the flag cannot contain the issue, or the new release is unstable.

### Level 3: Infrastructure recovery

If the problem is caused by infrastructure or configuration, recover the affected infrastructure or restore the known-good configuration. Application rollback alone cannot repair every failure.

Use the smallest recovery mechanism that safely reduces user impact.

---

## 9. The Database Rollback Trap

Imagine v2 renames:

```text
users.name -> users.display_name
```

and removes `users.name`. If v2 fails and you immediately restore v1, v1 may still expect `users.name` and fail too.

Therefore:

> Application rollback and database rollback are separate problems.

Use an expand-and-contract migration:

```text
1. Expand: add the new field while keeping the old field.
2. Deploy code that can read and write both fields.
3. Backfill old records into the new field.
4. Switch normal behavior to the new field.
5. Verify that old application versions are no longer needed.
6. Contract: remove the old field in a later release.
```

This preserves compatibility while old and new application versions coexist.

Avoid destructive schema changes in the same release as a risky application change. A rollback plan is incomplete if the previous binary cannot use the current schema.

---

## 10. Automated and Manual Rollback

A release controller can automate a response to a well-understood signal:

```text
Deploy v2
    |
    v
Monitor for 5 minutes
    |
    v
5xx rate > 5% for 3 consecutive minutes?
   /                         \\
 No                           Yes
 |                             |
Continue                    Roll back
```

Automation can reduce recovery time, but a careless threshold can cause flapping:

```text
Temporary spike -> automatic rollback -> redeploy -> rollback again
```

Automate clear, severe and reliably measured failures. Use human investigation for ambiguous failures involving data integrity, external payment providers, migrations or unusual traffic patterns.

A strong policy defines:

- the metric and query
- the threshold
- the evaluation window
- the minimum traffic volume
- the action
- who is notified
- how to pause automation

---

## 11. Production Incident Drill

You deploy:

```text
mern-api:v2.4.0
```

Infrastructure appears healthy:

```text
CPU         OK
Memory      OK
Containers  OK
Health      OK
```

Users report checkout failures. Investigation shows:

```text
API error rate:       normal
Health endpoint:      normal
Database:             normal
Checkout success:     98% -> 61%
```

This is a feature-level failure. If checkout is behind `newCheckout`, disable it immediately. No image rebuild, container rollback or infrastructure change is required for the first recovery action.

Then preserve evidence, open an incident record and investigate the checkout path, payment integration, configuration, data writes and recent changes.

---

## 12. Canary Decision Drill

At 10% traffic:

```text
v1 5xx rate = 0.3%
v2 5xx rate = 6.7%

v1 latency = 180 ms
v2 latency = 620 ms
```

Correct response:

```text
Stop the rollout
    |
    v
Keep v1 serving the majority of traffic
    |
    v
Inspect logs, traces and business metrics
    |
    v
Remove or isolate v2 if necessary
    |
    v
Return traffic to v1
    |
    v
Investigate before redeploying
```

Do not increase exposure simply because deployment succeeded. A successful container start is not evidence that the release is safe under real traffic.

---

## 13. Practical Exercise

Add a conceptual flag to your MERN application:

```text
NEW_CHECKOUT_ENABLED=false
```

Test this sequence:

1. Deploy the application with the flag disabled.
2. Confirm that the existing checkout behavior remains available.
3. Enable the flag in staging.
4. Test the new checkout path.
5. Enable the flag for a controlled production group.
6. Monitor technical and business metrics.
7. Disable the flag without rebuilding the Docker image.
8. Confirm that the old behavior returns.

The key objective is to demonstrate that feature recovery can happen independently of image creation.

---

## 14. Day 47 Summary

```text
CI
 |
v
Docker build
 |
v
Scan and registry
 |
v
Staging and smoke tests
 |
v
Production deployment
 |
v
Canary or gradual release
 |
v
Feature flag
 |
v
Observe metrics
 /              \\
Healthy          Unhealthy
 |                    |
Increase exposure    Disable or rollback
```

The most important lesson is:

> A good deployment strategy reduces blast radius. A good rollback strategy reduces recovery time. A good feature-flag strategy can reduce both.

For a production MERN system, keep the image immutable, keep the previous artifact available, monitor business outcomes, use backward-compatible migrations and choose the smallest safe recovery action.

**Next lesson:** CI/CD observability and deployment monitoring: logs, metrics, health checks, latency, error rates and automated release decisions.
