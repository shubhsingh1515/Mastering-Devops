# Day 55 Commands: Scaling and Capacity Planning

These are generic examples. Adapt service names, URLs, tools and thresholds to the project. Run load tests only against an approved environment, and never print secrets.

## 1. Inspect Current API Capacity

```bash
docker compose ps
docker compose ps --format json
docker stats --no-stream
docker compose events --since=15m
```

Inspect restart counts and images:

```bash
docker inspect <container> --format '{{.Name}} image={{.Config.Image}} status={{.State.Status}} restarts={{.RestartCount}}'
```

## 2. Scale API Replicas

Use the approved deployment process. A Compose example is:

```bash
docker compose up -d --no-deps --scale api=4 api
docker compose ps
```

After scaling, verify readiness and observe the downstream database before adding more instances:

```bash
curl --fail-with-body --silent https://api.example.com/health
curl --fail-with-body --silent https://api.example.com/ready
```

## 3. Measure HTTP Latency and Errors

```bash
curl --include --silent --write-out '\nstatus=%{http_code} total=%{time_total}s\n' https://api.example.com/api/products
curl --fail-with-body --silent https://api.example.com/api/products > /dev/null
```

Use the project metrics system for p50, p95 and p99 latency. A single curl request is not a capacity test.

## 4. Inspect API and Database Logs

```bash
docker compose logs --timestamps --since=15m api
docker compose logs --timestamps --since=15m mongodb
docker compose logs --no-color api | jq 'select(.status >= 500) | {instance, version, requestId, status, durationMs}'
```

Look for instance, version, request ID, route, latency, timeout and dependency error fields.

## 5. Inspect Network and Database Reachability

```bash
docker compose exec api getent hosts mongodb
docker compose exec api sh -c 'nc -zv mongodb 27017'
docker network inspect <project>_default
```

Do not print a complete MongoDB URI or credentials. Prefer a safe diagnostic that reports connectivity and sanitized configuration metadata.

## 6. Check Cache Behavior

For Redis-like systems, use a restricted diagnostic account and avoid exposing values:

```bash
redis-cli -h <redis-host> INFO stats
redis-cli -h <redis-host> INFO memory
redis-cli -h <redis-host> DBSIZE
```

Useful signals include cache hits, misses, evictions, memory use, key expiration and latency. A high hit rate is not automatically correct if the cached data is stale or incorrectly authorized.

## 7. Inspect Queue and Worker Health

Use the queue platform dashboard or approved CLI to inspect:

```text
queue depth
oldest job age
processing rate
active jobs
failed jobs
retry count
dead-letter jobs
worker concurrency
```

A queue whose depth grows continuously is receiving work faster than workers can process it.

## 8. Observe External Dependency Behavior

```bash
curl --include --silent --max-time 3 https://dependency.example.com/health
```

Use bounded timeouts for diagnostics. Record status, latency and failure rate without exposing tokens or customer data. Validate provider quotas and rate limits in the provider dashboard.

## 9. Run an Approved Load Test

Use the project's chosen load-testing tool and environment. A conceptual progression is:

```text
500 requests/second
1,000 requests/second
1,500 requests/second
2,000 requests/second
```

Record:

```text
throughput
p50/p95/p99 latency
5xx and timeout rate
API CPU and memory
MongoDB CPU, connections and query latency
cache hit rate
queue depth and job age
external dependency latency
business success rate
```

Stop the test when error rate, latency or dependency pressure exceeds the approved safety limit.

## 10. Validate a Scaling Change

```bash
docker compose ps
docker stats --no-stream
curl --fail-with-body --silent https://api.example.com/health
curl --fail-with-body --silent https://api.example.com/ready
```

Compare before and after:

```text
API capacity
API latency
MongoDB connections
MongoDB query latency
cache hit rate
queue depth
5xx rate
business success
```

## 11. Capacity Review Checklist

```text
[ ] Current request and business workload is recorded
[ ] Expected peak and growth are documented
[ ] API CPU and memory baseline is known
[ ] MongoDB connections and query latency are known
[ ] Cache hit/miss and eviction behavior is known
[ ] Queue depth and worker throughput are known
[ ] External dependency limits are documented
[ ] Load test is approved and repeatable
[ ] Scaling triggers have owners and actions
[ ] Cost and rollback limits are defined
```
