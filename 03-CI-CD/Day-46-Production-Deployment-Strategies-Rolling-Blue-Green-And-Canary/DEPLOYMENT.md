# Deployment Strategy - Day 46

## Deployment strategy

Rolling deployment

## Initial version

v1

## Target version

v2

## Health check

```text
/health
```

The new instance must pass the health endpoint before receiving production traffic.

## Rollback plan

```text
If v2 fails health checks or shows unacceptable error rates, stop the rollout.
Return traffic to v1.
Keep the previous version running until the problem is investigated.
```

## Database compatibility strategy

- Keep schema changes backward compatible.
- Add new fields before removing old ones.
- Deploy compatible application versions first.
- Migrate data gradually.
- Remove deprecated fields only after all versions are safely updated.

## Monitoring during deployment

Monitor:

- HTTP error rate
- latency
- CPU and memory usage
- health endpoint responses
- smoke test results
- database connection errors

## Failure criteria

Stop rollout immediately if any of the following occurs:

- health checks fail
- error rate exceeds the threshold
- latency spikes beyond the acceptable level
- smoke tests fail
- database compatibility issue appears
- application crashes or restarts repeatedly

## Practical rollout sequence

```text
1. Start v2 instance
2. Run /health check
3. If healthy, add v2 to traffic
4. Remove one v1 instance
5. Repeat until all instances are v2
6. Monitor all critical metrics during the rollout
7. Stop the rollout if health degrades
```

## Summary

This strategy keeps production available while reducing risk. The service remains partially on the older version while the new version is introduced. The key to success is health validation, careful monitoring, and a fast rollback path.
