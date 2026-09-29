# Production Incident Runbook

This runbook is for customer-facing MERN production incidents. Adapt commands, dashboard links, escalation contacts and deployment procedures to the project. Never store secrets in this file.

## Incident Header

Record these details at the beginning of the incident:

```text
Incident ID:
Incident commander:
Technical lead:
Communications owner:
Start time UTC:
Detection source:
Affected service:
Current severity:
Incident channel:
Dashboard:
Runbook version:
```

## 1. Detect

Check:

- Production alerts.
- Error rate by service, route and status code.
- p50, p95 and p99 latency.
- Availability, health and readiness.
- Checkout, payment and order-success metrics.
- Customer reports and support tickets.
- Recent deployment and configuration events.

Do not change production before capturing enough evidence to understand the initial state, unless immediate action is required to prevent serious harm.

## 2. Triage

Determine:

- Affected service and endpoints.
- Start time and current trend.
- Number or percentage of affected users.
- Regions, tenants or customer segments affected.
- Whether data is lost, duplicated or corrupted.
- Whether payments, authentication or sensitive data are involved.
- Current application version and feature flags.
- Whether the issue is isolated or expanding.

Write a concise incident summary:

```text
Since [time], [percentage/count] of [operation] is failing for [users/region].
The issue began after [change] and is currently [increasing/stable/decreasing].

Current impact: [business impact]
Current mitigation: [action or none]
Next decision: [decision]
```

## 3. Mitigate

Use the least risky action that reduces user impact:

- Stop or pause the rollout.
- Disable an isolated feature flag.
- Route traffic away from an unhealthy instance or region.
- Scale capacity when saturation is the likely cause.
- Use an approved fallback dependency or workflow.
- Roll back the application when the previous version is known-good and schema-compatible.
- Disable non-critical functionality.

Before rollback, confirm:

- The previous application version is available.
- The current schema and data are compatible with it.
- The rollback procedure is tested.
- The rollback will not duplicate payments or orders.
- A verification plan is ready.

Record the action, reason, owner and timestamp. Mitigation is not proof of root cause.

## 4. Investigate

Use evidence and test hypotheses one at a time.

### Logs

Search by:

- `requestId`.
- `service` and `environment`.
- `version`.
- `route` and `status`.
- `event` and `errorType`.
- Dependency name and timeout classification.

### Metrics

Compare:

- Error rates by route and version.
- Latency percentiles.
- Request volume.
- Container restarts and healthy instance count.
- CPU, memory and connection-pool utilization.
- MongoDB health and query latency.
- Payment-provider response rate and latency.
- Checkout and order-success rates.

### Changes

Review:

- Application deployments.
- Environment-variable changes.
- Feature-flag changes.
- Database migrations.
- Network, DNS and firewall changes.
- External dependency incidents.

### MERN checks

For MongoDB connectivity failures, verify the hostname, DNS, network path, credentials, TLS configuration and database health without exposing the full URI.

For Docker Compose, remember that `localhost` inside the API container refers to the API container. MongoDB is normally reached through its service name on the shared network.

For payment timeouts, verify provider status, configured timeout, retry behavior, idempotency keys and whether an order or charge partially completed.

## 5. Recover

Apply the selected mitigation or fix through the approved deployment process. Then verify:

- Health checks pass.
- Readiness checks pass continuously.
- 5xx rate returns to the accepted baseline.
- Latency returns to the accepted baseline.
- Relevant error events stop or return to normal background levels.
- Dependency connections remain healthy.
- Checkout and payment success recover.
- No duplicate orders, duplicate charges or data corruption appears.
- A representative customer flow succeeds safely.

A container in the `running` state is not enough. Confirm container, application and business health separately.

## 6. Incident Timeline

Record UTC timestamps as the incident progresses:

```text
[time] - Alert or report received
[time] - Scope and customer impact confirmed
[time] - Deployment/configuration change correlated
[time] - Rollout stopped or mitigation applied
[time] - Error rate and business metric checked
[time] - Recovery verified
[time] - Incident downgraded or resolved
```

Include decisions and their reasons, not only commands.

## 7. RCA

After service is stable, document:

- Customer and business impact.
- Trigger.
- Immediate failure or symptom.
- Root cause.
- Contributing factors.
- Why detection and mitigation took the time they did.
- What worked.
- What caused confusion or delay.
- Corrective actions.

Use Five Whys when helpful. Avoid assigning the root cause to an individual mistake. Identify missing validation, unsafe defaults, unclear ownership, insufficient testing or weak operational controls.

## 8. Corrective Action Register

| Action | Owner | Priority | Due date | Verification |
|---|---|---|---|---|
| Add production configuration validation |  | High |  | Invalid values block promotion |
| Add checkout smoke test in staging |  | High |  | Test fails safely when payment flow breaks |
| Add feature-flag rollback procedure |  | Medium |  | On-call can disable and verify fallback |
| Add release/version fields to alerts |  | Medium |  | Alert identifies affected deployment |
| Review payment idempotency and retry behavior |  | High |  | No duplicate orders or charges |

## Checkout Failure Drill

For a new release causing checkout failures:

1. Confirm 5xx, latency and checkout-success impact.
2. Stop the rollout.
3. Disable the isolated new checkout or payment feature if the fallback is safe.
4. Verify checkout recovery.
5. Consider application rollback if impact continues and compatibility is confirmed.
6. Investigate payment integration, configuration, network and application changes.
7. Preserve logs, metrics, request IDs and the deployment timeline.
8. Complete the RCA and corrective-action register.
