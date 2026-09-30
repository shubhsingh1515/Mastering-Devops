# DevOps Mentorship Program - Day 51

## Phase 3: CI/CD -> Production Operations

### Centralized Logging and Alerting Architecture

**Level:** Intermediate -> Professional   
**Focus:** Centralized logs, structured events, alert design, log retention, alert fatigue and MERN production incident response

Yesterday, Day 50, was the weekly revision and assessment. Today we resume the syllabus with a production question:

> When a MERN application runs across several containers or servers, how do you collect logs in one place and turn important failures into actionable alerts?

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain centralized logging architecture.
- Describe the collector -> storage -> search model.
- Explain why containers generally emit logs to stdout and stderr.
- Design structured JSON logs for a MERN application.
- Distinguish logs, dashboards and alerts.
- Design useful production alerts without creating alert fatigue.
- Choose a sensible log-retention and sampling policy.
- Protect logs from secrets and unnecessary personal data.
- Investigate an alert from initial scope through root cause and mitigation.
- Answer centralized logging interview questions.

---

## 2. Why Centralize Logs?

Imagine that production contains:

```text
Nginx
API-1
API-2
API-3
Worker-1
MongoDB
```

A customer reports that checkout failed at 2:15 PM. Without centralized logging, an engineer may have to connect to several machines and search each local log directory. That is slow, inconsistent and easy to get wrong.

With centralized logging, the investigation starts from one searchable system:

```text
Nginx ----|
API-1 ----|
API-2 ----|--> Log collector --> Central log store --> Search
API-3 ----|                                      |
Worker ---|                                      +--> Dashboards
MongoDB --|                                      +--> Alerts
```

Centralization does not automatically solve an incident. It gives responders a common place to find evidence across instances and services.

---

## 3. The Four-Part Architecture

Use this model:

```text
Application -> Collection -> Storage -> Search / Alerting
```

### 3.1 Application

The application emits events to stdout and stderr. A Node.js API should produce structured events such as:

```json
{
  "level": "error",
  "event": "order_creation_failed",
  "service": "mern-api",
  "environment": "production",
  "version": "9f72c1",
  "requestId": "83ad91",
  "route": "/api/orders",
  "status": 500,
  "errorType": "MongoServerSelectionError",
  "durationMs": 842
}
```

### 3.2 Collection

A logging agent, runtime logging driver or platform collector receives events from containers, hosts and managed services. The application should not need to know which vendor or storage engine receives the event later.

### 3.3 Storage

The central system stores logs for a defined period. Storage policy should consider debugging, security, compliance, business requirements, access control and cost.

### 3.4 Search and alerting

Responders search, filter, group and correlate events. Alert rules evaluate metrics or log-derived signals and notify a person or response system when a meaningful condition needs action.

---

## 4. Why Containers Use stdout and stderr

Containers are replaceable and often ephemeral. Files written inside a container can disappear when the container is recreated, and local files are difficult to aggregate across replicas.

The usual flow is:

```text
Node.js / Express
       |
       v
stdout or stderr
       |
       v
Container runtime / orchestrator
       |
       v
Collector
       |
       v
Central log store
```

Use stdout for normal structured events and stderr for errors when the platform distinguishes the streams. Do not depend on log files inside an ephemeral container for durable retention. File-based logging may still be appropriate for a deliberate host-level agent design, but it should be an explicit platform decision.

---

## 5. Structured Logs

Plain text is readable but difficult to query reliably:

```text
Order failed
```

Structured JSON carries fields that machines and people can filter:

```json
{
  "timestamp": "2026-09-28T14:15:03.000Z",
  "level": "error",
  "service": "mern-api",
  "environment": "production",
  "version": "9f72c1",
  "requestId": "83ad91",
  "event": "order_creation_failed",
  "route": "/api/orders",
  "status": 500,
  "errorType": "MongoServerSelectionError",
  "dependency": "mongodb",
  "durationMs": 842
}
```

Useful standard fields include:

| Field | Purpose |
|---|---|
| `timestamp` | When the event occurred, preferably in UTC |
| `level` | `debug`, `info`, `warn` or `error` |
| `service` | Component that emitted the event |
| `environment` | Environment such as staging or production |
| `version` | Commit, image or deployment identifier |
| `requestId` | Correlation value for one request |
| `event` | Stable operation name |
| `route` | Normalized route template |
| `status` | HTTP or operation result |
| `durationMs` | Time spent on the operation |
| `errorType` | Safe, searchable error classification |

Consistent field names matter. Do not alternate between `requestId`, `request_id` and `correlationId` without a documented reason.

---

## 6. Request IDs and Correlation

Generate or accept a validated request ID at the edge, then propagate it through the API and important downstream operations:

```text
Nginx requestId=83ad91
          |
          v
Express requestId=83ad91
          |
          v
Order service requestId=83ad91
          |
          v
MongoDB / payment logs requestId=83ad91
```

Searching for `requestId=83ad91` should reconstruct the request path. A request ID is for correlation, not authentication. It must not be treated as a secret or authorization credential.

---

## 7. Logs, Dashboards and Alerts

These tools have different jobs:

| Tool | Main question | Example |
|---|---|---|
| Log | What happened in this event? | One order failed with a database timeout |
| Metric | How is the system behaving over time? | 5xx rate is 7.2% |
| Dashboard | What is the current system state? | Error rate, latency and traffic together |
| Alert | Does someone need to act now? | 5xx rate exceeds 5% for 5 minutes |

A log is evidence. A metric shows a trend. An alert creates urgency.

Not every error deserves a page. Individual errors can be expected, transient or low impact. Alerts should generally use rates, thresholds, sustained conditions, dependency state or business outcomes.

---

## 8. What Makes a Good Alert?

A useful alert is:

- **Actionable:** a responder knows what decision or action is possible.
- **Specific:** it identifies the service, environment and condition.
- **Relevant:** it represents user or operational impact.
- **Timely:** it arrives while mitigation can limit harm.
- **Contextual:** it includes links or information needed for triage.

Weak alert:

```text
CPU changed
```

Stronger alert:

```text
Production API 5xx rate > 5% for 5 minutes
AND request volume >= 100 requests/minute
```

The traffic guard prevents a tiny number of requests from producing a misleading percentage.

---

## 9. Alert Fatigue

Alert fatigue happens when responders receive too many low-value notifications. The danger is operational, not merely annoying:

```text
Non-actionable alerts -> ignored notifications -> missed incident -> larger impact
```

Before adding an alert, define who responds, what action they can take, what evidence they need and whether the condition is already covered by a dashboard. Tune thresholds using real traffic and incident history.

---

## 10. Recommended MERN Alert Set

Useful starting alerts include:

1. API 5xx rate above 5% for 5 minutes.
2. API p95 latency above 1 second for 10 minutes.
3. Readiness or health checks failing continuously.
4. Repeated MongoDB connection failures.
5. Checkout success rate dropping significantly below its baseline.

The last alert is a business-health signal. CPU, memory and 5xx rate can look normal while customers are unable to complete checkout.

The complete alert definitions are in [ALERTS.md](ALERTS.md).

---

## 11. Retention, Volume and Sampling

Keeping every log forever is usually expensive and often makes investigations noisier. A retention policy should balance:

- Debugging and incident-response needs.
- Security and compliance requirements.
- Business and audit requirements.
- Search and indexing cost.
- Storage capacity.

An example policy might be:

```text
Hot searchable logs: 7 days
Archived logs:       30-90 days
Critical audit logs: according to policy
```

High-volume systems may sample routine successful requests while retaining all errors, security events and critical audit events. Sampling must be deliberate; do not discard the evidence required to investigate an incident.

---

## 12. Security and Sensitive Data

Logs can contain user IDs, IP addresses, URLs and operational metadata, so they need access control, encryption, retention rules, sensitive-data filtering and auditability.

Never log passwords, JWTs, cookies, authorization headers, API keys, database credentials, payment-card data or complete request bodies. Prefer an allowlist of safe fields. Redaction is defense in depth, not permission to log secrets first.

---

## 13. Incident Investigation Flow

When an alert fires:

```text
Alert
  |
  v
Confirm signal and scope
  |
  v
Check dashboard and deployment timeline
  |
  v
Identify affected service, route and user journey
  |
  v
Search logs by status, event and error type
  |
  v
Follow representative request IDs
  |
  v
Inspect dependencies and configuration
  |
  v
Mitigate, stop rollout or roll back
  |
  v
Validate recovery and preserve evidence
```

Do not restart everything as the first response. Establish the scope, preserve evidence and choose the smallest effective mitigation.

---

## 14. MERN Incident Example

Alert:

```text
Production API 5xx > 5%
```

Dashboard:

```text
5xx:    8.1%
p95:    2.1s
CPU:   43%
Memory: 51%
Traffic: normal
```

Logs:

```json
{
  "event": "order_creation_failed",
  "errorType": "MongoServerSelectionError",
  "service": "mern-api",
  "version": "10a7c1",
  "requestId": "83ad91"
}
```

The version was deployed seven minutes ago. That creates a strong release-correlation hypothesis:

```text
Deployment -> MongoDB-related errors -> 5xx spike and latency increase
```

Next steps:

1. Stop or pause the rollout.
2. Check the MongoDB hostname, credentials, network and service health.
3. Check whether the new image uses `localhost` instead of the MongoDB service name.
4. Roll back if customer impact is significant and rollback is safe.
5. Correct the configuration and validate in staging.
6. Redeploy and verify error rate, latency and checkout success.

In Docker Compose, `localhost` inside the API container means the API container itself. A MongoDB service commonly needs a URI such as `mongodb://mongodb:27017/mern`, using the actual service name and network.

---

## 15. Interview Model Answer

> I would standardize structured JSON logs and emit them to stdout and stderr so the container runtime or orchestrator can collect them. A central pipeline would store searchable logs with defined retention and access controls. I would retain all errors, security events and critical audit events while sampling or shortening retention for low-value high-volume events where appropriate. Alerts would be based on meaningful rates, thresholds, sustained conditions and business outcomes rather than every individual error. Secrets and sensitive data would be redacted or excluded. This keeps investigations reliable without creating unnecessary cost or alert fatigue.

---

## 16. Review Questions

1. Why centralize logs from multiple services?
2. Why should containers generally write to stdout and stderr?
3. What makes JSON logs easier to operate?
4. What is the difference between a log and an alert?
5. Why is alert fatigue dangerous?
6. Why can checkout success be a better incident signal than CPU?
7. What determines log retention?
8. Which events should be retained most aggressively?
9. How does a request ID help an incident investigator?
10. Why can `localhost` be wrong in a containerized MERN deployment?

---

## 17. Key Takeaway

```text
Deploy
  |
  v
Observe
  |
  v
Detect
  |
  v
Investigate
  |
  v
Mitigate
  |
  v
Recover and learn
```

Logs are evidence. Metrics provide trends. Alerts create urgency. A mature MERN production system connects all three to a response process that protects users and limits blast radius.

Next lesson: incident response and production troubleshooting.