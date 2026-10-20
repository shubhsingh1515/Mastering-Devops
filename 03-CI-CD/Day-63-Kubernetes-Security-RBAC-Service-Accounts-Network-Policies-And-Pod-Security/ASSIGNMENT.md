# Day 63 Assignment — Harden a MERN API Deployment

This file is part of the Day 63 Kubernetes security lesson.

# 11. 💻 Practical Exercise — Harden a MERN API Deployment
**Duration:** 5–10 minutes.

Use a local Kubernetes cluster or a disposable staging environment. Do not experiment with production RBAC or namespace enforcement without an approved rollout plan.

### Task A — Inspect current service accounts

```
kubectl get serviceaccounts -n production
```

Inspect the API deployment:

```
kubectl get deployment mern-api \
  -n production \
  -o yaml
```

Look for:

- `serviceAccountName`
- `automountServiceAccountToken`
- Container security contexts
- Image references
- Secret mounts and environment variables

### Task B — Check effective permissions

```
kubectl auth can-i list pods \
  --namespace production \
  --as=system:serviceaccount:production:app-observer
```

Then test access to secrets:

```
kubectl auth can-i get secrets \
  --namespace production \
  --as=system:serviceaccount:production:app-observer
```

These checks evaluate authorization for the specified identity. Ensure your own account is authorized to perform impersonation checks.

### Task C — Review network exposure

```
kubectl get networkpolicies -n production
```

Ask:

- Can unrelated pods connect to MongoDB?
- Does the API need unrestricted outbound access?
- Will DNS continue to work if default-deny egress is introduced?
- Are monitoring and health-check paths preserved?

### Task D — Harden the container
Verify that the API can run successfully with:

```
runAsNonRoot: true
allowPrivilegeEscalation: false
capabilities.drop: ["ALL"]
readOnlyRootFilesystem: true
seccompProfile.type: RuntimeDefault
```

Some applications need explicit writable temporary storage. Test application startup, logging, temporary-file handling, and health checks before enforcing these settings.

---

