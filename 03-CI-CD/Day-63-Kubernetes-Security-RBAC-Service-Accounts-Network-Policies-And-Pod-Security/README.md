# 🚀 Day 63 — Kubernetes Security: RBAC, Service Accounts, Network Policies & Pod Security

**Date:** Tuesday, October 13, 2026  
**Phase 4:** DevSecOps & Production Security  
**Duration:** 25–30 minutes  
**Level:** Intermediate → Advanced  
**Focus:** Kubernetes security, MERN production deployments, least privilege, network isolation, secure containers, and DevOps interview preparation.

## Lesson overview

Day 62 secured the software supply chain through dependency scanning, SBOMs,
provenance, and container image signing. Day 63 extends those protections into
the Kubernetes runtime.

The lesson explains how to ensure that only authorized identities can access
Kubernetes resources, application containers run with restricted privileges,
and network traffic is limited to the paths the application actually needs.

## Learning plan

| Time | Topic |
|---|---|
| 0–4 min | Kubernetes security fundamentals |
| 4–9 min | RBAC and service accounts |
| 9–14 min | Network policies |
| 14–19 min | Pod security and secure containers |
| 19–23 min | Secure MERN deployment |
| 23–26 min | Interview preparation and review |
| 26–30 min | Quiz and practical assessment |

## Lesson files

| File | Contents |
|---|---|
| [K8S-RBAC.md](./K8S-RBAC.md) | Kubernetes security fundamentals, RBAC, and service accounts |
| [NETWORK-POLICY.md](./NETWORK-POLICY.md) | Network isolation and workload communication rules |
| [POD-SECURITY.md](./POD-SECURITY.md) | Pod Security Standards and restricted container settings |
| [SECRETS-MANAGEMENT.md](./SECRETS-MANAGEMENT.md) | Kubernetes Secret limitations and secure credential delivery |
| [IMAGE-VERIFICATION.md](./IMAGE-VERIFICATION.md) | Secure MERN architecture and image verification context |
| [INCIDENT-RUNBOOK.md](./INCIDENT-RUNBOOK.md) | Response to a compromised Kubernetes workload |
| [DISASTER-RECOVERY.md](./DISASTER-RECOVERY.md) | Cumulative secure MERN platform and recovery requirements |
| [ASSIGNMENT.md](./ASSIGNMENT.md) | Hands-on API hardening exercise |
| [INTERVIEW-QUESTIONS.md](./INTERVIEW-QUESTIONS.md) | Kubernetes security interview preparation |
| [QUIZ.md](./QUIZ.md) | Knowledge-check questions and answer key |

## Core principles

1. **RBAC controls what an identity can do.**
2. **Service accounts provide workload identities.**
3. **NetworkPolicies restrict unnecessary pod communication.**
4. **Pod security controls reduce container privileges.**
5. **Kubernetes Secrets require appropriate encryption, access controls, and secure delivery.**
6. **Image signing does not replace runtime security.**
7. **Security controls must be tested without breaking legitimate application traffic.**

## Security model

```text
Trusted Software Supply Chain
            |
            v
     Verified Image
            |
            v
    Restricted Workload
            |
            v
     Least-Privilege IAM
            |
            v
      Network Isolation
            |
            v
     Secure Configuration
            |
            v
   Monitoring and Incident Response
            |
            v
      Recovery and Learning
```

> A secure Kubernetes deployment limits both the probability of compromise and
> the damage an attacker can cause after compromising a workload.

For a MERN production application, every identity, container, secret, and
network connection should be explicitly justified.

## Next syllabus lesson: Day 64

**Kubernetes Production Operations — Health Probes, Resource Requests and
Limits, Autoscaling, Rolling Updates, and Zero-Downtime MERN Deployments.**
