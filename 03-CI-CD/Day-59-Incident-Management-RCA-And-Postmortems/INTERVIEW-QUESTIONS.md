# Day 59 Interview Questions — Incident Management

## 1. What is the first priority during an incident?

Reduce user impact. Establish scope, assign incident ownership, and choose the
safest reversible mitigation. Root-cause investigation is important, but it
should not delay rollback, failover or feature-flag disablement.

## 2. What is the difference between mitigation and root cause?

Mitigation reduces or stops current impact, such as rolling back a release.
Root cause explains the mechanism that created the failure. A rollback can
restore service without explaining why the release was unsafe.

## 3. What makes a postmortem blameless?

It describes system behavior, decisions, constraints and safeguards rather than
attacking individuals. Accountability still exists: actions have owners and
dates, but the goal is learning and prevention rather than punishment.

## 4. How would you decide whether to roll back?

Compare the timing of the change with user-facing signals, isolate the affected
version or feature, estimate the rollback risk, and choose rollback when impact
is increasing and the previous version is known to be safer. Verify recovery
with the same metrics that detected the incident.

## 5. What belongs in an incident timeline?

Use UTC timestamps for alert creation, first user impact, acknowledgement,
deployments, hypotheses, decisions, mitigations, recovery and closure. Include
links to evidence and distinguish observed facts from later conclusions.

## 6. How do SLOs help incident response?

They quantify impact and urgency. Availability, latency and business-operation
SLIs show whether users are affected, while error-budget burn helps prioritize
mitigation and informs whether risky changes should continue.

## 7. What is a contributing factor?

A condition that made the incident more likely, more severe or harder to detect,
but is not necessarily the primary causal mechanism. Examples include missing
query-review checks, insufficient canary traffic analysis or an alert that
detected infrastructure symptoms but not checkout failures.

## 8. How do you prevent an RCA from becoming speculation?

Label hypotheses, preserve logs and deployment metadata, reproduce the behavior
where safe, correlate independent signals and record the evidence supporting the
final causal chain. State uncertainty explicitly when evidence is incomplete.

## 9. What is the difference between MTTD and MTTR?

Mean time to detect measures how long it takes to notice an incident. Mean time
to recovery measures how long it takes to restore service. Track both because a
team can detect quickly but recover slowly, or recover quickly after detecting
too late.

## 10. What is a strong final interview answer?

“I would establish impact and an incident commander, stop risky changes, and
apply the safest reversible mitigation. I would communicate status and preserve
evidence, then verify recovery using technical and business SLIs. After the
service is stable, I would reconstruct the timeline, identify root cause and
contributing factors, and create owned, dated preventive actions. Finally I
would review completion and feed the lessons into tests, alerts and release
safety.”

