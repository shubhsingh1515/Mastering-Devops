# Day 63 — Kubernetes RBAC and Service Accounts

This file is part of the Day 63 Kubernetes security lesson.

# 1. 🧠 Kubernetes Security Fundamentals
Consider this production architecture:

```
                   USERS
                     |
                     v
              Ingress / Gateway
                     |
                     v
              React Frontend
                     |
                     v
              Node.js / Express
                     |
              +------+------+
              |             |
              v             v
           MongoDB       Redis Cache
```

The application runs in Kubernetes.

Each component has different security requirements.

For example:

- React needs to serve frontend assets.
- Express needs to process requests and access MongoDB.
- MongoDB needs to accept connections from authorized application workloads.
- Monitoring agents need access to specific metrics.
- CI/CD needs permission to deploy approved workloads.
- Ordinary application pods generally do not need Kubernetes administrator privileges.
Your goal is to enforce these boundaries.

### Four important security layers
LayerSecurity controlIdentityKubernetes RBAC and service accountsNetworkNetwork policies and controlled ingressWorkloadPod security settings and restricted container privilegesSoftwareTrusted images, vulnerability scanning, and secure configuration

Security should be enforced at multiple layers because one control can fail or be misconfigured.

---

# 2. 🔑 Kubernetes RBAC: Who Can Do What?
RBAC stands for **Role-Based Access Control**.

It controls which identities can perform which actions on Kubernetes resources.

For example, a deployment engineer might need to:

```
Read deployments
Read pods
Inspect pod logs
Update application deployments
```

They probably do not need to:

```
Read every secret
Modify cluster-wide roles
Create cluster administrators
Change unrelated production workloads
```

### Kubernetes RBAC components
ComponentPurposeRoleDefines permissions within a namespaceClusterRoleDefines permissions that can be granted across namespaces or for cluster-scoped resourcesRoleBindingGrants a Role or ClusterRole's permissions within a namespaceClusterRoleBindingGrants ClusterRole permissions at the cluster levelServiceAccountProvides an identity for a workload or process

### Example: Read-only access to application pods
Suppose your MERN application runs in the `production` namespace.

Create a Role that permits reading pods and deployments.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-readonly
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]

  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "watch"]
```

This Role defines the permissions but does not grant them to anyone yet.

You must bind the Role to an identity.

### Bind the permissions to a service account

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-observer
  namespace: production
automountServiceAccountToken: false
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-observer-binding
  namespace: production
subjects:
  - kind: ServiceAccount
    name: app-observer
    namespace: production
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-readonly
```

The service account can now receive the specified permissions through the RoleBinding.

The setting `automountServiceAccountToken: false` prevents automatic mounting of the service account token into pods that use this service account by default. It does not remove the RBAC permissions themselves, and workloads that need Kubernetes API access must be configured accordingly.

### Verify the permissions
You can check whether the identity has a particular permission:

```
kubectl auth can-i list pods \
  --namespace production \
  --as=system:serviceaccount:production:app-observer
```

Expected result:

```
yes
```

Check access to secrets:

```
kubectl auth can-i get secrets \
  --namespace production \
  --as=system:serviceaccount:production:app-observer
```

Expected result:

```
no
```

This is the practical meaning of least privilege.

**Interview takeaway:** RBAC permissions are granted to identities. Creating a Role alone does not grant access.

---

# 3. 🤖 Service Accounts: Secure Application Identities
A Kubernetes service account provides an identity for a pod or process.

Suppose your Express application needs to access MongoDB.

Does it automatically need to call the Kubernetes API?

**No.**

If it doesn't need Kubernetes API access, avoid granting it unnecessary permissions.

Consider this insecure setup:

```
Express Pod
     |
     v
Default Service Account
     |
     v
Excessive Kubernetes Permissions
```

If an attacker compromises the Express application, excessive Kubernetes permissions may allow the attacker to manipulate other workloads or access additional resources.

A safer design is:

```
Express Pod
     |
     v
Dedicated Service Account
     |
     +----> No unnecessary API access
     |
     +----> No cluster-admin privileges
     |
     +----> Only required permissions
```

### Disable unnecessary token mounting
In a Deployment:

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
      containers:
        - name: api
          image: registry.example.com/mern-api@sha256:REPLACE_WITH_REAL_DIGEST
          ports:
            - containerPort: 5000
```

The image digest is a placeholder; replace it with the digest of the approved image from your Day 62 supply-chain workflow.

The dedicated `mern-api` service account must exist in the `production` namespace.

If your application genuinely needs Kubernetes API access, design the necessary permissions and token handling explicitly rather than disabling access blindly.

### Interview question
**Why should application pods avoid using the default service account with unnecessary permissions?**

A strong answer:

"An application pod may be compromised through an application vulnerability. If its service account has excessive permissions, the attacker could potentially use those permissions to access secrets or manipulate other Kubernetes resources. I would use dedicated service accounts, grant only required permissions, and disable token automounting for workloads that don't need Kubernetes API access."

---

