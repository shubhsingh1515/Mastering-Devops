# Day 52 Commands: Incident Investigation and Recovery

These examples are generic Docker and shell commands. Adapt container names, labels and log-query syntax to the platform used by your project. Capture evidence before changing or restarting production components.

## 1. Establish Current Scope

```bash
docker compose ps
docker compose top api
docker stats --no-stream
```

Check recent service output:

```bash
docker compose logs --timestamps --since=15m api
docker compose logs --timestamps --since=15m worker
docker compose logs --timestamps --since=15m nginx
```

## 2. Filter Structured Failures

If each line is JSON, use `jq` rather than searching only human-readable text:

```bash
docker compose logs --no-color api | jq 'select(.level == "error")'
docker compose logs --no-color api | jq 'select(.status >= 500)'
docker compose logs --no-color api | jq 'select(.event == "order_creation_failed")'
docker compose logs --no-color api | jq 'select(.errorType == "PaymentTimeout")'
```

Find representative failed request IDs:

```bash
docker compose logs --no-color api | jq -r 'select(.status >= 500) | .requestId' | head
```

## 3. Correlate a Request Across Services

```bash
docker compose logs --no-color nginx api worker | jq 'select(.requestId == "83ad91")'
```

Search the central log platform for the same request ID, release version and event name. A request ID is for correlation, not authentication.

## 4. Inspect Deployment and Image Context

```bash
docker compose images
docker image inspect <image>:<tag> --format '{{.Id}}'
git log -1 --oneline
```

Compare the image or commit identifier with the `version` field in structured events.

## 5. Inspect Safe Configuration Metadata

Do not paste secrets into a ticket or chat message. Check the variable name and hostname separately when possible:

```bash
docker compose exec api printenv MONGO_URI
docker compose exec api printenv PAYMENT_PROVIDER_URL
```

If output includes credentials, do not share it. Prefer a controlled validation command that reports only whether the value is present, parseable and reachable.

## 6. Test Container Network Resolution

In Docker Compose, `localhost` inside the API container points to the API container. Use the actual database service name on the shared network:

```bash
docker compose exec api getent hosts mongodb
docker compose exec api sh -c 'nc -zv mongodb 27017'
docker network ls
docker network inspect <project>_default
```

Use equivalent DNS and connectivity checks for the payment provider, but do not bypass production controls or send real payment requests while testing.

## 7. Preserve Evidence Before Cleanup

```bash
docker compose logs --no-color --since=30m > incident-logs.txt
docker compose ps > incident-containers.txt
docker compose config > incident-compose-config.txt
```

Restrict access to captured files and redact tokens, cookies, passwords, connection strings and payment data before sharing them. Keep the incident timestamp and release identifier with the evidence.

## 8. Stop or Roll Back Deliberately

Use the deployment system's approved procedure. Generic examples:

```bash
docker compose up -d --no-deps api
docker compose up -d --no-deps --scale api=3 api
```

Do not substitute an untested command for the project's release procedure. Before rollback, check application and database compatibility. If the new feature is isolated, disable the feature flag first.

## 9. Validate Recovery

```bash
curl -fsS https://example.invalid/health
curl -fsS https://example.invalid/ready
```

Then validate metrics and a safe representative workflow:

```text
5xx rate decreases
p95 latency returns to baseline
readiness remains healthy
PaymentTimeout events stop
Checkout success recovers
No duplicate orders or charges appear
```

## 10. Investigation Sequence

```text
Confirm alert
  -> establish time window and scope
Assess customer and business impact
  -> checkout, payment, order and data integrity
Check deployment and feature-flag timeline
  -> identify recent changes
Filter structured events
  -> status, event, errorType, version
Correlate request IDs
  -> trace representative failures
Inspect dependencies
  -> payment provider, MongoDB, DNS and network
Mitigate
  -> stop rollout, disable isolated feature or roll back safely
Validate
  -> technical metrics, business metrics and logs
Document
  -> timeline, root cause and corrective actions
```
