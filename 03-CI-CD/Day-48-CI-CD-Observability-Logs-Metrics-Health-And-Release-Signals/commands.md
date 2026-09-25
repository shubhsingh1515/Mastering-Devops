# Day 48 - Commands: CI/CD Observability

These examples are for a local or disposable Dockerized MERN environment. Replace container names, ports and endpoints with values from your application.

## 1. Inspect running containers

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
docker stats --no-stream
```

PowerShell:

```powershell
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
docker stats --no-stream
```

Container status helps identify restarts and resource pressure, but `Up` does not prove application correctness.

## 2. Check liveness and readiness

```bash
curl -i -fsS http://localhost:3000/health
curl -i -fsS http://localhost:3000/ready
```

PowerShell:

```powershell
Invoke-WebRequest http://localhost:3000/health
Invoke-WebRequest http://localhost:3000/ready
```

Confirm that readiness changes appropriately when a required dependency is unavailable. Do not treat a successful liveness response as proof that checkout works.

## 3. Inspect recent application logs

```bash
docker logs --tail 200 mern-api
```

Follow logs during a test:

```bash
docker logs --follow mern-api
```

PowerShell filtering:

```powershell
docker logs --tail 500 mern-api 2>&1 | Select-String 'error|request_id|status_code|duration_ms'
```

Avoid printing logs that may contain secrets in shared terminals or tickets.

## 4. Find a request by correlation ID

```bash
docker logs mern-api 2>&1 | grep '7fa92c'
```

PowerShell:

```powershell
docker logs mern-api 2>&1 | Select-String '7fa92c'
```

The same request ID should appear in the edge, API and downstream logs when propagation is configured.

## 5. Exercise a safe request

```bash
curl -i -H 'X-Request-ID: demo-48-001' http://localhost:3000/api/products
```

Check that the response includes a request ID and that the application log contains the same ID. In production, validate and sanitize externally supplied IDs before accepting them.

## 6. Test a health endpoint repeatedly

```bash
for i in $(seq 1 10); do
  curl -fsS http://localhost:3000/health || exit 1
done
```

PowerShell:

```powershell
1..10 | ForEach-Object {
  Invoke-WebRequest http://localhost:3000/health | Out-Null
}
```

This checks basic availability only. It is not a substitute for route-level smoke tests or business validation.

## 7. Measure a simple endpoint locally

```bash
curl -s -o /dev/null -w 'status=%{http_code} total=%{time_total}s\n' http://localhost:3000/health
```

PowerShell timing:

```powershell
$elapsed = Measure-Command { Invoke-WebRequest http://localhost:3000/health | Out-Null }
$elapsed.TotalMilliseconds
```

For real release decisions, use a metrics system that calculates percentiles over enough requests. Do not infer p95 from one request.

## 8. Inspect image and release identity

```bash
docker inspect --format '{{.Name}} -> {{.Config.Image}}' mern-api
docker image inspect registry.example.com/mern-api:2.4.0 --format '{{.RepoDigests}}'
```

Record the immutable image digest with the deployment marker so evidence identifies the exact artifact.

## 9. Inspect MongoDB container health

```bash
docker ps --filter name=mongo --format "table {{.Names}}\t{{.Status}}"
docker logs --tail 100 mongo
```

If your application exposes database metrics, compare query latency, connection-pool usage and errors against the pre-deployment baseline.

## 10. Validate Compose configuration

```bash
docker compose config
docker compose ps
```

A valid Compose file does not prove that the application is ready. Follow it with health checks and safe smoke tests.

## 11. Simulate a dependency-aware readiness failure

Only do this in a disposable environment. Stop the local database, call `/ready`, record the response, then restore the database:

```bash
docker stop mongo
curl -i http://localhost:3000/ready
docker start mongo
curl -i http://localhost:3000/ready
```

The expected result depends on the application contract, but readiness should not claim that an instance is safe for traffic when a required dependency is unavailable.

## 12. Preserve incident evidence

```bash
mkdir -p incident-evidence-48
docker logs mern-api > incident-evidence-48/mern-api.log
docker inspect mern-api > incident-evidence-48/container.json
docker stats --no-stream > incident-evidence-48/resources.txt
```

PowerShell:

```powershell
New-Item -ItemType Directory -Force incident-evidence-48 | Out-Null
docker logs mern-api | Out-File incident-evidence-48/mern-api.log
docker inspect mern-api | Out-File incident-evidence-48/container.json
docker stats --no-stream | Out-File incident-evidence-48/resources.txt
```

Check the evidence for secrets before sharing it. In a production platform, export the relevant dashboard time range, logs and traces using the platform's access-controlled tools.
