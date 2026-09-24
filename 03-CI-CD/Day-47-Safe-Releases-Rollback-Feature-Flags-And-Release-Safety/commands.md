# Day 47 - Commands: Safe Releases and Feature Flags

Use these commands to inspect release versions, exercise feature flags, compare metrics and practice rollback in a Dockerized MERN environment.

Replace image names, ports, container names and endpoints with values from your own application. Run destructive commands only in a disposable environment.

---

## 1. Inspect the running release

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
docker inspect --format '{{.Name}} -> {{.Config.Image}}' $(docker ps -q)
```

PowerShell:

```powershell
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
docker ps --format "{{.Names}} -> {{.Image}}"
```

Confirm that the running container uses the immutable image tag expected by the release record. Avoid relying on a mutable `latest` tag for rollback.

---

## 2. Check the health and readiness endpoints

```bash
curl -fsS http://localhost:3000/health
curl -fsS http://localhost:3000/ready
```

PowerShell:

```powershell
Invoke-WebRequest http://localhost:3000/health
Invoke-WebRequest http://localhost:3000/ready
```

A liveness check answers whether the process is running. A readiness check should also verify that the service is ready to receive traffic, including required dependencies where appropriate.

---

## 3. Inspect the feature flag value

```bash
docker exec mern-api printenv NEW_CHECKOUT_ENABLED
docker compose exec api printenv NEW_CHECKOUT_ENABLED
```

PowerShell:

```powershell
docker exec mern-api printenv NEW_CHECKOUT_ENABLED
docker compose exec api printenv NEW_CHECKOUT_ENABLED
```

If the flag is supplied through a runtime configuration service, query that service using its authenticated administrative interface rather than changing a container manually.

---

## 4. Start the API with the feature disabled

```bash
docker run -d \
  --name mern-api-v2 \
  -p 3002:3000 \
  -e NEW_CHECKOUT_ENABLED=false \
  registry.example.com/mern-api:2.4.0
```

PowerShell:

```powershell
docker run -d --name mern-api-v2 -p 3002:3000 -e NEW_CHECKOUT_ENABLED=false registry.example.com/mern-api:2.4.0
```

The safe default should preserve the existing checkout path while the new artifact is validated.

---

## 5. Verify the disabled behavior

```bash
curl -fsS http://localhost:3002/health
curl -i -X POST http://localhost:3002/api/checkout \
  -H 'Content-Type: application/json' \
  -d '{"cartId":"demo-cart","paymentMethod":"test"}'
```

Confirm from the response and application logs that the expected old path was used. Do not use real payment data in a practice environment.

---

## 6. Inspect logs for flag and checkout decisions

```bash
docker logs --tail 200 mern-api-v2
docker logs -f mern-api-v2
```

Search for useful structured fields such as release version, flag key, flag value, request ID and outcome. Avoid logging payment credentials or sensitive customer data.

---

## 7. Run a safe smoke-test loop

```bash
for endpoint in /health /ready /api/products; do
  curl -fsS "http://localhost:3002${endpoint}" || exit 1
done
```

PowerShell:

```powershell
$endpoints = @('/health', '/ready', '/api/products')
foreach ($endpoint in $endpoints) {
  Invoke-WebRequest "http://localhost:3002$endpoint" | Out-Null
  if ($LASTEXITCODE -ne 0) { exit 1 }
}
```

Smoke tests are a release gate, not a complete business validation suite. Add a safe checkout test with a test payment provider when the environment supports it.

---

## 8. Enable the flag in a disposable Compose environment

Example `compose.override.yml` setting:

```yaml
services:
  api:
    environment:
      NEW_CHECKOUT_ENABLED: "true"
```

Apply it locally:

```bash
docker compose up -d api
docker compose exec api printenv NEW_CHECKOUT_ENABLED
```

For production, prefer an authorized flag-management operation that records who changed the flag, when it changed and why.

---

## 9. Inspect container resources and restart behavior

```bash
docker stats --no-stream
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.RunningFor}}"
```

Resource health does not prove feature correctness, but CPU, memory, restarts and saturation can explain a release failure.

---

## 10. Compare version-specific error rates

If your service emits structured JSON logs, filter by release version and status code with your log platform. A local example is:

```bash
docker logs mern-api-v2 2>&1 | grep -E 'release_version|status_code|checkout'
```

PowerShell:

```powershell
docker logs mern-api-v2 2>&1 | Select-String 'release_version|status_code|checkout'
```

For a production decision, use a metrics query with a time window, baseline comparison and minimum request count instead of counting a few terminal lines.

---

## 11. Simulate an application rollback

Stop the candidate only after traffic has been removed or the container is isolated:

```bash
docker stop mern-api-v2
docker rm mern-api-v2
docker run -d --name mern-api-v1 -p 3002:3000 registry.example.com/mern-api:2.3.0
curl -fsS http://localhost:3002/health
```

The production equivalent is normally a deployment-controller or load-balancer operation that restores the previous image. Keep the old image available before starting the release.

---

## 12. Simulate a feature rollback without rebuilding

The conceptual operation is:

```text
newCheckout = true
        |
        v
newCheckout = false
```

With a configuration service, use its authenticated CLI or API. With a local Compose environment, change the environment setting and recreate only the disposable service:

```bash
docker compose run --rm -e NEW_CHECKOUT_ENABLED=false api npm run config-check
```

A true runtime flag system should apply the change without rebuilding the application image. Validate the effective value and test the old behavior after the change.

---

## 13. Validate Compose configuration

```bash
docker compose config
docker compose ps
docker compose logs --tail 100 api mongodb
```

Check that flag values, image tags, service names, networks and database settings are what the release expects.

---

## 14. Check MongoDB compatibility before rollback

Use a safe read-only query in a non-production or authorized environment:

```bash
docker compose exec mongodb mongosh --eval 'db.users.findOne({}, {name:1, display_name:1})'
```

Before restoring an older application image, confirm that fields and indexes expected by that image still exist. Do not drop fields as part of an emergency rollback.

---

## 15. Example release gate

```bash
set -e

curl -fsS http://localhost:3002/health
curl -fsS http://localhost:3002/ready
curl -fsS http://localhost:3002/api/products

echo "Release smoke tests passed"
```

A real gate should also check release-specific metrics after an observation window. Health checks and smoke tests are necessary but not sufficient for business success.

---

## 16. Useful emergency checklist

```text
1. Identify the release version and affected feature.
2. Stop increasing exposure.
3. Check user impact and business metrics.
4. Disable the feature flag if the failure is isolated.
5. Roll back the application if the broader release is unsafe.
6. Preserve logs, metrics and deployment metadata.
7. Confirm recovery with smoke tests and business checks.
8. Investigate the root cause before redeploying.
```
