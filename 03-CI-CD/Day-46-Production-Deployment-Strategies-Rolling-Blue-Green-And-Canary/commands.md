# Day 46 - Commands: Production Deployment Strategies

Use these commands to validate and practice rolling, health-check, and rollback behavior in a Dockerized MERN setup.

Replace ports, image names, and container names with values from your architecture.

---

## 1. Inspect running containers

```bash
docker ps
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
```

PowerShell:

```powershell
docker ps
```

This helps confirm how many API instances are active and what version they are running.

---

## 2. Check application health

```bash
curl -i http://localhost:3000/health
curl -i http://localhost:3001/health
curl -i http://localhost:3002/health
```

PowerShell:

```powershell
Invoke-WebRequest http://localhost:3000/health
Invoke-WebRequest http://localhost:3001/health
Invoke-WebRequest http://localhost:3002/health
```

A healthy service should return a success code such as 200 OK.

---

## 3. View logs during a rollout

```bash
docker logs --tail 100 api-1
docker logs --tail 100 api-2
docker logs --tail 100 api-3
```

PowerShell:

```powershell
docker logs --tail 100 api-1
docker logs --tail 100 api-2
docker logs --tail 100 api-3
```

This is useful when a new version starts but fails health checks or experiences runtime errors.

---

## 4. Run a simple rolling deployment simulation

Assume you have 3 API containers and you want to replace them one by one.

```bash
docker run -d --name api-1 -p 3001:3000 mern-api:v1
docker run -d --name api-2 -p 3002:3000 mern-api:v1
docker run -d --name api-3 -p 3003:3000 mern-api:v1
```

Then start one new container:

```bash
docker run -d --name api-1-v2 -p 3004:3000 mern-api:v2
curl -i http://localhost:3004/health
```

If healthy, redirect traffic to the new instance and remove the old version from the pool.

---

## 5. Stop and remove unhealthy instances

```bash
docker stop api-1-v2
docker rm api-1-v2
```

PowerShell:

```powershell
docker stop api-1-v2
docker rm api-1-v2
```

This is the operational action used when a release candidate fails readiness checks.

---

## 6. Inspect failure signals during rollout

```bash
docker stats
```

PowerShell:

```powershell
docker stats
```

You can track:

- CPU usage
- memory usage
- restart counts
- throughput changes

This helps identify whether the new version is unstable.

---

## 7. Check reverse-proxy upstream configuration

```bash
cat /etc/nginx/conf.d/default.conf
nginx -t
systemctl reload nginx
```

For Docker Compose-based setups:

```bash
docker compose logs nginx
```

If Nginx is routing traffic incorrectly, you may see failed upstream responses or 502 errors.

---

## 8. Confirm database compatibility

Check schema and application expectations:

```bash
mongo --host mongodb --eval "db.users.findOne()"
```

If using a local environment:

```bash
docker logs --tail 50 mongodb
```

This helps confirm whether the app version and database schema are compatible.

---

## 9. Run smoke tests after switching traffic

```bash
curl -i http://localhost:80/health
curl -i http://localhost:80/api/products
curl -i -X POST http://localhost:80/api/login -H "Content-Type: application/json" -d '{"email":"demo@example.com","password":"password"}'
```

Use only safe, non-destructive checks in the first smoke pass.

---

## 10. Simulate a rollback

If v2 fails, move traffic back to v1 and restart v1 instances:

```bash
docker stop api-1-v2
docker rm api-1-v2
docker run -d --name api-1 -p 3001:3000 mern-api:v1
curl -i http://localhost:3001/health
```

This demonstrates the principle:

```text
deploy v2 -> detect failure -> rollback to v1
```

---

## 11. Use Compose to inspect environment config

```bash
docker compose config
```

This helps verify:

- environment variables
- port mapping
- dependency naming
- network configuration

---

## 12. Useful production deployment checklist

```bash
docker ps
docker logs --tail 100 <container-name>
curl -i http://localhost:3000/health
docker inspect <container-name>
```

This is the minimum operational loop for a deployment investigation.

---

## 13. Example deployment policy

```bash
# Healthy deployment continues
curl -f http://localhost:3000/health || exit 1

# Smoke test gate
curl -f http://localhost:3000/api/products || exit 1
```

If the health endpoint or critical API route fails, the rollout should stop immediately.

---

## 14. Deployment decision example

```bash
if curl -fsS http://localhost:3000/health; then
  echo "Healthy; continue rollout"
else
  echo "Unhealthy; stop rollout and rollback"
  exit 1
fi
```

This is a simple automation decision point used in many deployment systems.

---

## 15. Key takeaway

Deployment scripts should automate more than image copy. They should also enforce:

- health check gate
- error threshold gate
- rollback trigger
- database compatibility validation

In other words, deployment automation should be safety-aware, not just speed-aware.
