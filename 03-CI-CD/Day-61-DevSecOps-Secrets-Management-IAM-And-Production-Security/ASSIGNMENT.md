# Day 61 Assignment — Secure a MERN Repository

## Goal

Design and demonstrate a secure delivery path for a MERN application without
including real credentials.

## Task 1 — Inventory

List every environment variable used by the API, frontend, workers, CI/CD, and
backup process. Mark each variable as public configuration, sensitive, or
critical secret. Identify who or what needs each value.

## Task 2 — Repository hardening

- Add safe placeholders to `.env.example`.
- Ignore local secret files.
- Add secret scanning to pull requests.
- Search Git history for possible credential exposure.
- Document the rotation owner and escalation path.

## Task 3 — Runtime hardening

Update application configuration so required secrets are validated at startup,
missing values produce actionable errors, and secret values are never logged.
Use different credentials for each environment.

## Task 4 — IAM design

Complete [IAM-ACCESS-MATRIX.md](./IAM-ACCESS-MATRIX.md) for your chosen cloud
provider. Explain the permissions required by the API, deployment identity,
backup job, and monitoring agent. Include explicit denied actions.

## Task 5 — Safe incident simulation

In a non-production environment, simulate a leaked test credential:

1. Detect it with the configured scanner.
2. Revoke the test credential.
3. Issue a replacement with the same minimum permissions.
4. Update runtime configuration.
5. Verify the application and health checks.
6. Review access logs.
7. Write a short blameless postmortem.

## Acceptance criteria

- No real secrets are committed.
- Secret scanning fails on a deliberate test fixture and passes after cleanup.
- Production credentials are inaccessible to build jobs.
- The API has no administrator-level database identity.
- Rotation and rollback steps are documented.
- Evidence includes command output, permission reasoning, and test results.
