# DevOps Mentorship Program - Day 52

## Phase 3: CI/CD -> Production Operations

### Production Incident Response and Troubleshooting

**Duration:** 25-30 minutes  
**Focus:** Incident triage, mitigation, root-cause analysis, rollback decisions and MERN production troubleshooting

Yesterday, Day 51, you learned centralized logging and alerting:

```text
Application
     |
     v
Structured logs
     |
     v
Log collector
     |
     v
Central storage
     |
     v
Search / Alerts
     |
     v
Incident response
```

Today we complete that loop.

When production breaks, how do you respond systematically without making the incident worse?

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Distinguish detection, triage, mitigation and root-cause analysis.
- Assess production impact quickly and communicate it clearly.
- Build a systematic incident-response workflow.
- Troubleshoot a MERN production outage using evidence.
- Decide when to rollback and when to investigate or fix forward.
- Explain why random restarts can make an incident harder to understand.
- Verify recovery using technical and business signals.
- Produce a useful post-incident review with preventive actions.
- Explain incident response confidently in DevOps interviews.

---

## 2. The Incident Lifecycle

A production incident typically follows this lifecycle:

```text
Detect
  |
  v
Triage
  |
  v
Assess impact
  |
  v
Mitigate
  |
  v
Investigate
  |
  v
Recover
  |
  v
Verify
  |
  v
Learn
```

These stages are related, but they are not interchangeable.

### Detection

Something appears abnormal. The signal might come from an alert, a dashboard, a health check, a support ticket or a customer report.

### Triage

Determine what is failing, where it is failing, when it started and how serious it is. Triage turns a vague report into a bounded incident.

### Impact assessment

Estimate the user and business consequences. A technical error rate alone does not tell you whether checkout, payments, data integrity or only an internal admin screen is affected.

### Mitigation

Take a safe action that reduces user impact. Mitigation is about stabilizing the situation, not proving the complete root cause.

### Investigation

Use logs, metrics, deployment history, configuration and dependency evidence to test possible causes.

### Recovery

Apply the change that returns the service to a stable state, such as disabling a feature, rolling back a release or fixing an invalid configuration.

### Verification

Confirm that the user-facing service and business outcomes have actually recovered. A process running in Docker is not enough.

### Learning

Document the timeline, root cause, contributing factors and corrective actions so that the same class of incident becomes less likely.

---

## 3. Example Alert: API 5xx Above 5 Percent

Suppose monitoring reports:

```text
PRODUCTION API 5XX > 5%
```

Do not immediately SSH into a server and restart it. First establish:

- What changed recently?
- How many users are affected?
- Which endpoints are failing?
- Which application version is serving requests?
- When did the problem begin?
- Is the error rate increasing, decreasing or stable?
- Is data being lost or duplicated?
- Are payments or other critical workflows affected?

This is triage. The alert is only the starting signal; it is not yet a diagnosis.

---

## 4. Step 1 - Establish Scope

Suppose the production dashboard shows:

```text
5xx rate:       0.3% -> 8.2%
Traffic:        normal
p95 latency:    300ms -> 2.4s
Affected route: POST /api/orders
```

You can now describe the incident more accurately:

```text
Not all requests
        |
        v
Order creation specifically
        |
        v
Significant customer impact
```

That is more useful than saying, "The API is broken."

During scope assessment, compare:

| Signal | Question |
|---|---|
| Error rate | Are failures isolated to one route, service or region? |
| Traffic | Is the service receiving normal, unusually high or unusually low demand? |
| Latency | Are requests timing out before they return an error? |
| Version | Did the affected requests reach a recent release? |
| Time | Did the incident begin after a deployment, configuration change or dependency event? |
| Data | Are writes failing, partially completing or being duplicated? |

Record the first reliable timestamps. A timeline becomes much more valuable when it is based on evidence rather than memory.

---

## 5. Step 2 - Assess User Impact

Ask:

- Who is affected?
- How many users or requests are affected?
- Which functionality is unavailable or degraded?
- Is the problem limited to one region, tenant or browser flow?
- Is data being lost, duplicated or corrupted?
- Are payments being charged incorrectly?
- Is the problem growing?
- Is there a workaround for users or support staff?

For a MERN e-commerce application:

| Severity example | Impact |
|---|---|
| Low | An internal admin report page is unavailable. |
| Medium | Product search is slow or intermittently degraded. |
| High | Checkout is failing for a significant percentage of customers. |
| Critical | Orders are duplicated, payments are charged incorrectly or customer data is exposed. |

Severity should reflect user and business impact, not only the number shown on a technical dashboard.

A useful incident summary is concise and specific:

> Since 21:04 UTC, approximately 8% of `POST /api/orders` requests are returning HTTP 500. Checkout success has fallen from 99% to 78%. The issue began four minutes after version 10.7 was deployed and appears limited to the new checkout flow.

---

## 6. Step 3 - Mitigate First

During an active incident, the first goal is to reduce user impact safely:

```text
Stop deployment
       |
       v
Disable feature flag or rollback
       |
       v
Route traffic away or scale capacity
       |
       v
Verify impact is reducing
```

Possible mitigation actions include:

- Stop an active rollout.
- Pause promotion from staging to production.
- Disable a broken feature flag.
- Roll back to a known-good application version.
- Route traffic away from an unhealthy instance or region.
- Scale capacity when resource exhaustion is the likely cause.
- Temporarily disable non-critical functionality.
- Apply a documented rate limit or circuit breaker.

For example:

```text
New checkout feature enabled
          |
          v
Checkout failures increase
          |
          v
Disable feature flag
          |
          v
Old checkout flow restored
```

Mitigation is not the same as root-cause analysis. It is acceptable to reduce the blast radius before you fully understand why the incident occurred, as long as the action is deliberate, reversible where possible and recorded.

### Avoid random restarts

A restart may temporarily hide a symptom without fixing the cause. It can also:

- Destroy useful in-memory evidence.
- Remove a process that is still needed for investigation.
- Cause a recovery storm when many instances restart together.
- Hide a configuration or dependency problem until it returns.
- Make the incident timeline harder to interpret.

If a restart is an appropriate mitigation, capture relevant logs and metrics first, record the reason, restart only the necessary scope and verify the result afterward.

---

## 7. Rollback Versus Fix Forward

A common interview question is:

> Would you rollback or fix forward?

There is no universal answer. Choose based on user impact, reversibility, evidence and data compatibility.

### Rollback

Return to the previous known-good application version.

Rollback is especially useful when:

- A new deployment clearly correlates with the failure.
- The previous version is known to work.
- The application can safely run against the current database schema.
- User impact is significant.
- The rollback procedure is tested and fast.

### Fix forward

Deploy a corrected version rather than returning to the old version.

Fix forward may be safer when:

- A database migration has made the old application incompatible.
- The old version would mishandle data already written by the new version.
- The cause is understood and a small, verified correction is available.
- Rollback would create more risk than the fix.
- A security or data-integrity issue requires a forward change.

The decision should be explicit. State the evidence, the expected benefit, the risks and the verification criteria.

---

## 8. Why Database Rollback Can Be Dangerous

Suppose the deployment sequence is:

```text
v1 application
       |
       v
v2 application + database migration
       |
       v
v2 fails
```

Simply returning to `v1` may break the old application if the new schema is incompatible:

```text
Application rollback != Database rollback
```

For example, `v2` may rename a field, remove a column or change a data format that `v1` still expects. This is why backward-compatible database migrations are a production safety mechanism.

A safer migration pattern is often:

1. Add new fields or tables without removing the old structure.
2. Deploy application code that can read both versions.
3. Backfill or migrate data safely.
4. Switch reads and writes to the new structure.
5. Remove old structures only after the old application version is no longer needed.

The exact migration strategy depends on the database and application, but the principle is stable: application rollback should remain possible without requiring an unsafe schema rollback.

---

## 9. Step 4 - Investigate With Evidence

Once user impact is controlled, investigate systematically:

```text
Mitigation
    |
    v
Logs
    |
    v
Metrics
    |
    v
Deployment history
    |
    v
Dependencies
    |
    v
Configuration
    |
    v
Code changes
```

Do not form a conclusion first and then search only for evidence that confirms it. Write down hypotheses and test them one at a time.

### Hypothesis-driven troubleshooting

Incident:

```text
POST /api/orders -> HTTP 500
```

Possible hypotheses:

- **H1:** MongoDB is unavailable.
- **H2:** The payment provider is failing or timing out.
- **H3:** The new application code contains a defect.
- **H4:** An environment variable changed incorrectly.
- **H5:** A network route or firewall rule is blocking a dependency.
- **H6:** A database query became slow and requests are timing out.

For H1, inspect structured logs and dependency metrics:

```text
MongoServerSelectionError
```

MongoDB connectivity is now a stronger hypothesis, but continue checking the actual connection details, network path, service health and deployment configuration before declaring root cause.

A simple investigation table keeps the response disciplined:

| Hypothesis | Evidence to check | Result |
|---|---|---|
| MongoDB unavailable | Connection errors, database health and network reachability | Confirmed, weakened or unknown |
| Payment provider failure | Provider status, timeout logs and dependency latency | Confirmed, weakened or unknown |
| New application bug | Version comparison, stack traces and changed code path | Confirmed, weakened or unknown |
| Invalid configuration | Effective environment values and startup validation | Confirmed, weakened or unknown |
| Network problem | DNS, service name, firewall and connection tests | Confirmed, weakened or unknown |
| Slow query | Query duration, database load and timeout traces | Confirmed, weakened or unknown |

Do not paste secrets into an incident channel while checking configuration. Record the variable name, validation result and safe metadata rather than the credential value.

---

## 10. Correlate With Deployment History

Suppose deployment and monitoring data show:

```text
21:00  Version 10.7 deployed
21:04  5xx rate increases
21:05  Latency increases
```

That is strong temporal correlation. Logs add more context:

```json
{
  "version": "10.7",
  "event": "order_creation_failed",
  "errorType": "MongoServerSelectionError",
  "route": "/api/orders",
  "status": 500
}
```

Now investigate what changed in version 10.7. Possibilities include:

- `MONGO_URI` points to the wrong hostname.
- The API uses a container-local address instead of a service name.
- The MongoDB connection pool is misconfigured.
- A network policy changed during deployment.
- The new code opens connections incorrectly.

Correlation narrows the search; it does not prove causation by itself. Confirm the relationship with logs, configuration, dependency checks and a controlled reproduction where possible.

---

## 11. MERN Incident Example: Orders Cannot Be Created

Production architecture:

```text
React
  |
  v
Nginx
  |
  v
Node / Express API
  |
  v
MongoDB
```

Users report:

> Orders are not being created.

Metrics show:

```text
5xx rate:   7.4%
p95:        2.1s
```

Structured logs show:

```json
{
  "event": "order_creation_failed",
  "requestId": "83ad91",
  "errorType": "MongoServerSelectionError",
  "route": "/api/orders",
  "status": 500
}
```

The production configuration contains:

```text
MONGO_URI=mongodb://localhost:27017/mern
```

The API runs inside Docker, while MongoDB is a separate container. In this situation, `localhost` refers to the API container itself, not the MongoDB container.

A Docker Compose network would normally use the MongoDB service name, for example:

```text
MONGO_URI=mongodb://mongodb:27017/mern
```

The exact hostname depends on the service definition, but the key lesson is that container-to-container communication uses the network-resolvable service name rather than an assumption that every service shares the same localhost.

A reasonable recovery sequence is:

```text
Stop rollout
       |
       v
Rollback if safe
       |
       v
Correct the connection configuration
       |
       v
Validate startup and readiness in staging
       |
       v
Redeploy deliberately
       |
       v
Monitor technical and business metrics
```

Do not call the incident resolved merely because the API process starts. Confirm that MongoDB connectivity, order creation and checkout success all recover.

---

## 12. Step 5 - Verify Recovery

After a rollback, feature-flag change or fix, verify all relevant signals:

```text
5xx rate decreases
       |
       v
Latency returns to normal
       |
       v
Health and readiness pass
       |
       v
Logs no longer show the failure pattern
       |
       v
Business metrics recover
```

For checkout, a useful business signal might be:

```text
Checkout success: 81% -> 99%
```

Verification should include:

- Error rate by endpoint and status code.
- p95 and p99 latency.
- Health and readiness checks.
- Application and dependency logs.
- Queue depth or worker backlog when relevant.
- Checkout, order or payment success rate.
- Duplicate order and payment indicators.
- Synthetic or manual customer-flow checks.
- A short observation period after the change.

### Container running is not recovery

This is a classic DevOps interview trap:

```text
Rollback v10 -> v9
Docker says: Container running
API still returns: 7% 5xx
```

The service is not recovered. Recovery means the user-facing system has returned to an acceptable operational state.

Keep three states separate:

```text
Container running
       !=
Application healthy
       !=
Business healthy
```

A production engineer verifies all three.

---

## 13. Incident Timeline

During the incident, record important events as they happen:

```text
21:00 - Version 10.7 deployed
21:04 - 5xx alert triggered
21:06 - Checkout failures confirmed
21:08 - Rollout stopped
21:10 - Version 10.6 restored
21:13 - 5xx rate normalized
21:20 - Checkout flow verified
```

A timeline helps the team understand:

- What happened.
- When it happened.
- Which signals appeared first.
- Which actions were taken.
- Which actions reduced impact.
- How long detection and recovery took.
- Where the process could improve.

Use UTC consistently and distinguish observed facts from assumptions.

---

## 14. Root Cause Versus Trigger

Suppose the chain is:

```text
Deployment
    |
    v
Bad environment variable
    |
    v
MongoDB connection failure
    |
    v
Order creation fails
```

What is the root cause?

It is probably not simply, "MongoDB was down." The database error is a symptom or immediate failure mode. A more useful root cause may be:

> An incorrect `MONGO_URI` was introduced by deployment configuration, and the pipeline did not validate the effective production connection before promotion.

Use precise language:

| Term | Meaning in this example |
|---|---|
| Trigger | Version 10.7 was deployed. |
| Immediate failure | The API could not select a MongoDB server. |
| User impact | Order creation and checkout failed. |
| Root cause | Invalid production connection configuration was allowed into deployment. |
| Contributing factor | Missing configuration validation and insufficient readiness verification. |

This distinction matters because fixing only the symptom may leave the system vulnerable to recurrence.

---

## 15. Five Whys

A simple RCA technique is the Five Whys:

### Problem

Orders fail.

### Why 1

Why do orders fail? MongoDB connections fail.

### Why 2

Why do MongoDB connections fail? The API uses an incorrect hostname.

### Why 3

Why is the hostname incorrect? Production configuration changed during deployment.

### Why 4

Why was the invalid configuration deployed? The pipeline did not validate the effective production configuration.

### Why 5

Why did the pipeline lack that validation? Configuration validation and production-like readiness checks were not included in the release controls.

The investigation has moved from:

> MongoDB failed.

To:

> Deployment controls allowed invalid configuration into production.

That conclusion leads to actions that improve the system rather than blaming an individual.

---

## 16. Corrective Actions

An RCA should not end with, "A developer made a mistake."

Instead, connect the failure to a system improvement:

```text
Invalid MONGO_URI
        |
        v
Validate required configuration and URI format
        |
        v
Test staging startup against production-like services
        |
        v
Add database readiness checks
        |
        v
Block invalid deployment promotion
        |
        v
Prevent recurrence
```

Useful corrective actions may include:

- Validate required environment variables before startup.
- Validate service hostnames and connection-string formats without exposing secrets.
- Add readiness checks that verify required dependencies.
- Run smoke tests for order creation and checkout in staging.
- Add a canary step before full production rollout.
- Add a feature flag for risky flows.
- Improve alert context with release and request identifiers.
- Document a tested rollback or fix-forward procedure.
- Add backward-compatible database migration rules.
- Review access controls and secret-management practices.

Each action should have an owner, a priority and a way to verify completion.

---

## 17. Interview Questions

### What do you do first during a production incident?

> I first confirm the signal and establish scope and user impact using monitoring, logs and business metrics. I then prioritize a safe mitigation to reduce impact. Once the system is stable, I investigate root cause using deployment history, metrics, logs, configuration and dependency health. Finally, I verify recovery and document preventive actions.

### Why should you not immediately restart a failing production service?

> A restart may temporarily hide the symptom without identifying the cause, and it can destroy useful evidence or create additional instability. I would first assess impact and capture relevant signals. If a restart is an appropriate mitigation, I would perform it deliberately, record the action and verify the result.

### When would you rollback?

> I would consider rollback when a recent deployment clearly caused significant user impact, the previous version is known to be good, and the current database schema and data remain compatible with that version. If schema compatibility or data safety makes rollback risky, I would prefer a verified fix forward.

### What proves recovery?

> Recovery is demonstrated by acceptable user-facing behavior and supporting technical signals: error rates and latency normalize, health and readiness pass, relevant logs return to normal, and business metrics such as checkout success recover. A running container alone does not prove recovery.

---

## 18. Review Questions From Earlier Topics

### Q1 - Centralized logging

**Why centralize logs across multiple MERN API instances?**

Centralization lets engineers search and correlate events across instances from one place, which makes distributed troubleshooting faster and reduces the chance of missing evidence on one replica.

### Q2 - Alerting

**Why should not every error generate a production alert?**

Alerting on every error creates noise and alert fatigue. Alerts should represent meaningful, actionable conditions that require investigation or intervention.

### Q3 - Deployment

**When is canary deployment especially useful?**

Canary deployment is useful when you want to expose a new version gradually and monitor real production behavior before sending all traffic to it.

### Q4 - Feature flags

**A new feature is broken but the underlying application is healthy. What is a fast mitigation?**

Disable the feature flag if the functionality is safely isolated and the old behavior remains available.

### Q5 - Database

**Why can a database migration make rollback dangerous?**

The previous application version may not be compatible with the new schema or data format. Backward-compatible migrations keep mixed versions and safer rollback possible.

---

## 19. Today's Quiz

### Q1

What is the first priority during a production incident?

A. Write the RCA  
B. Reduce user impact  
C. Rewrite the application  
D. Delete logs

### Q2

What is triage?

A. Determining scope, severity and impact  
B. Rebuilding Docker  
C. Creating Git branches  
D. Deleting containers

### Q3

When is rollback especially appropriate?

A. A new deployment clearly caused a serious issue and rollback is safe  
B. Every time CPU increases  
C. Whenever a log appears  
D. Before investigating anything

### Q4

What is a root cause?

A. The underlying reason a failure occurred  
B. Always the last error message  
C. Always a server restart  
D. A dashboard

### Q5

Why document an incident timeline?

A. To understand sequence, decisions and recovery  
B. To replace monitoring  
C. To reduce Docker image size  
D. To delete logs

### Q6

What should happen after rollback?

A. Assume success  
B. Verify metrics, health, logs and user-facing behavior  
C. Immediately deploy again  
D. Disable monitoring

### Q7

What is fix forward?

A. Deploying a corrected version rather than returning to the old version  
B. Restarting MongoDB  
C. Scaling horizontally  
D. Deleting Git history

### Q8

Why are backward-compatible migrations useful?

A. They make mixed application versions and safer rollback easier  
B. They eliminate databases  
C. They prevent logging  
D. They replace Docker

### Q9

What is a good RCA outcome?

A. Blame an individual  
B. Identify system improvements that prevent recurrence  
C. Delete incident data  
D. Ignore the incident

### Q10

What proves recovery?

A. Docker says `running`  
B. User-facing service and relevant health and business metrics return to acceptable levels  
C. Git says merged  
D. CI says passed

### Answer Key

```text
1 -> B
2 -> A
3 -> A
4 -> A
5 -> A
6 -> B
7 -> A
8 -> A
9 -> B
10 -> B
```

### Score

| Score | Result |
|---|---|
| 9-10 | Excellent |
| 7-8 | Good |
| 5-6 | Review incident response |
| Below 5 | Revisit today's lesson |

---

## 20. Practical Exercise - Create an Incident Runbook

Create `INCIDENT-RUNBOOK.md` in your project repository using this structure:

```markdown
# Production Incident Runbook

## 1. Detect
Check:
- Alerts
- Error rate
- Latency
- Availability

## 2. Triage
Determine:
- Affected service
- Affected endpoints
- User impact
- Start time
- Recent deployments

## 3. Mitigate
Possible actions:
- Stop rollout
- Disable feature flag
- Rollback
- Route traffic
- Scale service

## 4. Investigate
Check:
- Logs
- Metrics
- Configuration
- Database
- External dependencies

## 5. Recover
Verify:
- Health
- Readiness
- 5xx rate
- Latency
- Business metrics

## 6. RCA
Document:
- Root cause
- Trigger
- Contributing factors
- Corrective actions
```

Make the runbook operational rather than theoretical. Add the actual dashboard locations, alert names, deployment commands, feature-flag procedure, escalation contacts and rollback prerequisites used by your project. Do not document secrets in the runbook.

### Test the runbook

Walk through the checkout failure scenario from the next section. Confirm that another engineer could follow the document without guessing:

- What signal starts the response?
- How is the impact measured?
- What action reduces impact first?
- How is rollback safety checked?
- Which metrics prove recovery?
- Where is the timeline recorded?
- Who owns each follow-up action?

---

## 21. Practical Assessment - Checkout Failure Scenario

### Scenario

```text
14:00 - Version 12 deployed
14:04 - 5xx rate changes from 0.4% to 6.8%
14:05 - p95 latency changes from 300ms to 1.9s
14:06 - Checkout success changes from 99% to 78%
```

Logs show:

```text
PaymentTimeout
```

The feature-flag system shows:

```text
NEW_PAYMENT_FLOW=true
```

### Recommended response

1. Confirm the alert, affected endpoints and checkout impact.
2. Stop the rollout so the affected version does not spread further.
3. Disable `NEW_PAYMENT_FLOW` if the old payment flow is safe and available.
4. Verify checkout success, error rate and latency recover.
5. If impact continues, consider application rollback after checking database and data compatibility.
6. Investigate payment-provider health, timeout behavior, application changes and configuration.
7. Preserve the incident timeline, logs and deployment evidence.
8. Identify the root cause and contributing factors.
9. Add preventive controls such as payment timeouts, circuit breaking, canary monitoring, feature-flag safeguards or stronger payment-flow tests.

The order matters:

```text
Confirm impact
       |
       v
Stop rollout
       |
       v
Disable NEW_PAYMENT_FLOW
       |
       v
Verify checkout recovery
       |
       v
Consider rollback if needed
       |
       v
Investigate deeply
       |
       v
Prevent recurrence
```

Mitigate first, then investigate deeply.

---

## 22. Monthly Cumulative Project Milestone

Your cumulative MERN DevOps project should now include:

```text
CI/CD
├── Automated tests
├── Docker build
├── Security scan
├── Registry
├── Staging
└── Production promotion

Release safety
├── Rolling, blue-green or canary strategy
├── Health checks
├── Feature flags
└── Rollback

Observability
├── Metrics
├── Structured logs
├── Centralized logging
└── Alerts

Incident response
├── Triage
├── Impact assessment
├── Mitigation
├── Investigation
├── Recovery verification
├── RCA
└── Corrective actions
```

### New deliverable

Create `INCIDENT-RUNBOOK.md` and test it against the checkout failure scenario above.

Your project is becoming something you could discuss in a real DevOps interview:

> I designed the CI/CD pipeline, production deployment strategy, observability, alerting, rollback process and incident-response runbook for a Dockerized MERN application.

---

## 23. Interview Challenge

Answer this aloud:

> A new release causes checkout failures in production. Walk me through exactly what you would do.

A strong answer is:

> First I would confirm the alert and determine the scope and business impact using error rate, latency and checkout-success metrics. I would correlate the issue with the deployment and inspect structured logs for the affected endpoint. If the new checkout flow is behind a feature flag, I would disable it immediately to reduce user impact. If the issue affects the application more broadly, I would stop the rollout or roll back if doing so is safe. Once impact is mitigated, I would investigate the payment service, application changes, configuration, database and network dependencies. I would verify recovery using both technical and business metrics, document the incident timeline, identify the root cause and add preventive controls.

This answer demonstrates:

```text
Triage
  + Mitigation
  + Observability
  + Rollback judgment
  + Root-cause analysis
  + Prevention
```

---

## 24. Day 52 Summary

The most important incident-response model is:

```text
Incident
   |
   v
Detect
   |
   v
Triage
   |
   v
Assess impact
   |
   v
Mitigate
   |
   v
Investigate
   |
   v
Recover
   |
   v
Verify
   |
   v
RCA
   |
   v
Prevent
```

### Remember this interview rule

Mitigation and root-cause analysis are different activities.

During an incident:

```text
Users are suffering
       |
       v
Reduce impact first
       |
       v
Then investigate deeply
```

And do not confuse:

```text
Container running
       !=
Application healthy
       !=
Business healthy
```

A production engineer verifies all three.

---

## 25. Next Lesson

Day 53 will connect incident response to infrastructure and application reliability:

```text
Health checks
       |
       v
Readiness
       |
       v
Auto-healing
       |
       v
Restart policies
       |
       v
Horizontal scaling
       |
       v
Load balancing
```

These mechanisms help production systems detect unhealthy instances, stop sending traffic to them, recover automatically when appropriate and handle changing demand safely.
