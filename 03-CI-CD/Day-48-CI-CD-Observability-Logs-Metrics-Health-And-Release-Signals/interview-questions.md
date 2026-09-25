# Day 48 - Interview Questions: CI/CD Observability

## 1. What is observability?

Observability is the ability to understand the internal state and behavior of a running system from its outputs.

### Strong answer

> I use logs for detailed events, metrics for trends and thresholds, and traces for request paths and latency breakdowns. Together they show not only whether a deployment completed, but whether the running release is healthy for users.

## 2. What are the three pillars?

- Logs explain what happened.
- Metrics quantify how much and how often.
- Traces show where a request spent time across services.

## 3. What would you monitor after a deployment?

### Strong answer

> I would compare availability, request rate, 4xx and 5xx errors, p95 and p99 latency, CPU, memory, restarts, database latency and external dependency failures with the pre-deployment baseline. For a MERN checkout flow I would also monitor checkout success, payment completion and order creation. I would use defined thresholds to continue, pause or roll back the rollout.

## 4. What is the difference between liveness and readiness?

Liveness asks whether the process is alive. Readiness asks whether it can safely receive traffic, including required dependency checks.

### Strong answer

> A process can be running while MongoDB is unavailable. Liveness should remain a simple process check, while readiness should fail when the instance cannot serve normal requests safely so the traffic manager can remove it from service.

## 5. Why is CPU not enough to judge a release?

An application can be slow because of a database query, lock, connection pool, external API or application defect while CPU and memory remain normal.

## 6. What does p95 latency mean?

Approximately 95% of requests complete at or below the p95 value. It exposes user experience better than an average when a smaller group is very slow.

## 7. Why are p99 metrics useful?

They show tail latency affecting the slowest portion of users. A large gap between p95 and p99 can indicate occasional severe bottlenecks even when most requests are acceptable.

## 8. What is a correlation ID?

It is an identifier propagated through a request path so related logs and traces can be connected across Nginx, the API, databases and external services.

### Strong answer

> When a customer reports a failed order, I can search for the request ID and reconstruct what happened without guessing from unrelated log lines. I validate the format, propagate it consistently and avoid placing sensitive data in it.

## 9. A deployment succeeded but latency increased. How do you troubleshoot?

### Strong answer

> First I confirm the deployment timestamp and identify the affected endpoints and release version. I compare before and after p95 and p99 latency, request volume and error rates. Then I inspect structured logs and traces, database query latency, connection pools and external API timing. I review the code and configuration changes and reproduce safely in staging. If the degradation exceeds the release thresholds and is correlated with the release, I pause exposure, disable an isolated feature or roll back while preserving evidence.

## 10. Why are business metrics important?

Infrastructure can be healthy while a critical workflow is broken. A checkout success rate of 60% is a release failure even if containers are ready and CPU is low.

## 11. What should happen when 5xx increases from 0.2% to 6%?

Stop increasing exposure, keep the known-good version serving most traffic, investigate version-specific evidence and roll back or isolate the candidate when the impact is confirmed.

## 12. How do you avoid noisy automated rollback?

Use a meaningful threshold, minimum request sample, observation window, consecutive failures, cooldown and alert deduplication. A one-request spike should not trigger a release reversal.

## 13. Why should deployment markers be on dashboards?

They make it possible to correlate a release with metric changes. Without a marker, it is harder to distinguish a release regression from normal traffic or an unrelated dependency incident.

## 14. What belongs in a production log?

A timestamp, level, service, environment, release version, request ID, route, status code, duration and stable error code are useful. Passwords, tokens, payment credentials and unnecessary personal data do not belong there.

## 15. What is the best first recovery action when one flagged feature fails?

Disable the feature flag if it reliably reduces user impact and leaves the rest of the release healthy. Use an application rollback when the broader release is unstable or the defect is not isolated.

## 16. Should every alert trigger a rollback?

No. The response depends on severity, duration, sample size, scope, business impact and confidence that the release caused the problem. Some alerts require investigation or dependency recovery instead.

## 17. How would you explain release health gates?

### Sample answer

> Before increasing exposure, I require a defined observation period, enough traffic, acceptable 5xx and latency compared with the baseline, passing readiness, stable database behavior and healthy business KPIs. If a critical signal fails, I stop the rollout and use the smallest effective recovery action.

## 18. What is the difference between infrastructure and business health?

Infrastructure health says the service is running and dependencies may be reachable. Business health says users can successfully complete the workflows that matter, such as login, checkout and payment.
