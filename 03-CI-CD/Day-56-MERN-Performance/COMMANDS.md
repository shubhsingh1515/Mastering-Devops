# Day 56 Commands: MERN Performance

This reference collects the practical commands and code snippets used to inspect, optimize and validate MERN performance.

## MongoDB Index Commands

Connect to MongoDB with the Mongo Shell:

```bash
mongosh "mongodb://localhost:27017/mern_app"
```

Inspect existing indexes:

```javascript
db.products.getIndexes();
db.orders.getIndexes();
```

Create a single-field index for frequent user lookups:

```javascript
db.orders.createIndex({ userId: 1 });
```

Create a compound index for filtering by user and sorting by newest order:

```javascript
db.orders.createIndex({ userId: 1, createdAt: -1 });
```

Create an index for a category query sorted by newest product:

```javascript
db.products.createIndex({ category: 1, _id: -1 });
```

Inspect index usage statistics:

```javascript
db.products.aggregate([{ $indexStats: {} }]);
```

Drop an index only after validating its impact and rollback plan:

```javascript
db.orders.dropIndex("userId_1");
```

## MongoDB Query Analysis

Run a query with execution statistics:

```javascript
db.products
  .find({ category: "shoes" })
  .project({ name: 1, price: 1, thumbnail: 1 })
  .sort({ _id: -1 })
  .limit(20)
  .explain("executionStats");
```

Inspect an order query with a compound index:

```javascript
db.orders
  .find({ userId: "123" })
  .sort({ createdAt: -1 })
  .limit(20)
  .explain("executionStats");
```

Useful fields to inspect in the result:

```text
winningPlan
executionStats.executionTimeMillis
executionStats.totalKeysExamined
executionStats.totalDocsExamined
executionStats.nReturned
```

Compare query work with returned results:

```text
Work ratio = totalDocsExamined / nReturned
```

A high ratio is a signal to investigate filters, sort order, projections, pagination and indexes. It is not by itself proof that a new index is correct.

## Express Request Timing

Add a simple timing middleware during development or adapt it to the production metrics system:

```javascript
app.use((request, response, next) => {
  const startedAt = process.hrtime.bigint();

  response.on("finish", () => {
    const durationMs = Number(process.hrtime.bigint() - startedAt) / 1_000_000;

    console.log(JSON.stringify({
      route: request.route?.path ?? request.path,
      method: request.method,
      statusCode: response.statusCode,
      durationMs: Math.round(durationMs)
    }));
  });

  next();
});
```

## Projection and Pagination

Use a bounded page size and cap the client input:

```javascript
const page = Math.max(Number.parseInt(request.query.page, 10) || 1, 1);
const requestedLimit = Number.parseInt(request.query.limit, 10) || 20;
const limit = Math.min(Math.max(requestedLimit, 1), 100);
const skip = (page - 1) * limit;
```

Use projection and `lean()` for a read-only list response:

```javascript
const products = await Product.find({ category })
  .select("name price thumbnail")
  .sort({ _id: -1 })
  .skip(skip)
  .limit(limit)
  .lean();
```

Use cursor pagination for large, ordered datasets:

```javascript
const filter = after
  ? { _id: { $lt: new ObjectId(after) }, category }
  : { category };

const products = await Product.find(filter)
  .select("name price thumbnail")
  .sort({ _id: -1 })
  .limit(limit + 1)
  .lean();

const hasNextPage = products.length > limit;
const items = products.slice(0, limit);
const nextCursor = hasNextPage ? items.at(-1)._id : null;
```

## Cache Inspection and Cache-Aside

A generic Redis connection check:

```bash
redis-cli -u "$REDIS_URL" ping
```

Inspect a cache key without exposing sensitive values in shared terminals:

```bash
redis-cli -u "$REDIS_URL" EXISTS "products:all:1:20"
redis-cli -u "$REDIS_URL" TTL "products:all:1:20"
```

Do not print production secrets or private cached payloads into command history or shared logs.

Cache-aside example:

```javascript
async function getProducts({ category, page, limit }) {
  const cacheKey = `products:${category ?? "all"}:${page}:${limit}`;
  const cached = await redis.get(cacheKey);

  if (cached) {
    return JSON.parse(cached);
  }

  const products = await Product.find({ category })
    .select("name price thumbnail")
    .sort({ _id: -1 })
    .limit(limit)
    .lean();

  await redis.set(cacheKey, JSON.stringify(products), { EX: 60 });
  return products;
}
```

Invalidate a changed resource after a successful write:

```javascript
await redis.del(`products:${category}:1:20`);
```

The actual key set must match the application invalidation design. If a write can affect several pages, invalidate all affected keys or use a versioned namespace.

## HTTP Cache Headers

Inspect response headers with `curl`:

```bash
curl -I https://example.com/assets/main.8f31c2.js
curl -I https://api.example.com/api/products
```

Example headers for an immutable, content-hashed asset:

```http
Cache-Control: public, max-age=31536000, immutable
ETag: "8f31c2"
```

Example validation request:

```bash
curl -H 'If-None-Match: "8f31c2"' -i https://example.com/assets/main.8f31c2.js
```

## API Smoke and Timing Checks

Measure response headers and total request time:

```bash
curl -sS -o /dev/null -w 'status=%{http_code} total=%{time_total}s\n' https://api.example.com/api/products?limit=20
```

Repeat a request a small number of times for a basic comparison:

```bash
for i in 1 2 3 4 5; do curl -sS -o /dev/null -w '%{time_total}\n' https://api.example.com/api/products?limit=20; done
```

This is not a replacement for load testing, but it can quickly reveal an obvious regression or connectivity problem.

## Load Testing with k6

Install k6 according to the official documentation, then run a script:

```bash
k6 run performance/products.js
```

Example `performance/products.js`:

```javascript
import http from "k6/http";
import { check, sleep } from "k6";

export const options = {
  stages: [
    { duration: "30s", target: 10 },
    { duration: "60s", target: 10 },
    { duration: "30s", target: 0 }
  ],
  thresholds: {
    http_req_failed: ["rate<0.01"],
    http_req_duration: ["p(95)<500", "p(99)<1000"]
  }
};

export default function () {
  const response = http.get(`${__ENV.BASE_URL}/api/products?limit=20`);
  check(response, {
    "status is 200": (result) => result.status === 200,
    "response is bounded": (result) => result.body.length < 200000
  });
  sleep(1);
}
```

Run against a selected environment:

```bash
BASE_URL=https://staging.example.com k6 run performance/products.js
```

On PowerShell:

```powershell
$env:BASE_URL = "https://staging.example.com"; k6 run performance/products.js
```

Use the same traffic profile, dataset, cache state and duration for before-and-after comparisons.

## Docker and Application Inspection

Inspect running containers and resource usage:

```bash
docker compose ps
docker stats
```

Inspect recent API logs:

```bash
docker compose logs --tail=200 api
```

Follow API logs during a controlled test:

```bash
docker compose logs -f api
```

Check the health endpoint:

```bash
curl -fsS https://api.example.com/health
```

These commands help locate obvious runtime problems, but database metrics and distributed traces are still needed for complete diagnosis.

## Safe Command Checklist

Before running a production performance command:

```text
Confirm the target environment
Confirm the database and collection
Prefer read-only analysis first
Avoid printing secrets or private data
Capture the current index and query plan
Record the command and result in the change ticket
Have a rollback or recovery plan for writes and index changes
```
