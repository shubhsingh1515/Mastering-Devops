# MERN Incident Runbook

## Scope

This runbook covers API errors, latency regressions, failed business flows and
bad deployments. It is a coordination aid, not a replacement for service-specific
operational procedures.

## Severity guide

| Severity | Example impact | Initial response |
|---|---|---|
| SEV-1 | Checkout unavailable for most users, data integrity risk, or regional outage | Page on-call, appoint incident commander, update stakeholders immediately |
| SEV-2 | Material degradation for a subset of users or a key route | Page service owner, mitigate quickly, scheduled updates |
| SEV-3 | Limited degradation with workaround and no major business impact | Create owner-led incident, investigate during business hours |

Set severity from user and business impact, not from CPU alone.

## Response loop

1. **Detect:** acknowledge the alert and open an incident record.
2. **Assess:** identify affected users, regions, endpoints, versions and SLOs.
3. **Coordinate:** appoint an incident commander, communications lead and
   operations lead.
4. **Preserve:** capture dashboards, logs, deployment IDs and UTC timestamps.
5. **Mitigate:** stop rollout, disable a feature, scale safely, fail over or
   roll back.
6. **Verify:** check error rate, latency, availability and business success.
7. **Communicate:** provide concise updates at an agreed interval.
8. **Investigate:** reconstruct the timeline and test causal hypotheses.
9. **Prevent:** create and track corrective and preventive actions.

## Initial triage questions

- When did user impact begin?
- Which routes, tenants, regions and versions are affected?
- Is the problem availability, latency, correctness or data integrity?
- Did a deployment, configuration change or dependency change precede it?
- Is the previous version or a feature flag available?
- What is the safest reversible mitigation?

## Closure criteria

Close the active incident only when:

- user-facing SLIs are stable for an agreed observation window;
- business operations are succeeding;
- queued work and retries are under control;
- stakeholders have received a recovery update;
- evidence and decisions are stored;
- a postmortem owner and review date are assigned.

