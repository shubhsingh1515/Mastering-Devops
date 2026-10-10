# MERN Disaster Recovery Runbook

This runbook is an execution checklist. Adapt resource names, commands, and
owners to the environment before using it during a real event.

## 1. Declare and assign

- [ ] Open a disaster incident and record the declaration time in UTC.
- [ ] Appoint an incident commander, technical recovery lead, communications
      lead, database owner, and security lead.
- [ ] Announce the affected services, current impact, RPO, RTO, and next update.
- [ ] Freeze unrelated deployments and destructive database operations.

## 2. Assess the recovery point

- [ ] Identify the latest valid MongoDB snapshot or point-in-time timestamp.
- [ ] Confirm that the backup is complete, readable, and outside the failed
      environment when possible.
- [ ] Calculate `disaster time - recovered timestamp` and compare it with RPO.
- [ ] Decide whether a security investigation requires a clean-room restore.

## 3. Recover infrastructure

- [ ] Select the approved recovery region or environment.
- [ ] Review and apply the Infrastructure as Code plan.
- [ ] Create networking, compute, load balancing, storage, and monitoring.
- [ ] Confirm quotas, DNS ownership, certificates, and connectivity.
- [ ] Record start and finish times for each step.

## 4. Recover data and secrets

- [ ] Restore MongoDB into an isolated recovery target.
- [ ] Restore object storage or confirm its independent availability.
- [ ] Recreate secret references from the secret manager; never copy secrets
      into this runbook.
- [ ] Verify database indexes, collection access, schema expectations, and
      newest recovered business timestamp.
- [ ] Record the backup ID, restore timestamp, and operator.

## 5. Recover the application

- [ ] Deploy the last approved immutable API and frontend artifacts.
- [ ] Configure the recovered database and environment-specific endpoints.
- [ ] Enable logs, metrics, traces, health checks, and alerts.
- [ ] Confirm readiness only after dependencies are reachable.
- [ ] Run non-billing smoke tests for login, catalog, cart, and order flow.

## 6. Validate before traffic

- [ ] Compare critical record counts and representative records.
- [ ] Verify no duplicate orders, invalid payment states, or broken references.
- [ ] Confirm write operations use safe test data and are removed afterward.
- [ ] Confirm p95 latency, error rate, database CPU, and queue depth are normal.
- [ ] Obtain technical and incident-commander approval to switch traffic.

## 7. Restore traffic and observe

- [ ] Shift a small traffic percentage or a controlled test cohort first.
- [ ] Watch business and technical SLIs for at least the agreed observation
      period.
- [ ] Increase traffic only when the recovered service remains healthy.
- [ ] Keep the original environment isolated until data and security owners
      approve decommissioning or reconciliation.

## 8. Close and learn

- [ ] Record actual RPO, actual RTO, and remaining recovery margin.
- [ ] List manual steps, failed commands, missing permissions, and delays.
- [ ] Send stakeholder communication with current status and next actions.
- [ ] Schedule a blameless postmortem and assign dated corrective actions.
- [ ] Update Infrastructure as Code, alerts, backup policy, and this runbook.

## Communication template

```text
[UTC time] Disaster recovery update
Incident: <ID>
Impact: <services, regions, users, business workflows>
Recovery point: <backup ID and UTC timestamp>
RPO status: <met / breached / unknown>
RTO status: <met / at risk / breached>
Current step: <step and owner>
Next update: <UTC time>
```

