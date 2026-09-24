# Day 47 - Release Safety Checklist

Use this checklist before, during and after a production release.

## Before deployment

- [ ] Immutable image tag and digest recorded.
- [ ] Previous production image remains available.
- [ ] Staging smoke tests passed.
- [ ] Database migration is backward compatible.
- [ ] Feature flag has a safe default.
- [ ] Flag owner and expiry date are documented.
- [ ] Rollback commands or controller action are tested.
- [ ] Monitoring dashboards and alerts are ready.
- [ ] Release owner and on-call engineer are identified.

## During deployment

- [ ] v2 readiness check passed.
- [ ] Feature remains disabled during infrastructure validation.
- [ ] Canary traffic is within the approved percentage.
- [ ] v1 remains available.
- [ ] Error rate is compared by version.
- [ ] Latency is compared by version.
- [ ] Database and dependency metrics are healthy.
- [ ] Business metrics are being observed.
- [ ] No rollout stage is increased without evidence.

## During feature release

- [ ] Internal-user validation passed.
- [ ] Checkout success rate is acceptable.
- [ ] Payment failures are within baseline.
- [ ] Order creation is correct.
- [ ] No duplicate orders or data-integrity alerts exist.
- [ ] Flag change is recorded in the audit trail.

## Recovery decision

- [ ] Is the problem isolated to a flagged feature?
- [ ] If yes, disable the flag and verify recovery.
- [ ] Does the problem affect the broader application?
- [ ] If yes, stop exposure and restore the previous artifact.
- [ ] Is infrastructure or configuration the cause?
- [ ] If yes, recover that layer separately.
- [ ] Has evidence been preserved?
- [ ] Has the incident owner been notified?

## After release

- [ ] Full-exposure metrics remain healthy.
- [ ] Rollback artifact is still available.
- [ ] Temporary flag cleanup is scheduled.
- [ ] Release record is complete.
- [ ] Follow-up investigation or post-incident work is assigned.
