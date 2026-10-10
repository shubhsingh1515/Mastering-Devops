# Day 60 Interview Questions — Disaster Recovery

## 1. What is the difference between RPO and RTO?

RPO is the maximum acceptable amount of data loss measured in time. RTO is the
maximum acceptable time to restore service. Backup frequency and replication
influence RPO; automation and recovery architecture influence RTO.

## 2. Are replicas a replacement for backups?

No. Replicas improve availability and can reduce recovery time, but accidental
deletes, corruption, or malicious changes may replicate. Backups and
point-in-time recovery provide independent recovery points.

## 3. How do you prove that backups work?

Restore them into an isolated environment on a schedule, validate data
integrity and application workflows, measure actual RPO/RTO, and track failed
steps to completion. A successful backup job alone is not proof of recovery.

## 4. What must be recovered besides MongoDB?

Application artifacts, infrastructure, networking, secrets references,
object-storage data, DNS/CDN, certificates, queues, monitoring, dashboards,
alert rules, and deployment configuration. Anything needed to recreate
production must be backed up or reproducible from source.

## 5. What would you do if MongoDB were accidentally deleted?

Stop destructive activity, assess scope, preserve evidence, identify the last
valid recovery point, restore into a controlled environment, validate data and
critical workflows, then redirect traffic only after approval. After recovery,
investigate access controls and improve backup protection.

## 6. Why use Infrastructure as Code for DR?

It makes infrastructure repeatable, reviewable, versioned, and faster to
provision. It reduces manual configuration drift and makes the RTO measurable.
It does not replace data backups or restore testing.

## 7. How would you protect backups?

Use encryption, least-privilege access, separate credentials, restricted
network access, retention policies, deletion protection or immutability, audit
logging, and copies outside the primary failure domain.

## 8. What is a recovery drill?

A controlled exercise that provisions or selects a recovery environment,
restores data, deploys the application, validates workflows, and measures
actual RPO/RTO. It should use safe test data and produce improvement actions.

## 9. What if recovery takes 45 minutes but the RTO is 30 minutes?

The plan does not meet its objective. I would identify the slowest steps,
automate infrastructure and traffic switching, improve restore strategy, and
repeat the drill. I would not relabel the result as successful without changing
the objective or the process through an explicit business decision.

## 10. Design a DR strategy for a production MERN application.

“I would define business-driven RPO and RTO targets, protect MongoDB with
provider-supported backups and point-in-time recovery, and use replication for
availability rather than as a backup substitute. Infrastructure and application
artifacts would be reproducible through source control and Infrastructure as
Code. Backups would be encrypted, access-controlled, retained, and tested in an
isolated environment. During a disaster I would declare the incident, recover
infrastructure and data, validate integrity and critical workflows, restore
traffic gradually, measure actual RPO/RTO, and turn drill findings into owned
corrective actions.”

