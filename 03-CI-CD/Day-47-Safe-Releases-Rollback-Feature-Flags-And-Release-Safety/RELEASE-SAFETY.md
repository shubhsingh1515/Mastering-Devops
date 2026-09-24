# Release Safety - Day 47

## Purpose

This document defines the safe release process for a Dockerized MERN application. It separates deployment from feature release, limits blast radius and provides a tested recovery path.

## Deployment strategy

Use a canary or gradual rollout with the previous version available:

```text
Build once -> Scan -> Registry -> Staging -> Smoke tests
    |
    v
Deploy v2 with feature OFF
    |
    v
Internal users -> 1% -> 10% -> 50% -> 100%
```

Do not increase exposure unless the current stage meets its release criteria.

## Initial and target versions

```text
Previous known-good: registry.example.com/mern-api:2.3.0
Target release:       registry.example.com/mern-api:2.4.0
```

Record the image digest used in production. Do not rebuild the image during deployment or rollback.

## Feature-flag strategy

```text
NEW_CHECKOUT_ENABLED=false
```

Rules:

- The new feature is disabled by default.
- Enable it first for internal users or a controlled cohort.
- Change exposure only through an authorized operation.
- Record the actor, time, old value, new value and reason.
- Define a safe behavior if the flag service is unavailable.
- Assign an owner and removal date to the flag.
- Remove the flag after the old behavior is no longer needed.

A feature flag is a runtime release control, not a replacement for automated testing.

## Health checks

The candidate must pass these checks before receiving traffic:

```text
GET /health
GET /ready
```

`/health` confirms that the process is alive. `/ready` confirms that the service is ready to serve requests. Neither endpoint alone proves that checkout or payment behavior is correct.

## Smoke tests

Run safe tests after deployment and after each exposure increase:

```text
- Health endpoint
- Readiness endpoint
- Critical read API request
- Authenticated test request
- Checkout test using a non-production payment provider
- Order creation verification
```

Never use real customer payment data in a smoke test.

## Progressive rollout stages

### Stage 0: Validate the artifact

- Build and scan the image once.
- Push an immutable tag and digest.
- Run tests and migration checks in staging.
- Confirm the previous artifact remains available.

### Stage 1: Deploy with the feature disabled

- Deploy v2 with `NEW_CHECKOUT_ENABLED=false`.
- Route only approved canary traffic to v2.
- Check readiness, errors, latency, resources and database behavior.

### Stage 2: Internal release

- Enable checkout for internal users.
- Test success, payment failure, cancellation and retry paths.
- Confirm users outside the group still receive the old path.

### Stage 3: Gradual exposure

- Increase from 1% to 10%, then 50%, then 100% only when evidence is healthy.
- Keep v1 available throughout the observation period.
- Stop immediately when a critical signal crosses its threshold.

## Monitoring metrics

### Technical metrics

- 4xx and 5xx rate by release version
- p50, p95 and p99 latency
- Availability and request volume
- CPU, memory and container restarts
- MongoDB query latency and connection errors
- External payment-provider errors
- Logs, traces and active alerts

### Business metrics

- Checkout success rate
- Payment failure rate
- Order creation failures
- Duplicate order reports
- Cart abandonment
- Refunds and cancellations

Compare v2 against the v1 baseline. A green container and a passing health endpoint do not prove business success.

## Rollback thresholds

Stop the rollout and investigate when any of the following is true:

```text
- v2 5xx rate is above 5% for 3 consecutive minutes
- p95 latency is materially above the approved baseline
- Checkout success falls below the agreed business threshold
- Payment failures or duplicate orders increase unexpectedly
- Health, readiness or smoke tests fail
- Database errors or connection saturation appear
- A critical alert fires
- Data integrity is uncertain
```

Apply thresholds only with an appropriate observation window and minimum request volume. Avoid reacting to one isolated request or a low-volume percentage.

## Feature rollback procedure

Use this when the failure is isolated to the flagged feature:

```text
1. Stop increasing exposure.
2. Set NEW_CHECKOUT_ENABLED=false for the affected audience.
3. Confirm the effective flag value.
4. Verify the old checkout path with a safe test.
5. Confirm checkout and payment metrics recover.
6. Preserve logs, traces and flag audit records.
7. Keep v2 running only if unrelated behavior is healthy.
```

This recovery path does not require rebuilding the Docker image.

## Application rollback procedure

Use this when v2 is broadly unsafe or the flag cannot contain the failure:

```text
1. Stop the rollout.
2. Remove v2 from the traffic pool or reduce exposure to zero.
3. Route traffic to registry.example.com/mern-api:2.3.0.
4. Run health and smoke tests.
5. Confirm technical and business recovery.
6. Notify the incident owner.
7. Preserve evidence and investigate before redeploying.
```

Do not delete the previous production image during an incident.

## Infrastructure recovery

If configuration, networking, secrets, capacity or another platform dependency is the cause:

```text
1. Identify the affected infrastructure component.
2. Restore the last known-good configuration or capacity.
3. Validate connectivity and readiness.
4. Run smoke tests.
5. Decide separately whether the application image also needs rollback.
```

Application rollback and infrastructure recovery are separate decisions.

## Database compatibility strategy

Use expand-and-contract migrations:

```text
1. Add new fields or indexes without removing old ones.
2. Deploy code that can read and write both representations.
3. Backfill existing data.
4. Switch normal reads and writes to the new representation.
5. Verify that old application versions are no longer needed.
6. Remove old fields in a later release.
```

For example, keep both `users.name` and `users.display_name` while v1 and v2 may coexist. Removing `users.name` first can make a v1 application rollback fail.

## Automated rollback policy

Automation may stop a release when the failure is clear and measurable:

```text
IF v2 5xx rate > 5%
FOR 3 consecutive minutes
AND request volume exceeds the minimum sample size
THEN stop promotion, notify on-call and evaluate rollback
```

Use cooldowns and escalation to avoid rollback flapping. Require human investigation for data-integrity issues, destructive migrations, external-provider failures and ambiguous business metric changes.

## Release record

```text
Release owner: ____________________
On-call engineer: _________________
Previous version: _________________
Target version: ___________________
Image digest: _____________________
Flag owner: _______________________
Observation window: _______________
Final decision: ___________________
Rollback used: yes / no
Incident reference: _______________
```

## Completion criteria

The release is complete when:

- the intended users receive the intended behavior
- health and readiness remain healthy
- error and latency metrics remain within thresholds
- checkout and order metrics meet the approved baseline
- no critical alerts remain open
- the previous artifact remains available for the agreed retention period
- the feature flag has an owner and cleanup date
- the release record is complete
