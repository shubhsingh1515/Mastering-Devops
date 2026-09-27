# MERN Production Logging Standard

## Purpose

This document defines the minimum logging and incident-investigation standard for the production MERN application.

## Required JSON Event Shape

Every application event should be one JSON object on one line and should include the fields relevant to the event:

```json
{
  "timestamp": "2026-09-26T16:30:00.000Z",
  "level": "info",
  "service": "mern-api",
  "environment": "production",
  "version": "8f31c2a",
  "requestId": "7fa92c",
  "method": "POST",
  "route": "/api/orders",
  "status": 201,
  "durationMs": 184,
  "event": "request_completed"
}
```

Error events should add an event name and safe error type:

```json
{
  "timestamp": "2026-09-26T16:31:12.000Z",
  "level": "error",
  "service": "mern-api",
  "environment": "production",
  "version": "8f31c2a",
  "requestId": "9ac42e",
  "event": "order_creation_failed",
  "errorType": "MongoServerSelectionError",
  "dependency": "mongodb"
}
```

## Log Levels

- `DEBUG`: detailed diagnostics; disabled or restricted in normal production operation.
- `INFO`: normal lifecycle and successful operation events.
- `WARN`: unusual conditions, retries or approaching limits.
- `ERROR`: failed operations, dependency failures and unhandled application errors.

## Request ID Policy

- Assign a request ID at the edge of the request.
- Propagate it to downstream services and important asynchronous work where possible.
- Include it in access logs and application errors.
- Do not use a request ID as an authorization credential.
- Preserve a trusted upstream correlation value only after applying the platform's validation policy.

## Access Logs

Access events should capture:

- HTTP method.
- Normalized route template, not an unbounded raw URL when possible.
- Status code.
- Duration.
- Request ID.
- Service and deployment version.

Avoid query strings and headers unless there is a documented, redacted need.

## Application Logs

Application events should describe the operation and result. Examples include:

```text
order_creation_started
order_creation_completed
order_creation_failed
database_query_failed
payment_provider_timeout
```

Include safe context such as an operation name, dependency, error type and controlled identifier. Do not dump full request, response, user or database objects.

## Sensitive-Data Policy

Never log:

- Passwords or password reset values.
- JWTs, session tokens or cookies.
- Authorization headers.
- API keys, private keys or database credentials.
- Credit card numbers or payment authentication data.
- Full database connection strings containing credentials.
- Unnecessary personal or health information.

Use allowlists for fields that may be logged. Redaction is a defense in depth measure, not permission to log secrets. Review logging changes as part of code review.

## Container Strategy

Applications write structured logs to stdout and stderr. Docker or the orchestrator forwards them to the configured collector. Do not depend on files inside an ephemeral container for durable retention.

```text
MERN container
  |
  v
stdout / stderr
  |
  v
Runtime logging driver
  |
  v
Collector
  |
  v
Central log store
```

## Centralized Architecture

```text
Nginx ----|
API-1 ----|
API-2 ----|--> Collector --> Central store --> Search / Dashboard / Alert
Worker ---|
MongoDB --|
```

The central platform must define retention, access control, encryption, cost limits and deletion requirements. Restrict production logs to authorized responders and audit access where required.

## Incident Investigation Procedure

When the 5xx rate exceeds the agreed threshold:

1. Confirm the metric, threshold, time range and affected environment.
2. Check the deployment, configuration and infrastructure timeline.
3. Identify affected routes, instances and user journeys.
4. Search structured logs for `status=500` and relevant error events.
5. Group failures by `event`, `errorType`, service and dependency.
6. Select representative `requestId` values.
7. Search the same IDs across Nginx, API, worker, database and external-service logs.
8. Check MongoDB connectivity, DNS, service names, credentials, network policy and provider health.
9. Determine blast radius and business impact.
10. Stop the rollout, disable an isolated feature or roll back when that is the safest mitigation.
11. Preserve relevant evidence and record a timeline.
12. Complete root-cause analysis and follow-up actions after user impact is controlled.

## Example Docker Networking Check

Inside an API container, `localhost` refers to the API container. It does not automatically refer to MongoDB:

```text
Incorrect in many Compose deployments:
MONGO_URI=mongodb://localhost:27017/mern

Possible Compose service reference:
MONGO_URI=mongodb://mongodb:27017/mern
```

Verify the actual Compose service name, network and credentials before changing the value.

## Review Checklist

- [ ] JSON events are one line and machine-readable.
- [ ] Timestamp and level are present.
- [ ] Service, environment and version are available.
- [ ] Request IDs correlate request events.
- [ ] Access and application events are distinguishable.
- [ ] Error events include safe error types.
- [ ] Secrets and sensitive data are excluded.
- [ ] Containers write to stdout/stderr.
- [ ] Centralized collection and retention are configured.
- [ ] Incident responders can search by status, event, error type and request ID.
- [ ] Rollback and evidence-preservation steps are documented.
