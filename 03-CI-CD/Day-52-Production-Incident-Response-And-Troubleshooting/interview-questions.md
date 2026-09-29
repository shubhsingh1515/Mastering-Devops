# Day 52 Interview Questions: Incident Response and Troubleshooting

## 1. What is the incident lifecycle?

A practical lifecycle is detect, triage, assess impact, mitigate, investigate, recover, verify and learn. The stages prevent responders from confusing a notification with a diagnosis or a running process with recovery.

## 2. What is triage?

Triage establishes scope, severity, affected services and user impact. It answers what is failing, where it is failing, when it started, how many users are affected and whether the problem is getting worse.

## 3. What is your first priority during an incident?

Reduce user impact safely. I would confirm the signal, establish scope and choose a reversible mitigation such as stopping a rollout, disabling an isolated feature or routing traffic away. I would then investigate root cause using preserved evidence.

## 4. Why should you not immediately restart a failing service?

A restart may hide the symptom, destroy in-memory evidence or create a recovery storm. I would first capture relevant logs and metrics. If restarting is the correct mitigation, I would do it deliberately, record the reason and verify the outcome.

## 5. When would you roll back?

I would roll back when a recent deployment clearly caused significant impact, the previous version is known to be good and the application remains compatible with the current schema and data. I would not assume an application rollback makes a database rollback safe.

## 6. When would you fix forward?

I would fix forward when the schema or data is incompatible with the old version, when rollback would increase risk or when a small verified correction can be deployed faster and more safely than returning to the previous release.

## 7. When is a feature flag useful during an incident?

A feature flag is a fast mitigation when the broken behavior is isolated and the previous behavior remains safe. Disabling the flag can reduce impact without rolling back unrelated application changes.

## 8. Why can database migrations make rollback dangerous?

The previous application may expect fields or formats that the new migration removed or changed. Backward-compatible expand-and-contract migrations allow mixed application versions and make rollback safer.

## 9. How do you troubleshoot a `PaymentTimeout`?

I would establish the affected checkout scope, check the deployment and feature-flag timeline, inspect payment-provider status and latency, compare timeout configuration and retries, trace representative request IDs and check whether orders or charges partially completed. I would mitigate first by disabling the isolated flow or using an approved fallback.

## 10. What proves recovery?

Recovery requires acceptable user-facing behavior and supporting evidence: 5xx rate and latency return to baseline, health and readiness pass, failure logs stop, payment and database dependencies are healthy, checkout success recovers and no duplicate orders or charges are observed.

## 11. What is the difference between a trigger and a root cause?

The trigger is the event that started or exposed the incident, such as a deployment. The root cause is the underlying system weakness that allowed the failure, such as invalid configuration passing through a pipeline without validation.

## 12. How do you use hypotheses during troubleshooting?

I write plausible causes, identify evidence that could confirm or weaken each one and test them in priority order. For example, a MongoDB connection error leads to checks for service health, hostname, DNS, network access, credentials and recent configuration changes.

## 13. Why are business metrics important?

Infrastructure can look healthy while customers cannot complete important workflows. Checkout success, payment completion, login success and duplicate-order rates reveal impact that CPU, memory and basic health checks can miss.

## 14. What should an incident timeline contain?

It should contain UTC timestamps, alerts, observed impact, deployments, decisions, mitigation actions, verification results and recovery. Facts and assumptions should be clearly separated.

## 15. What makes a good post-incident review?

It explains impact, timeline, detection, mitigation, root cause, contributing factors and corrective actions. It focuses on system improvements, assigns owners and avoids blaming an individual.

## 16. How would you explain `localhost` in Docker Compose?

`localhost` inside the API container refers to that API container, not MongoDB in another container. The API normally connects to MongoDB through the MongoDB service name on the shared Docker network.

## 17. What evidence should be preserved before cleanup?

Preserve relevant logs, metrics, deployment metadata, container status, configuration validation results, request IDs and the incident timeline. Redact secrets, tokens, credentials, cookies, payment data and unnecessary personal information.

## 18. How do you prevent a similar incident?

Add configuration validation, production-like staging smoke tests, dependency readiness checks, canary monitoring, safe feature flags, backward-compatible migrations and business-metric alerts. Each action needs an owner and a verification method.
