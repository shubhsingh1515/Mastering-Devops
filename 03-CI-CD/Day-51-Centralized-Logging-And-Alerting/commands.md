# Day 51 Commands: Logging and Alert Investigation

These examples are generic Docker and shell commands. Adapt container names, labels and log-query syntax to the platform used by your project.

## 1. Read Container Logs

```bash
docker compose logs --timestamps --tail=200 api
docker compose logs --timestamps --since=10m api
docker compose logs --timestamps --since=10m worker
```

Follow a service while reproducing a problem:

```bash
docker compose logs --follow --timestamps api
```

## 2. Filter Structured Events

If each line is a JSON event, use `jq` instead of searching only human-readable text:

```bash
docker compose logs --no-color api | jq 'select(.level == "error")'
docker compose logs --no-color api | jq 'select(.status >= 500)'
docker compose logs --no-color api | jq 'select(.event == "order_creation_failed")'
docker compose logs --no-color api | jq 'select(.requestId == "83ad91")'
```

If the runtime adds a prefix before the JSON, configure the collector to parse the JSON payload before applying field queries.

## 3. Inspect Container and Network Context

```bash
docker compose ps
docker inspect <api-container>
docker network ls
docker network inspect <project>_default
```

Check the environment without printing secrets into a ticket or chat message:

```bash
docker compose exec api printenv MONGO_URI
```

Use caution with this command. Do not paste a credential-bearing URI into shared output. Prefer checking the hostname and configuration source separately.

## 4. Test MongoDB Name Resolution From the API Container

```bash
docker compose exec api getent hosts mongodb
docker compose exec api sh -c 'nc -zv mongodb 27017'
```

In Compose, `localhost` from inside the API container points to the API container. Use the actual MongoDB service name on the shared network.

## 5. Correlate a Request

First find a representative request ID:

```bash
docker compose logs --no-color api | jq -r 'select(.status >= 500) | .requestId' | head
```

Then search all relevant services for that ID:

```bash
docker compose logs --no-color nginx api worker | jq 'select(.requestId == "83ad91")'
```

## 6. Check Image and Deployment Version

```bash
docker compose images
docker image inspect <image>:<tag> --format '{{.Id}}'
git log -1 --oneline
```

Compare the deployed image or commit identifier with the version recorded in the structured events.

## 7. Preserve Evidence Before Cleanup

```bash
docker compose logs --no-color --since=30m > incident-logs.txt
docker compose ps > incident-containers.txt
```

Restrict access to captured files and redact sensitive data before sharing them. Do not use log collection commands as a reason to expose tokens, cookies, passwords or connection strings.

## 8. Investigation Sequence

```text
Confirm alert
  -> establish time window and scope
Check deployment/configuration timeline
  -> identify recent changes
Filter structured events
  -> status, event, errorType, version
Correlate request IDs
  -> trace representative failures
Inspect dependencies
  -> MongoDB, DNS, network, external providers
Mitigate
  -> pause rollout, disable isolated feature or roll back
Validate
  -> error rate, latency, business success and logs
```