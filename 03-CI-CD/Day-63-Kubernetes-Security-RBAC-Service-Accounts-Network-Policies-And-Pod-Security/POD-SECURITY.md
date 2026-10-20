# Day 63 — Pod Security and Restricted Containers

This file is part of the Day 63 Kubernetes security lesson.

# 5. 🛡️ Pod Security: Run Containers With Restricted Privileges
A container can run even when its security configuration is dangerously permissive.

Consider:

```
Express Container
       |
       v
Runs as root
       |
       v
Excessive Linux capabilities
       |
       v
Privileged access
```

A compromised application could have more operating-system privileges than necessary.

Kubernetes pod security controls help reduce this risk.

A common approach is to use the **Restricted** Pod Security Standard where workloads can meet its requirements.

For example, a namespace can enforce the Restricted policy:

```
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted \
  --overwrite
```

**Caution:** Applying enforcement to an existing production namespace can prevent noncompliant workloads from being created or updated. Test the policy in a staging environment and validate every workload before enabling enforcement in production.

## Harden the Express container
Example pod security settings:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mern-api
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mern-api
  template:
    metadata:
      labels:
        app: mern-api
    spec:
      serviceAccountName: mern-api
      automountServiceAccountToken: false

      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        seccompProfile:
          type: RuntimeDefault

      containers:
        - name: api
          image: registry.example.com/mern-api@sha256:REPLACE_WITH_REAL_DIGEST

          ports:
            - containerPort: 5000

          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
```

These settings reduce the container's privileges.

However, the image must support running as the specified non-root user. A read-only root filesystem may also require writable temporary volumes or other explicit storage for applications that need to write temporary files.

Test the application before applying these settings to production.

### Why drop Linux capabilities?
Linux capabilities divide certain traditionally root-only operations into smaller privileges.

Dropping all capabilities and adding back only those genuinely needed reduces the container's available privileges.

### Interview question
**Why should containers run as non-root?**

A strong answer:

"Running as a non-root user reduces the privileges available to a compromised application process. I would also disable unnecessary privilege escalation, drop unused Linux capabilities, use a read-only root filesystem where practical, and apply appropriate pod security policies. These controls reduce risk but do not eliminate every container escape or host-level vulnerability."

---

