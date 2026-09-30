# Day 53 Commands: Health Checks, Recovery and Graceful Operations

These commands are generic Docker and shell examples. Adapt service names, URLs and orchestration commands to the project. Do not expose secrets in command output.

## 1. Inspect Container and Restart State

```bash
docker compose ps
docker compose ps --format json
docker inspect <container> --format '{{.State.Status}} restarts={{.RestartCount}} exit={{.State.ExitCode}}'
docker compose events --since=15m
```

Inspect resource usage without assuming it proves application health:

```bash
docker stats --no-stream
```

## 2. Probe Liveness and Readiness

```bash
curl --fail-with-body --show-error --silent https://api.example.com/health
curl --fail-with-body --show-error --silent https://api.example.com/ready
```

Inspect status codes explicitly:

```bash
curl --include --silent https://api.example.com/health
curl --include --silent https://api.example.com/ready
```

A healthy process may return:

```text
/health -> 200
/ready  -> 503
```

That means the process is alive but should not receive traffic.

## 3. Test from Inside the API Container

```bash
docker compose exec api node -e "fetch('http://localhost:3000/health').then(async r => console.log(r.status, await r.text()))"
docker compose exec api node -e "fetch('http://localhost:3000/ready').then(async r => console.log(r.status, await r.text()))"
```

Check Docker DNS and the MongoDB port:

```bash
docker compose exec api getent hosts mongodb
docker compose exec api sh -c 'nc -zv mongodb 27017'
docker network inspect <project>_default
```

Inside the API container, `localhost` refers to the API container, not MongoDB.

## 4. Inspect Health-Check Configuration

```bash
docker inspect <container> --format '{{json .Config.Healthcheck}}'
docker compose config
```

For a Compose health check, a conceptual configuration is:

```yaml
healthcheck:
  test: ["CMD-SHELL", "wget -q -O - http://localhost:3000/health || exit 1"]
  interval: 10s
  timeout: 3s
  retries: 3
  start_period: 20s
```

Do not use readiness as liveness unless restarting on dependency failure is truly intended.

## 5. View Logs Around a Failure

```bash
docker compose logs --timestamps --since=15m api
docker compose logs --timestamps --since=15m mongodb
docker compose logs --timestamps --since=15m load-balancer
```

Filter structured events when logs are JSON:

```bash
docker compose logs --no-color api | jq 'select(.event == "readiness_failed")'
docker compose logs --no-color api | jq 'select(.errorType == "MongoServerSelectionError")'
docker compose logs --no-color api | jq 'select(.signal == "SIGTERM")'
```

## 6. Inspect Restart Policy and History

```bash
docker inspect <container> --format '{{.HostConfig.RestartPolicy.Name}}'
docker inspect <container> --format '{{.State.StartedAt}} {{.State.FinishedAt}} {{.State.ExitCode}}'
docker compose logs --timestamps --since=30m api
```

Repeated starts with the same non-zero exit code indicate a possible restart loop. Preserve logs before removing the container.

## 7. Check the Dependency Without Printing Credentials

```bash
docker compose exec api printenv MONGO_URI
```

Use this only in a controlled terminal because it may print secrets. Prefer a safe validation script that reports presence, hostname format and connectivity without printing the full URI. Never paste credentials, tokens or connection strings into incident channels.

## 8. Controlled Recovery Actions

Use the project's approved deployment process. Generic examples:

```bash
docker compose up -d --no-deps api
docker compose up -d --no-deps --scale api=3 api
docker compose restart api
```

Do not restart every replica for a single-instance readiness failure. Record the reason, timestamp and expected result for every recovery action.

## 9. Observe a Rolling Replacement

```bash
docker compose ps
docker compose logs --timestamps --follow api
```

A replacement should follow this sequence:

```text
container starts
  -> liveness passes
  -> initialization completes
  -> readiness passes
  -> traffic is enabled
```

## 10. Validate Graceful Shutdown

```bash
docker stop --time=15 <container>
docker compose logs --since=2m api
```

Look for evidence that the service received `SIGTERM`, stopped new work, completed active requests, closed resources and exited cleanly. Confirm that no requests were abruptly terminated during the test.

## 11. Recovery Checklist

```text
[ ] /health returns 200
[ ] /ready returns 200 continuously
[ ] Load balancer excludes unready instances
[ ] Restart count is stable
[ ] Error rate is acceptable
[ ] Latency is acceptable
[ ] MongoDB connections are healthy
[ ] Representative user workflow succeeds
[ ] No duplicate orders or charges occurred
[ ] Logs and recovery timestamps are recorded
```
