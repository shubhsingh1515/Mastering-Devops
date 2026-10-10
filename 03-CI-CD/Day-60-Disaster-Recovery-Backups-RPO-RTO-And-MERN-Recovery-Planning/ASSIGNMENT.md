# Day 60 Assignment — Design and Test a MERN Disaster Recovery Plan

## Goal

Define a recovery strategy for the cumulative MERN project and prove that the
strategy is executable. The assignment separates availability, backup,
recovery, and validation concerns rather than treating “we have replicas” as a
complete disaster recovery plan.

## Scenario

At `14:00 UTC`, the production MongoDB environment becomes unavailable after a
regional failure. The business objectives are:

```text
RPO: 1 hour
RTO: 30 minutes
```

The latest valid recoverable database point is `13:45 UTC`. A recovery drill
records:

```text
Infrastructure provisioning: 7 minutes
Database restore:            8 minutes
Application deployment:      5 minutes
Validation:                  6 minutes
Traffic switch:              3 minutes
```

## Required work

1. Identify the disaster declaration criteria and incident roles.
2. Explain the difference between the RPO and RTO in this scenario.
3. Calculate potential data loss and total recovery duration.
4. List everything required to recreate the service, not only MongoDB.
5. Define backup frequency, retention, location, encryption, access controls,
   and protection from accidental deletion.
6. Write a step-by-step restore procedure for infrastructure, database, API,
   frontend, secrets, DNS/CDN, and monitoring.
7. Define at least five post-restore data and application validation checks.
8. Describe how a recovery drill can avoid real customer orders and payments.
9. Identify the one-minute RTO margin and propose improvements.
10. Complete [`DISASTER-RECOVERY.md`](./DISASTER-RECOVERY.md) and
    [`DR-RUNBOOK.md`](./DR-RUNBOOK.md) for your project.

## Acceptance criteria

- [ ] RPO and RTO are stated as measurable objectives.
- [ ] Backup and replication are explained as different controls.
- [ ] Restore steps name an owner, input, expected result, and failure action.
- [ ] Recovery is tested in an isolated environment before traffic changes.
- [ ] Database validation covers integrity and business workflows.
- [ ] Secrets and backups are protected without being committed to Git.
- [ ] Actual recovery time and recovered data timestamp are recorded.
- [ ] The plan includes communication, rollback, and post-recovery learning.

