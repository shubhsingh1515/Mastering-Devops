# Day 54 Commands: Load Balancing and High Availability

These are generic Docker, HTTP and shell examples. Adapt names, URLs and orchestration commands to the project. Do not expose secrets in terminal output.

## 1. Inspect Replica State

```bash
docker compose ps
docker compose ps --format json
docker compose events --since=15m
docker stats --no-stream
```

Inspect an individual instance:

```bash
docker inspect <container> --format '{{.Name}} status={{.State.Status}} restarts={{.RestartCount}} exit={{.State.ExitCode}}'
docker inspect <container> --format '{{.Config.Image}}'
```

## 2. Probe Liveness and Readiness

```bash
curl --include --silent https://api.example.com/health
curl --include --silent https://api.example.com/ready
```

A healthy process can still be unavailable for traffic:

```text
/health -> 200
/ready  -> 503
```

That means the process is alive but a critical dependency or initialization step prevents safe traffic.

## 3. Test Every Backend Directly

Use internal addresses or an approved diagnostic route to compare replicas:

```bash
curl --include --silent http://api-1:3000/health
curl --include --silent http://api-2:3000/health
curl --include --silent http://api-3:3000/health
curl --include --silent http://api-1:3000/ready
curl --include --silent http://api-2:3000/ready
curl --include --silent http://api-3:3000/ready
```

If the public load balancer returns intermittent failures, compare direct backend results and include the response headers or instance identity when available.

## 4. Inspect Load-Balancer Configuration

```bash
docker compose config
docker compose logs --timestamps --since=15m load-balancer
docker compose exec load-balancer cat /etc/nginx/nginx.conf
```

Look for:

```text
health-check endpoint
check interval and timeout
healthy/unhealthy thresholds
backend addresses and ports
weighted or sticky routing
connection draining behavior
TLS and forwarded headers
```

## 5. Inspect Network and DNS Paths

```bash
docker compose exec api getent hosts mongodb
docker compose exec api getent hosts api-2
docker compose exec api sh -c 'nc -zv mongodb 27017'
docker network inspect <project>_default
```

Inside a container, `localhost` refers to that container. It does not refer to MongoDB or another API instance.

## 6. Check MongoDB Pressure

Use the approved database dashboard or a restricted diagnostic account. Generic signals include:

```text
active connections
connection pool utilization
query latency
slow queries
CPU and memory
replication lag
disk and I/O
```

Never paste credentials or full connection strings into incident channels.

## 7. Inspect Instance Identity in Logs

```bash
docker compose logs --no-color --timestamps api | jq 'select(.status >= 500) | {instance, version, requestId, status}'
docker compose logs --no-color api | jq 'select(.event == "readiness_failed") | {instance, version, dependencies}'
docker compose logs --no-color api | jq 'select(.signal == "SIGTERM") | {instance, version, signal}'
```

The goal is to answer whether errors cluster around one instance or version.

## 8. Scale an API Safely

Use the project-approved deployment mechanism. A Compose example is:

```bash
docker compose up -d --no-deps --scale api=4 api
docker compose ps
```

After scaling, verify:

```bash
curl --fail-with-body --silent https://api.example.com/health
curl --fail-with-body --silent https://api.example.com/ready
```

Scaling is not complete until healthy capacity, latency, error rate and database connection usage are acceptable.

## 9. Observe a Rolling Replacement

```bash
docker compose ps
docker compose logs --timestamps --follow api
```

Expected sequence:

```text
instance starts
  -> liveness passes
  -> initialization completes
  -> readiness passes
  -> load balancer enables traffic
```

## 10. Test Graceful Shutdown

```bash
docker stop --time=15 <container>
docker compose logs --since=2m api
```

Confirm that the instance leaves traffic, active requests drain, dependency connections close and the process exits before the deadline.

## 11. Controlled Failure Drill

Only run this in a safe environment:

```bash
docker compose stop api-2
curl --include --silent https://api.example.com/ready
curl --include --silent https://api.example.com/representative-route
```

Expected result: traffic continues through healthy replicas and the load balancer does not route new requests to the stopped instance.

## 12. High-Availability Recovery Checklist

```text
[ ] All expected replicas are running
[ ] /health passes for healthy replicas
[ ] /ready passes continuously for traffic-eligible replicas
[ ] Unhealthy replicas receive no new traffic
[ ] Errors are not concentrated on one instance or version
[ ] MongoDB connections and latency are within limits
[ ] Graceful shutdown and draining completed
[ ] Representative user workflow succeeds
[ ] Business metrics are normal
[ ] Recovery evidence and timestamps are recorded
```
