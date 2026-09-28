# Day 51 Interview Questions: Centralized Logging and Alerting

## 1. Why centralize logs?

Centralized logging collects events from multiple services and instances into one searchable system. It makes distributed incident investigation faster and more reliable than searching hosts one at a time.

## 2. What is the collector -> storage -> search model?

Applications emit events, a collector gathers and parses them, a central store retains them, and search, dashboards and alerting systems turn the stored events into operational evidence and notifications.

## 3. Why should containers generally write to stdout and stderr?

Containers are often ephemeral and replaced during deployment or scaling. The runtime or orchestrator can collect standard streams consistently, while files inside a container may disappear with the container.

## 4. What is structured logging?

Structured logging records stable fields, usually as JSON, instead of relying only on free-form messages. Fields such as `service`, `version`, `requestId`, `status` and `errorType` make filtering and aggregation predictable.

## 5. What is the difference between a log and an alert?

A log records evidence that something happened. An alert notifies someone about a meaningful condition that requires action. A single 500 log may be evidence; a sustained 5xx rate above a threshold may justify an alert.

## 6. What makes an alert actionable?

It has a meaningful threshold, duration and scope, identifies the affected service and environment, names the responder and provides enough context or links to begin investigation.

## 7. What is alert fatigue?

Alert fatigue occurs when responders receive too many low-value or non-actionable notifications. They may begin ignoring alerts, which increases the chance that a real incident is missed or handled late.

## 8. Why use traffic guards on percentage alerts?

A percentage based on only a few requests can be statistically misleading. A minimum request-volume condition prevents low traffic from producing noisy pages.

## 9. Why monitor business metrics?

Infrastructure can appear healthy while users cannot complete important workflows. Checkout success, login success and payment completion can reveal customer impact that CPU, memory and basic 5xx metrics miss.

## 10. Why not keep every log forever?

Log volume increases storage, indexing, network and search costs. Retention should balance debugging, security, compliance, business needs and cost. High-value events may be retained longer than routine successful requests.

## 11. What should be sampled or retained?

Many systems retain 100% of errors, security events and critical audit events, while sampling routine successful requests. The exact policy depends on the investigation and compliance requirements.

## 12. What must not be logged?

Do not log passwords, JWTs, cookies, authorization headers, API keys, database credentials, payment-card data or unnecessary complete request bodies.

## 13. How does a request ID help?

It correlates events from Nginx, API replicas, workers and dependencies so an investigator can reconstruct one request path across distributed components.

## 14. Why is `localhost` often wrong for MongoDB in Docker Compose?

Inside the API container, `localhost` refers to that API container. MongoDB is a different container, so the API normally connects through the MongoDB service name on the shared Docker network.

## 15. What should happen after an API 5xx alert?

Confirm the signal and scope, check deployments and configuration changes, identify affected routes, search structured logs, correlate request IDs, inspect dependencies, determine blast radius and mitigate by pausing, disabling an isolated feature or rolling back when appropriate.