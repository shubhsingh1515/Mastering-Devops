# Day 58 Commands: SLOs, SLIs, SLAs and Error Budgets

These commands help you collect the measurements needed to define reliability objectives for a MERN application. Replace example URLs, service names and query names with values from your environment.

## Inspect an HTTP health endpoint

```bash
curl -i https://api.example.com/health
```

Measure total request time:

```bash
curl -sS -o /dev/null -w "status=%{http_code} total=%{time_total}s\n" https://api.example.com/health
```

## Calculate availability and error budget

For 99.9% availability across 1,000,000 requests:

```text
allowed failure rate = 1 - 0.999 = 0.001
error budget          = 1,000,000 × 0.001 = 1,000 failed requests
```

For a 30-day calendar window:

```text
30 days × 24 hours × 60 minutes = 43,200 minutes
99.9% SLO allows 43.2 minutes of unavailability
```

## Query request outcomes

The exact query language depends on your metrics platform. The following PromQL-style examples express the intent:

Successful request ratio:

```promql
sum(rate(http_requests_total{status=~"2.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

5xx error ratio:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

Request rate:

```promql
sum(rate(http_requests_total[5m]))
```

## Estimate a latency percentile

If histogram buckets are available:

```promql
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

Do not calculate p95 by averaging instance averages. Aggregate histogram buckets first, then calculate the percentile.

## Record a deployment decision

```text
Version: v12
Traffic: 10%
Stable 5xx rate: 0.2%
Canary 5xx rate: 3.8%
Decision: Stop rollout and investigate or roll back
Reason: Canary is consuming error budget faster than the stable version
```

## Safety reminders

- Never paste production credentials or private response bodies into shared terminals.
- Define the time window and traffic scope for every SLI.
- Keep technical and business indicators separate so a healthy server cannot hide a broken checkout flow.
- Alert on user-impacting SLO degradation, not on every isolated CPU fluctuation.
