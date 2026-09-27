# DevOps Mentorship Program - Day 49

## Phase 3: CI/CD -> Production Operations

### MERN Production Logging and Centralized Log Management

**Level:** Intermediate -> Professional  
**Duration:** 25-30 minutes  
**Focus:** Structured logging, request IDs, centralized logs, sensitive-data handling and production incident investigation

Yesterday, Day 48, you learned how to monitor releases using health checks, metrics, logs and business signals. Today we go deeper into logs because when a MERN application fails in production, logs are often the first source of detailed evidence.

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain why production logging matters.
- Design useful Node.js and Express logs.
- Explain structured JSON logging.
- Use request and correlation IDs conceptually.
- Distinguish access logs from application logs.
- Explain centralized log management.
- Avoid logging passwords, tokens and sensitive data.
- Troubleshoot a MERN production incident using logs.
- Answer production logging questions in DevOps interviews.

---

## 2. Why Logs Matter

Suppose production reports an HTTP 500 error. Metrics tell you that the 5xx rate is 4.2%, but metrics alone do not explain why the request failed.

Logs may identify the cause:

```text
MongoServerSelectionError
Payment provider timeout
TypeError: Cannot read properties of undefined
```

Use this mental model:

```text
Metrics
  |
  v
Something is wrong
  |
  v
Logs
  |
  v
What happened?
```

Metrics help detect and measure the problem. Logs provide detailed evidence about the event.

---

## 3. MERN Logging Flow

A production request may travel through several layers:

```text
Browser
  |
  v
Nginx
  |
  v
Node / Express
  |
  v
MongoDB
  |
  v
External API
```

Each layer can produce useful evidence:

| Layer | Useful information |
|---|---|
| Nginx | Request, status code, response time, client-facing errors |
| Express | Route, request ID, status, duration, application error |
| MongoDB | Connection errors, query problems, timeout information |
| External service | Timeout, rejected request, provider error |

The challenge is connecting events from different layers. A request ID makes that possible.

---

## 4. Bad Logs Versus Useful Logs

This log is too vague:

```text
Error happened
```

This is better because it identifies the operation:

```text
POST /api/orders failed
```

A structured event is more useful for machines and people:

```json
{
  "level": "error",
  "event": "order_creation_failed",
  "requestId": "7fa92c",
  "method": "POST",
  "route": "/api/orders",
  "status": 500,
  "durationMs": 842,
  "errorType": "MongoServerSelectionError"
}
```

A good log answers: what happened, when it happened, where it happened, which request was affected and how serious it was.

---

## 5. Structured JSON Logging

Structured logging records fields instead of only a human-oriented sentence:

```text
ERROR Order creation failed for request 7fa92c
```

```json
{
  "timestamp": "2026-09-26T16:31:12.000Z",
  "level": "error",
  "service": "mern-api",
  "environment": "production",
  "version": "8f31c2a",
  "requestId": "9ac42e",
  "event": "order_creation_failed",
  "errorType": "MongoServerSelectionError"
}
```

The JSON event can move through this pipeline:

```text
JSON log
  |
  v
Log collector
  |
  v
Central log store
  |
  v
Search and filtering
  |
  v
Dashboards and alerts
```

Structured fields make queries predictable:

```text
status = 500
event = "order_creation_failed"
requestId = "7fa92c"
```

### Fields worth standardizing

- `timestamp`
- `level`
- `service`
- `environment`
- `version` or deployment identifier
- `requestId` or trace identifier
- `event`
- `method`
- `route`
- `status`
- `durationMs`
- `errorType`

Use consistent field names. Do not alternate between `requestId`, `request_id` and `correlation` without a documented reason.

---

## 6. Request IDs and Correlation

When a request enters the API, generate a request ID if one does not already exist. If a trusted upstream proxy supplies one, validate and propagate it according to your platform policy.

Conceptually:

```text
Nginx
  requestId=7fa92c
      |
      v
Express API
  requestId=7fa92c
      |
      v
Payment service
  requestId=7fa92c
      |
      v
Order creation
  requestId=7fa92c
```

Without correlation, logs from several services are disconnected. With the ID, an incident investigator can search for `7fa92c` and reconstruct the request path.

A request ID is not a replacement for distributed tracing, but it is a practical first step toward correlating events across services.

---

## 7. Log Levels

| Level | Meaning | Examples |
|---|---|---|
| `DEBUG` | Detailed diagnostic information | Development-only payload-free diagnostics |
| `INFO` | Normal significant event | Service started, request completed, deployment version |
| `WARN` | Unusual condition that is not yet fatal | Retry, pool nearing capacity, deprecated configuration |
| `ERROR` | A request or operation failed | Database failure, payment failure, unhandled error |

Production logging should be useful without being noisy. Excessive debug logging increases storage cost, makes searches harder and can expose data accidentally.

---

## 8. Sensitive-Data Policy

Never log:

- Passwords
- JWTs or session tokens
- API keys
- Database credentials
- Credit card numbers
- Authorization headers
- Private keys
- Complete request bodies when they may contain personal or secret data

Do not log entire application objects casually:

```js
console.log(user);
```

The object may contain passwords, tokens, internal fields or personal data. Prefer a small allowlist of operational fields:

```js
logger.info({
  event: 'user_lookup_completed',
  userId,
  requestId,
  result: 'found'
});
```

Log events, not entire application state. If an identifier is needed, prefer a controlled identifier such as `userId` over a complete user document. Follow the organization's privacy, retention and redaction requirements.

---

## 9. Access Logs and Application Logs

An **access log** answers who requested what and how the server responded:

```text
GET /api/products 200 120ms
```

An **application log** answers what the application did while processing the request:

```text
MongoDB query timed out while loading products
```

You usually need both:

```text
Request
  |
  +-- Access log: method, route, status, duration
  |
  +-- Application log: operation, dependency, error and context
```

Access logs help measure traffic and response behavior. Application logs explain internal operations and failures.

---

## 10. Container Logging

For containers, write application logs to standard output and errors to standard error:

```text
Application
  |
  v
stdout / stderr
  |
  v
Docker logging driver
  |
  v
Log collector
```

Containers are ephemeral. If logs exist only in `/app/logs/app.log` inside the container filesystem, they may disappear when the container is replaced. External collection separates log retention from container lifecycle.

The application should still emit one structured event per line so the runtime and collector can parse it reliably.

---

## 11. Centralized Log Management

Consider a deployment with API replicas, Nginx, a worker and MongoDB. SSHing into each machine to investigate an incident does not scale.

```text
API-1  --|
API-2  --|
API-3  --|
Nginx  --|--> Log collector --> Central log store --> Search / Dashboard / Alert
Worker --|
```

Centralization provides:

- One search location across services and replicas.
- Correlation using request IDs.
- Retention and access controls.
- Dashboards and saved queries.
- Alerting based on error patterns.
- Evidence for incident timelines.

Centralized logging does not remove the need for good event design. It makes useful events easier to retain and find.

---

## 12. Production Incident Investigation

Incident alert:

```text
5xx rate > 5%
```

Use an evidence-based sequence:

```text
1. Confirm the metric and time window
2. Check the deployment timeline
3. Identify affected endpoints and instances
4. Search structured logs for status=500
5. Group failures by event and error type
6. Select request IDs for representative failures
7. Inspect correlated dependency logs
8. Determine blast radius and user impact
9. Stop rollout or roll back if necessary
10. Preserve evidence and investigate root cause
```

Example:

```text
5xx: 0.3% -> 5.4%
  |
  v
 event=database_query_failed
  |
  v
 errorType=MongoServerSelectionError
  |
  v
 Check MongoDB connectivity
```

A common container mistake is:

```text
MONGO_URI=mongodb://localhost:27017/mern
```

Inside the API container, `localhost` refers to the API container itself, not the MongoDB container. With Docker Compose, the service name may be used instead:

```text
MONGO_URI=mongodb://mongodb:27017/mern
```

The exact hostname depends on the Compose service and network configuration. Check environment variables, service names, DNS, credentials, network policy and MongoDB health before changing production configuration.

---

## 13. Logs, Metrics and Traces Together

These signals answer different questions:

| Signal | Primary question |
|---|---|
| Metrics | What is changing and how much? |
| Logs | What specific event happened? |
| Traces | Where did the request spend time? |

A strong production investigation connects them:

```text
Metric alert
  |
  v
Scope and impact
  |
  v
Structured log event
  |
  v
Request or trace ID
  |
  v
Dependency and request path
  |
  v
Mitigation and root-cause analysis
```

Do not rely only on logs. Metrics are better for trends and alert thresholds, while traces are better for understanding cross-service request paths.

---

## 14. Interview Takeaways

### Why should applications log to stdout in containers?

Containers are ephemeral, so logs written only to the container filesystem are difficult to retain and aggregate. Structured stdout and stderr allow the runtime and centralized logging infrastructure to collect logs independently of the container lifecycle.

### What makes a good production log?

A good production log is structured, timestamped, correlated with a request or trace ID, contains enough context to diagnose the event, uses an appropriate severity level and avoids sensitive data.

### How would you investigate a sudden 5xx spike?

Start with the error-rate metric to establish scope and impact. Correlate the spike with deployments or infrastructure changes. Search structured logs by status, event and error type, then use request IDs to inspect related dependency events. Mitigate user impact by stopping the rollout or rolling back when justified, and preserve logs for root-cause analysis.

---

## 15. Day 49 Summary

Production logs should provide detailed evidence behind alerts and metrics. For a MERN application, the target model is:

```text
Request
  |
  v
requestId
  |
  v
Nginx -> Express -> MongoDB / External API
  |
  v
Structured logs to stdout/stderr
  |
  v
Centralized search
  |
  v
Incident response
```

The most important habits are to use structured events, propagate request IDs, log at the correct level, send container logs to stdout/stderr, centralize collection and protect sensitive information.

**Next lesson:** production log aggregation and alerting architecture, including log collectors, retention, alert fatigue and incident response.
