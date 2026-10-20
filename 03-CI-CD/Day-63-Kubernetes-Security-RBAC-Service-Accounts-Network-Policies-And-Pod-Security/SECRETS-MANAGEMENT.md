# Day 63 — Kubernetes Secrets Management

This file is part of the Day 63 Kubernetes security lesson.

# 6. 🔐 Kubernetes Secrets: Important Limitations
Your Day 61 lesson covered application secrets.

Kubernetes provides a Secret resource:

```
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-credentials
  namespace: production
type: Opaque
stringData:
  MONGODB_URI: "REPLACE_WITH_SECRET_FROM_SECURE_PROVISIONING"
```

The value shown is a placeholder. Do not commit real credentials to Git.

A Secret resource is not automatically a complete secrets-management solution.

Kubernetes Secret data is base64-encoded in its standard representation, and base64 encoding is not encryption.

For production, consider:

- Encryption at rest for Kubernetes Secret data.
- Strict RBAC for reading and modifying secrets.
- Secure delivery of credentials to workloads.
- External secrets managers where appropriate.
- Credential rotation.
- Avoiding secret exposure through logs and deployment output.
Your application should only receive the secrets it needs.

For example:

```
Express API
    |
    v
MongoDB credential
    |
    v
Restricted database identity
```

The React frontend must never receive MongoDB credentials or server-side signing keys.

Anything shipped to a browser should be treated as potentially visible to the user.

---

