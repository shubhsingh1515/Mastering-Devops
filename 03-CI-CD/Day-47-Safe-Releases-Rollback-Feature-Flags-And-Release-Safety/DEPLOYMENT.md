# Release Safety Deployment Plan - Day 47

## Release objective

Release `mern-api:v2` safely while keeping the previous known-good artifact available and allowing the new checkout feature to be controlled independently.

## Deployment strategy

Use a canary or gradual rollout behind a feature flag.

```text
v2 deployed with checkout OFF
        |
        v
Infrastructure and smoke validation
        |
        v
Internal users receive checkout
        |
        v
1% -> 10% -> 50% -> 100%
```

The exact percentages may change with traffic volume, but every stage requires an observation window and release decision.

## Initial version

```text
registry.example.com/mern-api:2.3.0
```

Keep this immutable image available until the new release has completed its observation period and rollback risk has been reviewed.

## Target version

```text
registry.example.com/mern-api:2.4.0
```

Do not rebuild the artifact during deployment. Production should use the same image digest that passed CI, security scanning and staging validation.

## Feature-flag strategy

Flag:

```text
NEW_CHECKOUT_ENABLED=false
```

Rules:

- Default to `false` for new environments and new users.
- Enable first for internal users or an approved test cohort.
- Increase exposure only after technical and business signals remain healthy.
- Record actor, timestamp, previous value, new value and reason for every change.
- Restrict flag changes to authorized operators.
- Define behavior if the flag service is unavailable.
- Remove the flag after the feature is stable and the old path is retired.

## Health checks

Required endpoints:

```text
GET /health
GET /ready
```

The candidate must pass readiness checks before receiving production traffic. Readiness should represent the ability to serve requests, not only whether the process is running.

## Smoke tests

Run safe checks after deployment and after each traffic change:

```text
- GET /health
- GET /ready
- GET /api/products
- authenticated test request
- test checkout with a non-production payment provider
```

Smoke tests must not use real customer payments or destructive production data.

## Rollout stages

### Stage 0 - Build and validate

- Build the image once.
- Scan the image.
- Push it with an immutable version tag and digest.
- Deploy to staging.
- Run smoke tests and migration checks.

### Stage 1 - Production with feature disabled

- Deploy v2 with `NEW_CHECKOUT_ENABLED=false`.
- Route only the planned canary traffic to v2.
- Confirm health, readiness, error rate, latency and database behavior.

### Stage 2 - Internal users

- Enable checkout for internal users only.
- Test payment, order creation, cancellation and failure paths.
- Confirm that old users still receive the existing behavior.

### Stage 3 - Small canary

- Enable checkout for approximately 1% of eligible traffic.
- Observe for the defined window.
- Compare v2 with the v1 baseline.

### Stage 4 - Larger exposure

- Increase to 10%, then 50% only when release criteria remain healthy.
- Keep v1 available until the final observation period ends.

### Stage 5 - Full release

- Enable for 100% of the approved audience.
- Continue monitoring after promotion.
- Schedule removal of the temporary flag and old code path.

## Monitoring metrics

Technical metrics:

- 4xx and 5xx error rate by version
- p50, p95 and p99 latency by version
- availability and request volume
- CPU and memory usage
- container restarts
- MongoDB query latency and errors
- payment-provider dependency errors
- logs, traces and alert state

Business metrics:

- checkout success rate
- payment authorization failure rate
- order creation failure rate
- duplicate order reports
- cart abandonment
- refunds and cancellations

Always compare v2 with the known-good v1 baseline and use a meaningful observation window.

## Rollback thresholds

Stop increasing exposure when any of these occurs:

```text
- v2 5xx rate exceeds 5% for 3 consecutive minutes
- p95 latency is materially above the approved baseline
- checkout success drops below the approved business threshold
- payment failures or duplicate orders increase unexpectedly
- readiness or smoke tests fail
- database errors or connection saturation appear
- critical alerts fire
- data integrity is uncertain
```

Thresholds must be reviewed against normal traffic volume. A single failed request or a low-volume percentage can be misleading.

## Feature rollback procedure

Use this first when the defect is isolated to checkout:

```text
1. Stop increasing traffic.
2. Set NEW_CHECKOUT_ENABLED=false for the affected audience.
3. Confirm the effective flag value.
4. Verify the old checkout path with a safe test.
5. Confirm checkout success and error metrics recover.
6. Preserve logs and flag-change audit data.
7. Keep v2 running only if unrelated functionality remains healthy.
```

## Application rollback procedure

Use this when v2 is broadly unsafe or the feature flag cannot contain the failure:

```text
1. Stop the rollout and freeze further promotion.
2. Remove v2 from the traffic pool or reduce it to zero.
3. Route traffic to mern-api:2.3.0.
4. Run health and smoke tests.
5. Confirm technical and business recovery.
6. Notify the incident owner and preserve evidence.
7. Investigate before attempting a new deployment.
```

Do not delete the previous artifact during an incident.

## Infrastructure recovery procedure

If the cause is configuration, networking, secrets, platform capacity or another infrastructure layer:

```text
1. Identify the affected infrastructure component.
2. Restore the last known-good configuration or capacity.
3. Validate connectivity and readiness.
4. Re-run smoke tests.
5. Decide separately whether the application image also needs rollback.
```

An application rollback cannot repair every infrastructure failure.

## Database compatibility strategy

Use expand-and-contract migrations:

```text
1. Add new fields or indexes without removing old ones.
2. Deploy code that reads and writes both representations.
3. Backfill existing records.
4. Switch normal behavior to the new representation.
5. Verify old application versions are no longer required.
6. Remove old fields in a later controlled release.
```

For example, keep both `users.name` and `users.display_name` while v1 and v2 may coexist. Removing `users.name` before v1 is retired can make an application rollback fail.

## Approvals and incident contacts

Before production promotion, identify:

```text
Release owner: ____________________
On-call engineer: _________________
Product owner: ____________________
Database owner: ___________________
Payment integration owner: ________
Approval recorded at: _____________
```

High-risk schema changes, payment changes and ambiguous business-metric failures require human review.

## Release record

```text
Release version: __________________
Image digest: _____________________
Previous version: _________________
Flag owner: _______________________
Start time: _______________________
Canary percentage: ________________
Observation window: _______________
Final decision: ___________________
Rollback used: yes / no
Incident reference: _______________
```

## Completion criteria

The release is complete only when:

- all planned users receive the intended behavior
- health and readiness remain healthy
- error and latency metrics are within thresholds
- checkout and order metrics meet the approved baseline
- no critical alerts remain open
- rollback artifacts and records are still available
- the temporary flag has an owner and cleanup date
