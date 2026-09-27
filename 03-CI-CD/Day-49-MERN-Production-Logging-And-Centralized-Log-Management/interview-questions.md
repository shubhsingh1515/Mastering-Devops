# Day 49 Interview Questions: MERN Production Logging

## 1. What is structured logging?

Structured logging records machine-readable fields, commonly as one JSON object per event, instead of only a free-form message. Fields such as `level`, `service`, `requestId`, `status` and `durationMs` make logs searchable and aggregatable.

## 2. Why are request IDs useful?

A request ID connects events produced by Nginx, the Express API, downstream services and background operations. Searching one ID helps reconstruct a request path during an incident.

## 3. What is the difference between access and application logs?

Access logs describe HTTP requests and responses, such as method, route, status and duration. Application logs describe internal operations, decisions and failures, such as a MongoDB query timeout or payment provider error.

## 4. Why should containers log to stdout and stderr?

Containers are ephemeral. Logs stored only inside a container can disappear when it is replaced. stdout and stderr allow Docker or the orchestration platform to collect and route logs independently of the container filesystem.

## 5. What should never be logged?

Passwords, JWTs, API keys, database credentials, authorization headers, private keys, credit card data and unnecessary personal information should never be logged.

## 6. Why should you avoid logging an entire user object?

The object may contain passwords, tokens, personal data or internal fields. Log a small allowlist of fields needed for the operation, such as a controlled user ID and the result.

## 7. Explain DEBUG, INFO, WARN and ERROR.

`DEBUG` is detailed diagnostic information, usually too noisy for normal production. `INFO` records normal significant events. `WARN` identifies an unusual condition that is not necessarily fatal. `ERROR` indicates a failed operation or serious application problem.

## 8. How would you investigate a sudden 5xx increase?

I would confirm the metric and time window, identify affected routes and instances, correlate the increase with deployments or configuration changes, search structured logs for status 500, group failures by event and error type, then use request IDs to inspect MongoDB and external-service events. I would mitigate user impact by stopping the rollout, disabling an isolated feature or rolling back when justified, then preserve evidence for root-cause analysis.

## 9. Why are centralized logs useful?

They provide one searchable location across replicas and services, support correlation and dashboards, enable alerting, enforce retention and access controls, and make incident timelines easier to build.

## 10. How do logs, metrics and traces complement one another?

Metrics identify trends and scope, logs provide detailed event evidence, and traces show the path and timing of one request across services. Together they support detection, diagnosis and mitigation.

## 11. Why can `mongodb://localhost:27017/mern` fail inside an API container?

Inside the API container, `localhost` refers to that API container. It does not refer to the MongoDB container. In a Docker Compose network, the MongoDB service name is commonly used as the hostname, such as `mongodb://mongodb:27017/mern`.

## 12. What makes a production log good?

It is structured, timestamped, appropriately leveled, searchable, correlated with a request or trace ID, contextual enough to diagnose the event and free of sensitive data.

## 13. Are request IDs a replacement for distributed tracing?

No. Request IDs provide basic event correlation. Distributed tracing adds spans, timing and dependency relationships across a request path. Request IDs are a useful foundation but not a complete tracing system.

## 14. What is log cardinality and why does it matter?

Cardinality is the number of unique values in a field. Extremely unique fields or unbounded values can increase storage and query cost. Use request IDs for correlation, but avoid indexing every high-cardinality field indiscriminately and follow the log platform's design guidance.

## 15. Why should logs not be the only alert source?

Logs can be noisy, delayed or incomplete. Metrics are better for stable thresholds such as error rate and latency, while traces help explain request paths. Alerting should use the signal that best represents the condition.
