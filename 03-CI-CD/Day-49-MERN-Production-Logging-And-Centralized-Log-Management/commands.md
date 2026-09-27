# Day 49 Commands: Production Logging Investigation

These commands are examples for local Docker and Linux-based investigation. Adapt service names and log backends to your environment.

## 1. Inspect Container Logs

```bash
docker compose logs --tail=100 api
docker compose logs --tail=100 --follow api
docker compose logs --since=30m nginx api mongodb
```

Use `--follow` during an active investigation, but avoid leaving noisy sessions running indefinitely.

## 2. Read JSON Logs

If each log line is valid JSON and `jq` is installed:

```bash
docker compose logs --no-color api | jq -c .
docker compose logs --no-color api | jq 'select(.level == "error")'
docker compose logs --no-color api | jq 'select(.status == 500)'
```

Filter a known request ID:

```bash
docker compose logs --no-color api | jq 'select(.requestId == "7fa92c")'
```

## 3. Search Plain Text Logs Carefully

For text logs or mixed output:

```bash
docker compose logs --no-color api | rg 'status[=: ]+500'
docker compose logs --no-color api | rg 'MongoServerSelectionError|database_query_failed'
docker compose logs --no-color api | rg '7fa92c'
```

Do not paste raw production logs into tickets or chat without checking for secrets and personal data.

## 4. Inspect Service and Network Names

```bash
docker compose ps
docker compose config --services
docker network ls
docker compose config
```

Inside a Compose network, the MongoDB hostname is commonly the service name, not `localhost`:

```text
mongodb://mongodb:27017/mern
```

Verify the actual service name and configuration before changing `MONGO_URI`.

## 5. Check Environment Variable Names Without Exposing Values

List variable names from a running container while avoiding secret values in output:

```bash
docker compose exec api sh -lc 'printenv | cut -d= -f1 | sort'
```

Do not run `printenv` or `env` in a shared terminal if it will expose credentials. Never paste database passwords, tokens or API keys into logs.

## 6. Check Basic Connectivity from the API Container

```bash
docker compose exec api getent hosts mongodb
docker compose exec api sh -lc 'nc -zv mongodb 27017'
```

The availability of `getent` or `nc` depends on the image. Use an approved diagnostic image or platform tooling when these commands are unavailable.

## 7. Check Container Health and Restart History

```bash
docker inspect --format '{{.Name}} {{.State.Status}} restart={{.RestartCount}}' $(docker compose ps -q)
docker compose ps
```

A running container is not proof that the application is healthy. Check readiness, dependencies and user-facing behavior as well.

## 8. Follow an Incident Search Sequence

```text
1. Confirm the metric and time window
2. Check deployment and configuration changes
3. Identify affected route and service
4. Filter status=500
5. Group by event and error type
6. Select request IDs
7. Search all correlated services
8. Check MongoDB and external providers
9. Mitigate: stop rollout, disable a flag or roll back
10. Preserve evidence and document the timeline
```

## 9. Example Central Log Queries

The syntax depends on the log platform, but the intent is consistent:

```text
status = 500
```

```text
event = "order_creation_failed" AND environment = "production"
```

```text
errorType = "MongoServerSelectionError"
```

```text
requestId = "7fa92c"
```

```text
service = "mern-api" AND level = "error" | group by errorType
```

## 10. Safe Logging Checklist

Before sharing or storing a diagnostic event, verify that it does not contain:

```text
password
JWT
Authorization header
API key
Database URI with credentials
Credit card number
Private key
Unnecessary request body
```
