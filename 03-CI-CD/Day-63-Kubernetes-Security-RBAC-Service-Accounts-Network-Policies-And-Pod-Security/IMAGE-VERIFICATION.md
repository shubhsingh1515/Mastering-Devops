# Day 63 — Secure MERN Architecture and Image Verification

This file is part of the Day 63 Kubernetes security lesson.

# 7. 🏗️ Secure MERN Architecture in Kubernetes
Let's combine today's concepts.

```
                       USERS
                         |
                         v
                  Ingress / Gateway
                         |
                         v
                 React Frontend Pods
                         |
                         v
                   Express API Pods
                         |
              +----------+----------+
              |                     |
              v                     v
          MongoDB Pods          Redis Pods
```

Now apply the security controls:

ComponentSecurity controlFrontend podsNon-root containers and restricted privilegesExpress APIDedicated service account, minimal privilegesMongoDBNetwork policy restricting access to authorized workloadsRedisRestricted network accessKubernetes APIRBAC and controlled service-account token usageContainer imagesVulnerability scanning, trusted signatures, and digest verificationApplication secretsControlled delivery and least-privilege accessNamespacePod Security enforcement appropriate to workload requirements

### Production design principle
Security is strongest when several controls work together.

For example, if an attacker compromises an Express pod:

1. Restricted container privileges reduce operating-system permissions.
2. RBAC limits Kubernetes API access.
3. Network policies limit reachable workloads.
4. Database permissions limit accessible data and operations.
5. Monitoring and audit logs support detection and investigation.
This is **defense in depth**.

---

# 8. 🎤 Interview Preparation

### Question 1: What is the difference between Role and ClusterRole?
**Strong answer:**

"A Role defines permissions within a namespace. A ClusterRole defines permissions that can be granted across namespaces or for cluster-scoped resources. A RoleBinding grants permissions within a namespace, while a ClusterRoleBinding grants the referenced ClusterRole permissions at the cluster level."

### Question 2: What is a Kubernetes service account?
**Strong answer:**

"A service account provides an identity for a pod or process that may need to interact with the Kubernetes API or other identity-aware systems. I would use dedicated service accounts and assign only the permissions required by each workload. If a workload doesn't need Kubernetes API access, I would avoid unnecessary token mounting."

### Question 3: What is a NetworkPolicy?
**Strong answer:**

"A NetworkPolicy defines rules governing network traffic to or from selected pods. It helps restrict communication between workloads, such as allowing MongoDB ingress only from the Express API. Its enforcement depends on the cluster's network implementation."

### Question 4: How would you harden a production container?
**Strong answer:**

"I would run it as a non-root user, drop unnecessary capabilities, disable privilege escalation, use a read-only root filesystem where practical, apply an appropriate seccomp profile, scan the image for vulnerabilities, and enforce suitable pod security controls. I would test the configuration before production rollout."

### Question 5: How do you secure a Kubernetes MERN deployment?
**Strong answer:**

"I would combine RBAC, dedicated service accounts, network policies, restricted container privileges, secure secret delivery, and image verification. I'd also monitor application behavior and audit access to sensitive resources. The aim is to restrict both who can control the workloads and what a compromised workload can reach."

---

