# Day 63 — Kubernetes Network Policies

This file is part of the Day 63 Kubernetes security lesson.

# 4. 🌐 Kubernetes Network Policies
Kubernetes networking commonly permits broad pod-to-pod communication unless additional controls are implemented.

This can create unnecessary exposure.

Imagine the following architecture:

```
Frontend
   |
   v
Express API
   |
   v
MongoDB
```

The intended communication is:

```
Frontend → API
API      → MongoDB
```

But suppose the database can also be reached by unrelated pods:

```
Unrelated Pod → MongoDB
```

That creates a larger attack surface.

**Network policies allow you to restrict which pods can communicate with other pods or network endpoints.**

Their enforcement requires a compatible network implementation in the cluster. Creating a NetworkPolicy does not guarantee enforcement if the networking implementation doesn't support it.

## Example: Restrict MongoDB access
Suppose MongoDB pods have this label:

```
app: mongodb
```

And the Express API pods have:

```
app: mern-api
```

A policy can restrict ingress to MongoDB so that only the API pods in the production namespace are authorized by the policy.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mongodb-ingress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: mongodb
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: mern-api
      ports:
        - protocol: TCP
          port: 27017
```

### What does this policy do?
It selects MongoDB pods and restricts their incoming connections to traffic from pods labeled `app: mern-api` in the same namespace, on TCP port `27017`.

It does not automatically secure outbound traffic from MongoDB, and it does not establish a complete cluster-wide security policy.

Other required network paths must be considered separately.

### What about Redis?
You might define a separate policy for Redis.

```
Express API
    |
    +----> MongoDB: 27017
    |
    +----> Redis: 6379
```

Don't allow every workload to access every backend service simply because the services share a cluster.

### Important production consideration
When introducing default-deny network policies, plan the allowed communication paths first.

Otherwise, you could accidentally block:

- DNS resolution.
- Monitoring.
- Health checks.
- Required external API connections.
- Application-to-database traffic.
Roll out network restrictions carefully and verify the actual network behavior.

**Interview takeaway:** Network policies reduce lateral movement by restricting unnecessary communication between workloads.

---

