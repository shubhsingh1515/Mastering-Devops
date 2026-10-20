# Day 63 — Kubernetes Security Interview Questions

This file is part of the Day 63 Kubernetes security lesson.

# 9. 🔁 Review Questions From Earlier Topics
These five questions connect today's lesson to the previous curriculum.

**Q1 — Day 59, Incident Management**

A new deployment increases API errors and checkout failures. What should you prioritize first?

**Q2 — Day 60, Disaster Recovery**

What is the difference between RPO and RTO?

**Q3 — Day 61, Secrets Management**

Why is deleting a leaked password from the latest Git commit insufficient?

**Q4 — Day 62, Supply-Chain Security**

What is the difference between container image signing and vulnerability scanning?

**Q5 — Day 62, Software Provenance**

What information does software provenance provide that an SBOM alone does not necessarily provide?

<details>
<summary><strong>Reveal the answers</strong></summary>

**1.** Assess user impact and mitigate the incident, such as stopping the rollout or rolling back when appropriate.

**2.** RPO defines the acceptable data-loss window; RTO defines the target time to restore service.

**3.** The secret may remain in Git history, clones, logs, or artifacts. Revoke or rotate it and investigate possible misuse.

**4.** Signing provides evidence about artifact integrity and signing identity; scanning identifies potential vulnerabilities. Neither replaces the other.

**5.** Provenance describes how an artifact was produced, including information about its source and build process. An SBOM primarily inventories the artifact's components.

</details>

---

# Day 63 — Final Interview Challenge

This challenge is part of the Day 63 Kubernetes security lesson.

# 14. 🎤 Final Interview Challenge
**Question:**

> Design a secure Kubernetes environment for a production MERN application. Explain how you would limit the impact of a compromised Express pod.
Structure your answer:

```
1. Identity and RBAC
2. Dedicated service accounts
3. Restricted container privileges
4. Network isolation
5. Secure secret delivery
6. Trusted image verification
7. Monitoring and auditing
8. Incident containment
9. Recovery and validation
10. Preventive improvements
```

### Model interview answer
"I would assign the Express application a dedicated service account and grant only the Kubernetes permissions it actually requires. If it doesn't need Kubernetes API access, I would disable automatic service-account token mounting. I'd run the container as a non-root user, drop unnecessary capabilities, prevent privilege escalation, and apply suitable pod security controls.

"For networking, I'd use NetworkPolicies to restrict database access to authorized workloads and control unnecessary outbound traffic, while preserving required DNS and monitoring paths. I'd deliver credentials securely and use a restricted database identity. I'd also verify container signatures and required provenance, scan images for vulnerabilities, and monitor application and audit logs.

"If the pod appeared compromised, I'd coordinate containment, preserve relevant evidence, rotate potentially exposed credentials, assess data integrity, and verify recovery. Afterward, I'd use the incident findings to improve access controls, network segmentation, monitoring, and deployment policies."

---

