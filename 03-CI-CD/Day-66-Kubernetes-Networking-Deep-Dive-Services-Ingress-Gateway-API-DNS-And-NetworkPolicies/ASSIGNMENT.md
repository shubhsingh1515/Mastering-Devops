# Day 66 Assignment — Trace a MERN Request

## Objective

Trace a request from `shop.example.com` to the React frontend, through the
Express API, and finally to MongoDB. The goal is to identify the first failing
networking layer instead of restarting workloads or weakening security.

## Scenario

The frontend loads successfully, but checkout requests return HTTP 503.
The architecture is:

```text
Public DNS
  -> Cloud Load Balancer
  -> Ingress or Gateway
  -> mern-api Service
  -> Express API pods
  -> mongodb Service
  -> MongoDB pods
```

## Tasks

### 1. Establish impact

Record the affected URL, HTTP method, response code, timestamp, affected
users, and whether static frontend requests still succeed.

### 2. Verify the internal Services

Inspect `mern-api` and `mongodb`. Record their ports, target ports, selectors,
namespace, and type. Explain whether each Service should be public or
internal.

### 3. Verify endpoints

Inspect EndpointSlices for both Services. Compare the selected pod labels with
the Service selectors and confirm that the selected pods are Ready.

### 4. Trace external routing

Inspect the Ingress or Gateway and document the host rule, `/api` path rule,
backend Service, backend port, TLS state, and controller or Gateway status.

### 5. Test DNS and TCP

From an approved diagnostic pod, resolve:

```text
mern-api.production.svc.cluster.local
mongodb.production.svc.cluster.local
```

Then test the required TCP ports. Explain why successful DNS resolution does
not prove that the API can authenticate to MongoDB or execute a query.

### 6. Review NetworkPolicies

Document which identities may communicate with the API and MongoDB. Confirm
that DNS, health checks, monitoring, and required application traffic remain
allowed.

### 7. Write an incident note

Include:

1. The failing request path.
2. The first component where the request failed.
3. Evidence that identified the failure.
4. The smallest corrective action.
5. Verification checks and user-facing recovery evidence.

## Expected outcome

Produce a short, evidence-based diagnosis. Do not report only that pods are
`Running`; demonstrate that DNS, routing, ready endpoints, ports, policies, and
the checkout workflow all work together.

## Acceptance criteria

- The API and database Services have correct selectors and ports.
- EndpointSlices contain the intended ready backends.
- External host and path routing reaches the API.
- API pods can resolve and connect to MongoDB.
- Unauthorized traffic remains denied.
- Recovery is verified with an application-level checkout request.
