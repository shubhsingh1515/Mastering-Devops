# Day 58 Interview Questions

## 1. What is the difference between an SLI, SLO and SLA?

An SLI is a measurement of actual service behavior, such as successful request ratio or p95 latency. An SLO is the internal target for that measurement. An SLA is a formal customer or contractual commitment, often with consequences when it is missed.

## 2. What is an error budget?

An error budget is the amount of unreliability allowed by an SLO. A 99.9% SLO leaves a 0.1% budget for failed requests or unavailability during the defined window.

## 3. Why should an SLO not simply be 100%?

Absolute reliability is usually impractical and disproportionately expensive. A realistic target allows teams to balance resilience work, controlled change and product delivery.

## 4. Why are p95 and p99 more useful than average latency?

An average can hide a small group of extremely slow requests. Percentiles expose tail behavior and show the experience of users affected by slow dependencies, exhausted pools or cold caches.

## 5. What should happen when the error budget is exhausted?

Prioritize restoring reliability, reduce release risk and investigate the cause. A complete deployment freeze is not always necessary, but new changes should use a smaller canary, stronger rollback controls or a feature flag.

## 6. How would you use SLOs in a CI/CD pipeline?

After tests and security checks, deploy progressively. Compare canary availability, latency, error rate and business success metrics with the stable version. Stop or roll back when the canary degrades SLOs or burns the error budget too quickly.

## 7. Why are business SLIs important?

Infrastructure can report healthy while a critical user journey is broken. Checkout success, payment success and order creation success measure whether customers can complete the outcome that matters.

## 8. What is burn rate?

Burn rate describes how quickly a service consumes its error budget relative to the permitted rate. A fast burn indicates that the service may exhaust its budget before the end of the SLO window.

## 9. How should alerts be designed around SLOs?

Alert on sustained user impact and budget consumption. Include the affected SLO, time window, severity, owner, dashboard and immediate response. Avoid paging solely because one infrastructure metric crossed a threshold.

## 10. What is the strongest response when a canary has a 4.5% 5xx rate and stable has 0.2%?

Stop the rollout immediately, preserve logs and deployment metadata, investigate the difference, and roll back if the cause is not quickly understood. Verify recovery against the same SLO metrics before resuming.
