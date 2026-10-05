# MERN Reliability Objectives

## Service

MERN Production API

## Objective 1 — Availability

- **Target:** At least 99.9% successful requests over a rolling 30-day window.
- **SLI:** `successful requests / total eligible requests`.
- **Eligible requests:** Requests to production API routes that are expected to return a response; exclude intentionally rejected invalid input only if that exclusion is consistently implemented and documented.
- **Data source:** Application request metrics and load-balancer access logs.
- **Error budget:** 0.1% of eligible requests.

## Objective 2 — Latency

- **Target:** p95 latency below 500 ms for read APIs over a rolling 30-day window.
- **SLI:** The 95th percentile of server-side request duration.
- **Data source:** Request-duration histogram, grouped by route and status class.
- **Guardrail:** Track p99 separately so a good p95 does not hide severe tail latency.

## Objective 3 — Error rate

- **Target:** HTTP 5xx responses below 0.5% over a rolling 30-day window.
- **SLI:** `5xx responses / total requests`.
- **Data source:** API metrics and structured logs.
- **Alert:** Page when the short-window rate is high enough to burn the monthly budget rapidly.

## Objective 4 — Checkout success

- **Target:** At least 99.5% of valid checkout attempts create an order successfully.
- **SLI:** `successful order creations / valid checkout attempts`.
- **Data source:** Checkout service events, order records and payment outcome events.
- **Important distinction:** A healthy API response is not sufficient if payment or order creation failed afterward.

## Error-budget policy

For a 99.9% availability SLO:

```text
failure allowance = 100% - 99.9% = 0.1%
```

If there are 1,000,000 eligible requests:

```text
monthly request budget = 1,000,000 × 0.001 = 1,000 failed requests
```

When more than 50% of the budget is consumed, require additional review for risky releases. When a canary causes rapid burn or a clear SLO regression, stop the rollout and investigate or roll back.

## Alert policy

Every alert must identify the SLO, affected scope, time window and action:

| Severity | Example condition | Action |
|---|---|---|
| Warning | More than 50% of budget consumed before the midpoint of the window | Review changes and reduce release risk |
| High | Canary 5xx rate materially exceeds stable | Stop rollout and compare logs, dependencies and configuration |
| Critical | Sustained rapid burn threatens to exhaust the budget | Roll back or mitigate immediately; open an incident |

## Deployment policy

1. Run tests, build and security checks.
2. Deploy a small canary.
3. Compare canary and stable availability, p95/p99 latency, 5xx rate and checkout success.
4. Stop the rollout when a user-impacting SLO regresses beyond the agreed threshold.
5. Preserve deployment ID, version, configuration, logs and metric screenshots or queries.
6. Roll back or fix forward only after the failure mode is understood.
7. Resume traffic gradually after recovery is verified against the same measurements.

## Recovery process

During an incident, mitigate user impact first, keep evidence intact, identify the change or dependency that introduced the regression, and document the follow-up work. An error budget is a decision tool, not a reason to hide failures or disable monitoring.
