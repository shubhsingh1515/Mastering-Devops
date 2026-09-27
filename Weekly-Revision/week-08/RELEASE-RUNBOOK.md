# MERN Production Release Runbook


## Purpose

This runbook describes how to build, validate, release, observe and recover a Dockerized MERN application. It is designed so another engineer can execute a release without relying on undocumented knowledge.

## 1. Build Process

1. Merge or push the approved commit to the protected branch.
2. GitHub Actions runs `npm ci` using the lockfile.
3. Run linting and unit/integration tests.
4. Build the frontend and backend.
5. Build one Docker image from the tested commit.
6. Scan the image for known vulnerabilities.
7. Publish the immutable image to the registry.

Example artifact:

```text
registry.example.com/mern-api:<git-sha>
```

A failed required check blocks image publication or promotion.

## 2. Image Naming and Tagging

Use the Git commit SHA as the deployment tag:

```text
mern-api:9f72c1
```

Optionally add a human-readable release tag:

```text
mern-api:release-2026-09-27
```

The SHA or image digest is the source of truth. Do not use `latest` as the only production identifier.

Record the following for every release:

- Git commit SHA.
- Image tag and digest.
- Build workflow URL.
- Image scan result.
- Release approver.
- Deployment time.

## 3. Registry Location

Configure the registry before release:

```text
Registry: <registry-url>
Repository: <repository-name>
Image: <registry-url>/<repository-name>:<git-sha>
```

Confirm that production can pull the exact digest and that registry credentials use the minimum required permissions.

## 4. Staging Deployment

Deploy the exact image that passed CI:

```text
Image: <registry-url>/<repository-name>:<git-sha>
Environment: staging
```

Do not rebuild the image for staging. Confirm:

- The expected environment variables are configured.
- MongoDB and external dependencies are reachable.
- Secrets are injected by the approved secret-management process.
- The deployed version reports the expected commit SHA.
- Logs are being collected.

## 5. Health and Readiness Check

A running container is not enough. Verify the readiness endpoint:

```bash
curl --fail --show-error https://<staging-host>/health/ready
```

Expected result:

```text
HTTP 200
```

A failed readiness check blocks promotion. Confirm application startup, database connectivity and other critical dependencies without making the health endpoint unnecessarily expensive.

## 6. Smoke Tests

Run the smallest set of production-like user-flow checks:

```text
1. Application responds.
2. User authentication flow works.
3. Product listing works.
4. Order or checkout flow reaches the expected safe state.
5. Health and readiness remain successful.
6. No unexpected 5xx errors appear in logs.
```

A failed smoke test blocks production promotion. Record the test result and artifact version.

## 7. Production Deployment Strategy

**Selected strategy:** Replace with rolling, blue-green or canary.

### Rolling

Replace instances gradually while keeping sufficient healthy capacity.

### Blue-green

Deploy the new version beside the old version, validate it and switch traffic. Keep the previous environment available for rapid reversal.

### Canary

Start with a small percentage of traffic, for example:

```text
95% -> current version
 5% -> new version
```

Increase exposure only when error rate, latency, logs, traces and business signals remain healthy.

## 8. Release Metrics

Monitor the release against the previous baseline:

- 5xx error rate.
- p50, p95 and p99 latency.
- Request rate.
- Container restarts.
- CPU and memory.
- MongoDB connection and query health.
- External-provider errors and timeouts.
- Login, checkout or other critical business success rate.
- Structured error events grouped by `errorType`.

Normal CPU does not prove that a release is healthy. Business signals and dependency behavior are required.

## 9. Rollback Procedure

Trigger rollback when user impact is significant, the release fails health checks, a canary regresses or the incident cannot be mitigated quickly.

```text
1. Pause the rollout.
2. Confirm the last known-good image tag or digest.
3. Route traffic back to the previous version.
4. Verify health and smoke tests.
5. Confirm error rate and business signals recover.
6. Preserve logs, metrics and release evidence.
7. Notify stakeholders.
8. Open a root-cause investigation.
```

Rollback target:

```text
Previous image: <registry-url>/<repository-name>:<known-good-sha>
```

Do not rebuild the previous version during the incident.

## 10. Feature-Flag Procedure

Use a feature flag when the failure is isolated behind a safely reversible feature flag:

```text
1. Confirm the affected feature and blast radius.
2. Disable the flag using the approved control plane.
3. Verify the previous behavior with a smoke test.
4. Check error rate, latency and business success metrics.
5. Record who changed the flag and why.
6. Create follow-up work to fix or remove the broken path.
```

If the failure affects shared infrastructure or multiple unrelated flows, use the release rollback or incident procedure instead.

## 11. Incident Investigation Steps

When the 5xx rate rises:

```text
1. Confirm scope, severity and time window.
2. Check the deployment and configuration timeline.
3. Identify affected routes, versions and instances.
4. Search structured logs for status=500.
5. Group events by error type and dependency.
6. Select representative request IDs.
7. Correlate Nginx, API, worker, database and provider logs.
8. Inspect traces for slow or failed spans.
9. Determine the blast radius and business impact.
10. Mitigate with a flag disable, rollout pause or rollback.
11. Preserve evidence and document the timeline.
12. Complete root-cause analysis after recovery.
```

Never log or share passwords, JWTs, API keys, database credentials, authorization headers, credit-card data or unnecessary personal information while investigating.

## Release Decision Checklist

- [ ] CI passed.
- [ ] Tests passed.
- [ ] Docker image built once.
- [ ] Image scan completed.
- [ ] Immutable tag or digest recorded.
- [ ] Staging uses the exact production artifact.
- [ ] Readiness check passed.
- [ ] Smoke tests passed.
- [ ] Deployment strategy selected.
- [ ] Rollback target identified.
- [ ] Feature-flag recovery path understood.
- [ ] Metrics and business signals are visible.
- [ ] Structured logs and request IDs are searchable.
- [ ] Release owner and approver are known.

## Release Completion

A release is complete only when the new version is healthy after deployment and the evidence is recorded:

```text
Artifact -> Staging -> Health -> Smoke -> Production -> Observe -> Verify
```

Store the release version, deployment time, verification result and any follow-up actions with the change record.
