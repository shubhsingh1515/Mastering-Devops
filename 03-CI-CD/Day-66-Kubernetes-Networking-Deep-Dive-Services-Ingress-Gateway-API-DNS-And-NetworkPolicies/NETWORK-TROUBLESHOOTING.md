# Day 66 — Network Troubleshooting Runbook

## Symptom: frontend works, API returns 503

Start by capturing the exact hostname, path, response code, timestamp, and
whether the failure affects every user. Then trace the request from the outside
in:

```text
Public DNS
  -> Load Balancer
  -> Ingress or Gateway
  -> API Service
  -> Ready API endpoint
  -> API pod
  -> MongoDB Service
  -> Ready MongoDB endpoint
  -> MongoDB pod
```

## Layer 1: DNS

Check that the public hostname resolves to the intended load balancer. For
internal failures, check that the Service exists in the expected namespace and
that the client uses the correct Service name. A DNS failure is different from
a refused TCP connection.

## Layer 2: Load balancer and routing

Check load-balancer health probes, listener ports, TLS certificates, firewall
rules, Ingress host and path matches, Gateway listeners, and controller logs.
An Ingress object with no healthy implementation cannot route traffic.

## Layer 3: Service and EndpointSlices

Compare:

1. Service namespace and name.
2. Service selector.
3. Pod labels.
4. Service `port`.
5. Pod `targetPort`.
6. EndpointSlice addresses and readiness.

If EndpointSlices are empty, fix the selector, labels, workload health, or
readiness configuration through the source-controlled deployment definition.

## Layer 4: Pod and application health

A pod can be `Running` but fail readiness probes. Review pod events, readiness
probe output, application logs, listen address, and container port. Confirm that
the process listens on the expected interface and port rather than only on
localhost.

## Layer 5: Internal database connectivity

From an approved diagnostic context:

1. Resolve the MongoDB Service name.
2. Test TCP port `27017`.
3. Confirm MongoDB endpoints are ready.
4. Check NetworkPolicies.
5. Verify authentication, TLS, and connection limits.
6. Confirm the application can execute a health query.

Do not treat a successful `nslookup` as proof that the database is usable.

## Layer 6: NetworkPolicy

List policies in the namespace and identify whether ingress or egress is
default-denied. Ensure that the API can reach MongoDB, workloads can reach
cluster DNS, and monitoring and health checks are not unintentionally blocked.
Policy enforcement depends on the installed CNI implementation.

## Recovery verification

After one targeted change:

- Confirm ready EndpointSlices.
- Confirm internal DNS and TCP connectivity.
- Confirm API health and application logs.
- Send a representative checkout request.
- Check HTTP error rate and latency.
- Verify that unauthorized traffic is still denied.

## Common incorrect fixes

- Restarting every pod before collecting logs and endpoint evidence.
- Exposing MongoDB publicly to bypass a NetworkPolicy.
- Assuming a `Running` pod is ready.
- Assuming an Ingress resource works without a controller.
- Disabling all policies instead of allowing the specific required path.
