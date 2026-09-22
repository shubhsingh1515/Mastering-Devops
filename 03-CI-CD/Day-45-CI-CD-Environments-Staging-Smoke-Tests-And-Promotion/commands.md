# Day 45 - Commands: Staging, Smoke Tests, and Promotion

This file includes practical commands for validating a staging deployment and checking whether a release is safe to promote.

---

## 1. Check the Current Git State

```bash
git status
git branch --show-current
git log -1 --oneline
```

PowerShell:

```powershell
git status
git branch --show-current
git log -1 --oneline
```

These commands confirm which commit is being released and whether the working tree is clean.

---

## 2. Inspect the Image Tag You Will Promote

```bash
docker images
```

PowerShell:

```powershell
docker images
```

Or inspect a specific image:

```bash
docker image inspect registry.example.com/mern-api:<git-sha>
```

Example:

```bash
docker image inspect registry.example.com/mern-api:8f31c2a
```

This confirms the exact artifact that passed earlier CI and is now being staged.

---

## 3. Pull the Image for Staging Validation

```bash
docker pull registry.example.com/mern-api:<git-sha>
```

PowerShell:

```powershell
docker pull registry.example.com/mern-api:<git-sha>
```

This ensures the environment is running the exact same artifact that was published by CI.

---

## 4. Validate the Health Endpoint

```bash
curl -i https://staging.example.com/api/health
```

PowerShell:

```powershell
Invoke-WebRequest -Uri https://staging.example.com/api/health -Method Get
```

Expected behavior:

```text
HTTP/1.1 200 OK
```

Example response:

```json
{
  "status": "ok"
}
```

If the response is a 502 or timeout, the deployment should not be promoted.

---

## 5. Smoke Test a Real API Route

```bash
curl -i https://staging.example.com/api/products
```

PowerShell:

```powershell
Invoke-WebRequest -Uri https://staging.example.com/api/products -Method Get
```

For a production-like check, verify:

- status code is expected
- body is valid JSON
- the response is not empty or an error page

---

## 6. Check Container Status in Docker Compose

```bash
docker compose ps
docker compose logs --tail 100 api
```

PowerShell:

```powershell
docker compose ps
docker compose logs --tail 100 api
```

This is one of the first troubleshooting steps when healthy-looking infrastructure still fails smoke tests.

---

## 7. Inspect Environment Variables for the Service

```bash
docker compose config
docker exec -it <api-container-name> env
```

PowerShell:

```powershell
docker compose config
docker exec -it <api-container-name> env
```

Check for:

```text
NODE_ENV
MONGO_URI
PORT
```

This confirms whether the staging environment is using the correct values.

---

## 8. Troubleshoot MongoDB Connection Problems

If the API fails with a database error, inspect the connection string:

```bash
echo $MONGO_URI
```

Or inside the container:

```bash
docker exec -it <api-container-name> printenv MONGO_URI
```

PowerShell:

```powershell
docker exec -it <api-container-name> printenv MONGO_URI
```

Common wrong value:

```text
mongodb://localhost:27017/mern
```

Correct value in Docker network:

```text
mongodb://mongodb:27017/mern
```

This is a classic staging configuration bug.

---

## 9. Test a Full HTTP Flow Locally

```bash
curl -i http://localhost:3000/health
curl -i http://localhost:3000/api/products
```

PowerShell:

```powershell
Invoke-WebRequest http://localhost:3000/health
Invoke-WebRequest http://localhost:3000/api/products
```

This helps validate the same behavior before moving to a staging environment.

---

## 10. Check Reverse Proxy / Nginx Health

```bash
docker compose logs --tail 100 nginx
```

PowerShell:

```powershell
docker compose logs --tail 100 nginx
```

Look for messages such as:

```text
upstream connection refused
502 Bad Gateway
connection reset by peer
```

These are common signs that the API service is not ready or not reachable from the proxy.

---

## 11. Verify Service-to-Service Communication

```bash
docker network ls
docker network inspect <project-network>
```

PowerShell:

```powershell
docker network ls
docker network inspect <project-network>
```

This confirms containers can resolve each other using service names like:

```text
mongodb
api
nginx
```

rather than local loopback addresses.

---

## 12. Simulate a Failed Deployment

To learn the failure path, temporarily set staging configuration to a wrong API value:

```text
MONGO_URI=mongodb://localhost:27017/mern
```

Redeploy the service, then test:

```bash
curl -i https://staging.example.com/api/health
```

The expected result is a failed health check or 502.

Then fix the config and redeploy:

```text
MONGO_URI=mongodb://mongodb:27017/mern
```

This is a powerful practical drill.

---

## 13. Check for Promotion Readiness

Before promoting, a team often checks:

```bash
curl -f https://staging.example.com/api/health
curl -f https://staging.example.com/api/products
```

If one fails, the pipeline should stop.

Example:

```bash
if curl -fsS https://staging.example.com/api/health > /dev/null; then echo "Healthy"; else echo "Do not promote"; fi
```

This is a basic shell gate used in many pipelines.

---

## 14. Compare Staging and Production Variables

```bash
echo $NODE_ENV
echo $MONGO_URI
echo $PORT
```

PowerShell:

```powershell
$env:NODE_ENV
$env:MONGO_URI
$env:PORT
```

Review whether each variable is environment-specific and whether the secret values are stored appropriately.

---

## 15. Rollback to a Previously Validated Artifact

If production fails after promotion, redeploy the last known-good image:

```bash
docker pull registry.example.com/mern-api:<last-known-good-tag>
docker run -d --name mern-api-prod -p 3000:3000 registry.example.com/mern-api:<last-known-good-tag>
```

In a real deployment system, this often means rolling back to the previous release tag or version reference.

This works because the image was already validated and stored in the registry.

---

## 16. Typical Staging Workflow Checklist

Use this sequence:

```bash
docker pull registry.example.com/mern-api:<git-sha>

docker compose up -d

curl -i https://staging.example.com/api/health
curl -i https://staging.example.com/api/products

# if checks pass
# promote image to production
```

This is the practical version of the promotion gate.

---

## 17. Interview-Style Summary

A strong statement you can say out loud:

> Staging is where I validate the exact built artifact in an environment that behaves like production. I run minimal smoke tests to confirm health and connectivity before approving promotion. If those checks fail, I stop the release, investigate the cause, and do not send the artifact to production.

---

## 18. Suggested Practice Flow

Use this as a lab exercise:

```bash
# 1. build and push the image
# 2. deploy to staging
# 3. curl the health endpoint
# 4. check a real API route
# 5. confirm config values
# 6. if healthy, promote
# 7. if broken, investigate and stop
```

This is the same pattern used in real DevOps pipelines.
