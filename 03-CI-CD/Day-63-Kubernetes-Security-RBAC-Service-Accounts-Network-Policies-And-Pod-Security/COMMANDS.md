# Day 63 Kubernetes Security Commands

This file collects the operational commands used throughout the Day 63
Kubernetes security lesson. Run them only against a local cluster or an
approved staging environment unless a production change has been reviewed.

## RBAC and service accounts

```bash
kubectl get serviceaccounts -n production

kubectl auth can-i list pods \
  --namespace production \
  --as=system:serviceaccount:production:app-observer

kubectl auth can-i get secrets \
  --namespace production \
  --as=system:serviceaccount:production:app-observer
```

The `kubectl auth can-i` checks require the caller to be authorized to perform
the requested impersonation check.

## Inspect a deployment

```bash
kubectl get deployment mern-api \
  -n production \
  -o yaml
```

Review the service account, token automounting, security contexts, image
reference, secret mounts, and environment variables.

## Network policies

```bash
kubectl get networkpolicies -n production
```

Before applying a default-deny policy, confirm that DNS, monitoring,
health-check, database, and required external API paths remain available.

## Pod Security Standards

```bash
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted \
  --overwrite
```

Test workloads in staging first. Enforcement can prevent noncompliant
workloads from being created or updated.

## Security review checklist

- Confirm application pods use dedicated service accounts.
- Confirm unnecessary service-account token mounting is disabled.
- Verify expected `kubectl auth can-i` results.
- Inspect NetworkPolicies and required traffic paths.
- Verify non-root execution and dropped capabilities.
- Confirm the deployed image uses the approved immutable digest.
- Check that secrets are not printed, committed, or unnecessarily mounted.
