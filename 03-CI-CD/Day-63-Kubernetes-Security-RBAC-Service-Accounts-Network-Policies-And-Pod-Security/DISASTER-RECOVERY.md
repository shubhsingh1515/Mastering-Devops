# Day 63 — Secure MERN Platform Project and Recovery Requirements

This file is part of the Day 63 Kubernetes security lesson.

# 13. 🏆 October Cumulative Project — Secure MERN Platform on Kubernetes
Continue your cumulative DevOps project by integrating today's Kubernetes security controls with the previous lessons on SLOs, incident response, disaster recovery, secrets management, and software supply-chain security.

## Target architecture

```
                    Git Repository
                          |
                          v
                    Secure CI/CD
                          |
              +-----------+-----------+
              |                       |
              v                       v
        Security Scanning        Automated Tests
              |                       |
              +-----------+-----------+
                          |
                          v
                 Build and Verify Image
                          |
                          v
                  Approved Image Digest
                          |
                          v
                    Kubernetes
                          |
                  +-------+-------+
                  |               |
                  v               v
            React Frontend    Express API
                                  |
                           +------+------+
                           |             |
                           v             v
                        MongoDB        Redis
```

Apply the following controls:

- Dedicated service accounts.
- Least-privilege RBAC.
- Restricted pod security.
- Network policies for required communication paths.
- Secure secret delivery.
- Image signature and provenance verification.
- Vulnerability scanning.
- Production monitoring and SLO-based alerts.
- Tested database recovery procedures.

## New deliverables
Create or update these files in your project:

FilePurpose`K8S-RBAC.md`Document identities, roles, and permissions`NETWORK-POLICY.md`Describe permitted workload communication`POD-SECURITY.md`Document container and pod security settings`SECRETS-MANAGEMENT.md`Explain how credentials are delivered and rotated`IMAGE-VERIFICATION.md`Define image-signing and provenance checks`INCIDENT-RUNBOOK.md`Describe response to a compromised workload`DISASTER-RECOVERY.md`Document recovery objectives and tested procedures

### Project acceptance criteria
Your test environment should demonstrate that:

1. The MERN API does not require unnecessary Kubernetes API permissions.
2. The application runs without unnecessary root privileges.
3. MongoDB traffic is restricted to authorized workloads.
4. Secrets are not committed to Git or unnecessarily exposed to pods.
5. Only approved container images can be deployed under your release policy.
6. Security alerts and suspicious behavior can be investigated.
7. Critical application workflows remain healthy after security controls are introduced.
8. Backup and recovery capabilities remain available.
**The goal is not to add YAML files for the sake of documentation. It is to demonstrate that the security controls actually work.**

---

