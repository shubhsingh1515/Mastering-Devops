# Day 61 Interview Questions

## 1. How do you manage secrets in production?

Use a managed secrets service, separate credentials by environment, deliver
values at runtime, restrict access by workload identity, rotate regularly, and
audit reads. CI logs and image layers must never contain raw values.

## 2. What is least privilege?

Least privilege gives an identity only the permissions needed for its current
job, on the smallest scope, for the shortest practical duration.

## 3. What is your first action after a secret leaks into Git?

Revoke or rotate the credential immediately, then investigate access and remove
the exposure. Deleting the latest file alone is not sufficient.

## 4. Why is OIDC useful in CI/CD?

OIDC can let a trusted workflow obtain a short-lived, scoped token without
storing a long-lived cloud access key in the repository.

## 5. Why should secrets not be baked into Docker images?

Image layers, registries, caches, and `docker history` can preserve them. Runtime
injection allows rotation without rebuilding and limits who can retrieve values.

## 6. Why should backup access be separate from application access?

An attacker who compromises the application should not automatically be able to
delete or encrypt recovery data. Separation protects recovery independence.

## 7. How do you verify a secrets manager integration?

Test startup validation, allowed and denied identities, rotation behavior,
redaction, audit events, and recovery when a secret is unavailable.

## 8. What security checks belong in CI?

Dependency scanning, secret scanning, linting, unit tests, image scanning,
artifact integrity checks, and policy checks should run before promotion.
