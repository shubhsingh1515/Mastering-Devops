# MERN Disaster Recovery Policy

## 1. Scope and service

This policy applies to the production MERN platform: React frontend, Node.js
API, MongoDB data, object storage, DNS/CDN, secrets, CI/CD artifacts,
monitoring, and infrastructure definitions.

The recovery objective is to restore a safe, observable service, not merely to
make a process respond to HTTP requests.

## 2. Recovery objectives

| Asset or workflow | RPO | RTO | Validation owner |
|---|---:|---:|---|
| Orders and payment state | 1 hour | 30 minutes | Incident commander |
| Product catalog | 4 hours | 60 minutes | Application owner |
| User-uploaded assets | 4 hours | 60 minutes | Storage owner |
| Logs and dashboards | 24 hours | 4 hours | Observability owner |

RPO is the maximum acceptable time-based data loss. RTO is the maximum
acceptable time to restore the service. These are business commitments to
validate, not guesses to repeat in documentation.

## 3. Backup policy

- Use provider-supported snapshots and point-in-time recovery for MongoDB when
  available; use encrypted logical exports for an additional recovery path.
- Capture application artifacts, infrastructure definitions, dashboards, DNS
  configuration, and object-storage data or make them reproducible from source.
- Keep copies in a separate recovery location with restricted access,
  encryption, retention controls, audit logging, and deletion protection.
- Use separate backup credentials and never store credentials in a repository.
- Monitor backup completion, age of the newest valid recovery point, and
  available retention window.
- Perform a restore test at least monthly and after major backup changes.

## 4. Disaster declaration

Declare a disaster when the primary environment cannot meet the normal
incident recovery path, when a region is unavailable, when data integrity is
uncertain, or when the expected outage threatens the RTO. The incident
commander records the declaration time in UTC and freezes unrelated changes.

## 5. Recovery sequence

1. Confirm impact, scope, last known good data point, and security concerns.
2. Preserve evidence and prevent further destructive writes.
3. Provision the recovery environment from reviewed Infrastructure as Code.
4. Restore secrets references, network access, monitoring, and application
   artifacts.
5. Restore MongoDB to an isolated target at the selected recovery timestamp.
6. Validate data integrity, indexes, connectivity, and critical workflows.
7. Deploy the API and frontend, then run safe smoke tests.
8. Switch DNS, CDN, load-balancer, or traffic-manager routing gradually.
9. Monitor error rate, latency, checkout success, database health, and logs.
10. Record actual RPO/RTO and keep the incident open until stability is proven.

## 6. Security and safety controls

The recovery team must use least-privilege access, approved break-glass
procedures, audit logging, and a two-person review for destructive operations.
Restore into a clean environment when compromise is possible. Rotate
credentials after a security-related disaster and do not restore known-malicious
configuration or artifacts.

## 7. Recovery success criteria

Recovery is successful only when:

- the recovered timestamp meets the declared RPO;
- traffic restoration meets the declared RTO;
- data integrity checks pass;
- critical read and write workflows pass with safe test data;
- monitoring and alerting are active;
- the service is stable for the agreed observation period; and
- stakeholders have received the recovery update.

