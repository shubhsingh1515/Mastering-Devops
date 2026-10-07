# Day 59 Commands — Incident Management, RCA and Postmortems

Use these commands only with approved credentials and redacted production data. Replace
example names, URLs and selectors with values from your environment.

## Triage and health checks

```bash
curl -fsS -o /dev/null -w "status=%{http_code} total=%{time_total}s\n" \
  https://api.example.com/health

curl -fsS https://api.example.com/ready | jq .

kubectl get pods -n production -o wide
kubectl get events -n production --sort-by=.lastTimestamp | tail -30
kubectl describe deployment api -n production
```

Do not assume that a healthy process means a healthy service. Check readiness,
dependencies and a representative business operation such as order creation.

## Preserve evidence

Record the incident ID, current deployment version, UTC timestamps, dashboards,
queries and mitigation decisions before changing the system. Export only the
minimum required logs and remove secrets, tokens, cookies and personal data.

```bash
git rev-parse HEAD
git log -5 --oneline
kubectl rollout history deployment/api -n production
kubectl get deployment api -n production \
  -o jsonpath='{.spec.template.metadata.labels.version}{"\n"}'
```

## Rollout mitigation

```bash
# Pause a rollout while evidence is collected
kubectl rollout pause deployment/api -n production

# Roll back the latest deployment when impact is increasing
kubectl rollout undo deployment/api -n production
kubectl rollout status deployment/api -n production --timeout=5m

# Confirm the rollback revision
kubectl rollout history deployment/api -n production
```

If the service uses feature flags, disable only the suspected feature first. A
targeted mitigation is safer than disabling unrelated functionality.

## Query incident signals

PromQL-style examples:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))

histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)

sum(rate(checkout_completed_total[5m]))
/
sum(rate(checkout_started_total[5m]))
```

Compare the affected version with the stable version. Use the same time window,
traffic slice and aggregation method; otherwise a comparison can be misleading.

## Incident calculations

```text
MTTD = detection time - incident start time
MTTR = recovery time - incident start time

impact rate = failed or incorrect operations / total operations
burn rate   = observed error rate / permitted error rate
```

For a 99.9% availability SLO, the permitted failure rate is 0.1%. An observed
5% failure rate burns the budget at approximately:

```text
5% / 0.1% = 50x
```

## Safe communication

```text
[UTC time] Incident commander: <name>
Impact: <users, routes, regions, business operation>
Current symptoms: <measured signals>
Mitigation: <action and result>
Next update: <time>
```

Keep the incident channel focused on facts, decisions, owners and next updates.
Move long-term investigation notes into the postmortem after recovery.

