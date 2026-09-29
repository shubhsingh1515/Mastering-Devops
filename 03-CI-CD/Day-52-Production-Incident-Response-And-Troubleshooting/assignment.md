# Day 52 Assignment: Build an Incident Response Workflow

## Objective

Create a practical incident-response workflow for a production MERN application. The workflow must help responders reduce user impact, preserve evidence, investigate with hypotheses and verify recovery.

## Scenario

Your production system contains Nginx, three Node/Express API replicas, a worker and MongoDB. All services run as containers. A new release has caused checkout failures.

```text
14:00 - v12 deployed
14:04 - 5xx rate: 0.4% -> 6.8%
14:05 - p95 latency: 300ms -> 1.9s
14:06 - checkout success: 99% -> 78%
```

Logs show:

```text
errorType=PaymentTimeout
```

The feature-flag system shows:

```text
NEW_PAYMENT_FLOW=true
```

## Required Deliverables

1. Create `INCIDENT-RUNBOOK.md` with detect, triage, mitigate, investigate, recover and RCA sections.
2. Define five incident alerts in `ALERTS.md`.
3. Write the response sequence for the checkout scenario.
4. Record the evidence that would confirm recovery.
5. Document at least three preventive corrective actions.

## Incident Response Questions

Answer the following in your notes:

1. What is the difference between detection, triage, mitigation and root-cause analysis?
2. Which user and business metrics determine incident severity?
3. What should be checked before rolling back an application release?
4. Why can a database migration make rollback unsafe?
5. When is disabling a feature flag safer than rolling back the application?
6. Why can a random restart destroy useful evidence?
7. Which hypotheses explain a `PaymentTimeout`?
8. What technical and business signals prove recovery?
9. What is the difference between a trigger, symptom, root cause and contributing factor?
10. Which pipeline control would prevent this incident from recurring?

## Incident Drill

Write your response in this order:

1. Confirm the alert and establish the affected scope.
2. Measure checkout and payment impact.
3. Stop the rollout.
4. Disable `NEW_PAYMENT_FLOW` if the fallback path is safe.
5. Verify checkout recovery.
6. Consider application rollback if the issue continues and compatibility is confirmed.
7. Investigate payment-provider health, timeout behavior, application changes, configuration and network dependencies.
8. Preserve logs, metrics, deployment metadata and the incident timeline.
9. Identify root cause and contributing factors.
10. Add preventive controls and assign owners.

## Completion Criteria

- The response mitigates impact before deep investigation.
- The runbook identifies a responder, escalation path and timeline location.
- Alerts include actionable thresholds, duration and volume context.
- Rollback safety includes database-schema compatibility.
- Feature-flag mitigation is considered before a broad rollback.
- Recovery includes technical and business metrics.
- Sensitive values are not copied into incident notes.
- Corrective actions improve the system rather than assigning blame.
