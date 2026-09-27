# Day 49 Assignment: Structured MERN Production Logs

## Objective

Design a production logging approach for a MERN API that produces useful, searchable and secure events.

## Scenario

Your API exposes `POST /api/orders`. The request passes through Nginx, Express, MongoDB and a payment provider. A customer reports that an order failed.

Create a short logging design that answers:

1. How is a request ID generated and propagated?
2. What fields are included in a successful access event?
3. What fields are included in an order failure event?
4. Which logs are access logs and which are application logs?
5. How are logs sent from Docker containers to a central system?
6. Which fields must be redacted or excluded?
7. How would you search for one failed request?

## Required Events

Write JSON examples for these events:

### Request completed

Include:

- `timestamp`
- `level`
- `service`
- `environment`
- `version`
- `requestId`
- `method`
- `route`
- `status`
- `durationMs`

### Order creation failed

Include:

- `timestamp`
- `level`
- `service`
- `requestId`
- `event`
- `errorType`
- A safe operation or dependency field

Do not include passwords, JWTs, authorization headers, API keys, payment data or complete request bodies.

## Incident Drill

Your dashboard reports:

```text
5xx rate: 0.3% -> 5.4%
```

Write the investigation steps in order. Your answer must include:

- Checking the deployment timeline.
- Identifying affected routes.
- Searching `status=500`.
- Grouping by event and error type.
- Finding representative request IDs.
- Inspecting MongoDB and external dependency logs.
- Determining blast radius.
- Stopping the rollout or rolling back when appropriate.

## Deliverable

Create or update `OBSERVABILITY.md` in your cumulative MERN project with:

- Structured JSON logging standard.
- Log levels.
- Request ID policy.
- Access and application log policy.
- Sensitive-data policy.
- Container stdout/stderr strategy.
- Centralized logging architecture.
- Incident investigation procedure.

## Completion Criteria

- Every event has a timestamp and severity.
- Request events contain a request ID.
- Errors identify an event and error type.
- Logs are machine-readable JSON.
- No secret or payment data is logged.
- Containers write logs to stdout/stderr.
- The incident procedure prioritizes mitigation and evidence.
