# Day 51 Assignment: Design a MERN Alert Set

## Objective

Design a centralized logging and alerting approach for a production MERN application. Your design must help a responder detect important failures, investigate them quickly and avoid unnecessary noise.

## Scenario

Your production system contains Nginx, three Node/Express API replicas, a worker and MongoDB. All services run as containers. A customer reports that checkout failed after a deployment.

## Required Deliverable

Create or update `ALERTS.md` in your cumulative MERN project. Define these alerts:

1. API 5xx rate.
2. API p95 latency.
3. Health/readiness failures.
4. MongoDB connectivity.
5. Checkout success rate.

For each alert, document:

- Name.
- Condition and time window.
- Severity.
- Minimum traffic or event-volume guard, if applicable.
- Who responds.
- Initial investigation.
- Possible recovery.
- Dashboard or runbook link placeholder.

## Logging Design Questions

Answer the following in your notes:

1. Where do Node.js containers emit logs?
2. What fields are required in a request-completed event?
3. What fields are required in an order-failure event?
4. How does a request ID connect Nginx, Express and dependency logs?
5. Which values must never be logged?
6. Which logs are hot, and which can be archived or sampled?
7. How does your design avoid alert fatigue?

## Incident Drill

You receive:

```text
ALERT: Production API 5xx > 5%
5xx: 8.1%
p95: 2.1s
CPU: 43%
Memory: 51%
Traffic: normal
version: 10a7c1
```

Logs show:

```text
event=order_creation_failed
errorType=MongoServerSelectionError
version=10a7c1
```

The version was deployed seven minutes ago. Write your reasoning and investigation order. Include:

- Confirming scope and customer impact.
- Checking the deployment timeline.
- Searching by status, event and error type.
- Following representative request IDs.
- Checking MongoDB hostname, DNS, credentials and network access.
- Stopping the rollout or rolling back when appropriate.
- Validating the recovery after mitigation.

## Completion Criteria

- All five alerts have actionable conditions.
- Thresholds include duration and volume context where needed.
- Every alert names a responder and first investigation step.
- Logs are structured and sent to stdout/stderr.
- Secrets and sensitive data are excluded.
- Retention and sampling are explained.
- The incident procedure prioritizes mitigation without destroying evidence.