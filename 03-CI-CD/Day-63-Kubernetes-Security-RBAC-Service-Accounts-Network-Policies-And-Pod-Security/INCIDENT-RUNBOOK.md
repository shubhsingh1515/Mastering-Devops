# Day 63 — Compromised Kubernetes Workload Incident Runbook

This file is part of the Day 63 Kubernetes security lesson.

# 12. 🧪 Practical Assessment — A Compromised MERN Pod
You are operating a production Kubernetes cluster.

Your monitoring system detects suspicious behavior from the Express API pod.

Initial investigation finds:

```
Application: MERN API

Container:
Running as root

Service account:
Default service account

RBAC:
Excessive permissions

Network:
MongoDB reachable from unrelated pods

Secrets:
Production credentials mounted into the container

Image:
Vulnerability scanning completed
Signature verification passed
```

The image's signature is valid, but the runtime configuration is weak.

### Task 1 — What are the immediate priorities?
A strong response should:

1. Coordinate an incident and assess potential impact.
2. Contain the suspicious workload safely.
3. Preserve relevant logs and audit evidence.
4. Rotate credentials if exposure or compromise is suspected.
5. Determine whether unauthorized API or database access occurred.
6. Verify the integrity of critical data and services.
Avoid immediately deleting all evidence or restarting everything without considering the investigation and containment requirements.

### Task 2 — Which Kubernetes controls should be improved?

```
Excessive RBAC
      |
      v
Restrict permissions
      |
      v
Unnecessary service-account token
      |
      v
Disable automatic mounting where appropriate
      |
      v
Root container
      |
      v
Run as non-root
      |
      v
Broad network access
      |
      v
Introduce tested NetworkPolicies
```

Also review secret delivery and the permissions associated with the database identity.

### Task 3 — Does the valid image signature prove that the pod is secure?
**No.**

The signature provides evidence about the image's integrity and signing identity. It does not prove that the running workload has safe permissions, that its secrets are protected, or that its network exposure is appropriate.

### Task 4 — How do you verify the remediation?
Check that:

- The API runs successfully with restricted privileges.
- Its service account has only required permissions.
- Unnecessary Kubernetes API access is disabled.
- MongoDB accepts traffic only from intended workloads.
- Required application workflows continue working.
- Suspicious activity has stopped or is understood.
- Monitoring, audit logging, and incident follow-up are complete.

### Assessment scoring
AreaPointsIncident containment20RBAC and service-account remediation20Network isolation20Container hardening20Verification and prevention20**Total****100**

**90–100:** Strong production security judgment.
**75–89:** Good foundation; refine isolation and remediation sequencing.
**60–74:** Review Kubernetes security controls.
**Below 60:** Repeat the practical exercise before progressing.

---

