# Secrets Management Standard

This document defines how the MERN platform stores, delivers, rotates, and audits
secrets. It is a safe reference: it intentionally contains placeholders only.

## 1. Secret inventory

| Secret or setting | Used by | Classification | Preferred storage |
|---|---|---|---|
| `MONGODB_URI` | API runtime | Critical | Managed secrets manager |
| `JWT_SECRET` | API runtime | Critical | Managed secrets manager |
| `PAYMENT_API_KEY` | Payment worker | Critical | Managed secrets manager |
| `REGISTRY_TOKEN` | Image publishing job | High | CI/CD secret store or OIDC |
| `DATABASE_BACKUP_TOKEN` | Backup process | Critical | Managed secrets manager |
| `LOGGING_ENDPOINT` | API and workers | Sensitive | Secret store if authenticated |

Non-sensitive configuration such as `PORT`, feature flags, and service hostnames
may be stored as ordinary environment configuration.

## 2. Storage and delivery rules

1. Never place real credentials in source code, `.env` files, Dockerfiles,
   Docker image layers, README files, tickets, screenshots, or chat messages.
2. Keep `.env.example` limited to variable names and safe placeholders.
3. Store production secrets in a managed secrets service with encryption,
   access logging, retention, and backup controls enabled.
4. Use separate credentials for development, test, staging, and production.
5. Deliver secrets at runtime through the deployment platform; do not bake them
   into an image during `docker build`.
6. Grant each workload access only to the named secrets it requires.
7. Prevent secret values from appearing in logs, error responses, metrics,
   traces, shell history, and CI artifacts.

## 3. Rotation and revocation procedure

When a secret may have been exposed:

1. Declare a security incident and record the time and suspected scope.
2. Revoke or disable the old credential immediately.
3. Review access logs for use of the old credential.
4. Generate a new credential with the smallest required permissions.
5. Store the replacement in the approved secret store.
6. Restart or reload only the affected workloads.
7. Validate health checks and critical application flows.
8. Remove the leaked value from current files and CI logs, then assess whether
   history rewriting is required.
9. Document the incident, evidence, user impact, and preventive actions.

Deleting a value from the latest Git commit is not rotation. Existing clones,
forks, logs, and image layers may still contain the original value.

## 4. CI/CD controls

- Run secret scanning on pull requests and protected branches.
- Fail the pipeline when a verified secret is detected.
- Prefer short-lived OIDC/federated credentials over long-lived cloud keys.
- Restrict production secret access to approved deployment environments.
- Mask secret values in job output and never print the process environment.
- Keep build, image-publish, and deploy permissions separate.
- Retain audit records without retaining raw secret values.

## 5. Local development checklist

```text
cp .env.example .env
# Fill .env with local-only values; never commit it.
git status --short
git check-ignore .env
```

Before opening a pull request, verify:

- `.env` and `.env.*` are ignored except `.env.example`.
- No credential appears in source, tests, fixtures, or documentation.
- CI logs do not echo environment variables.
- The application fails clearly when a required secret is missing.

## 6. Required application behavior

Applications must fail closed when a critical secret is absent or invalid.
They should emit a safe configuration error containing the variable name, but
never the value. Secret reads should happen through one configuration boundary
so validation, redaction, and rotation behavior remain consistent.
