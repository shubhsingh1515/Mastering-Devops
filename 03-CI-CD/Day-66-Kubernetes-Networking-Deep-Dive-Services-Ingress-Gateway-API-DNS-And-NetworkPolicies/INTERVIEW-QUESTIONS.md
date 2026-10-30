# Day 66 — Kubernetes Networking Interview Questions

## 1. What is the difference between a Service and an Ingress?

A Service provides a stable virtual endpoint for a set of pods. Ingress
defines HTTP or HTTPS routing rules that send external requests to Services.
Ingress requires a compatible controller or implementation; creating the
resource alone does not process traffic.

## 2. Why can a Service exist without working endpoints?

The selector may match no pod labels, selected pods may be unready, or the
workload may not exist in the expected namespace. Inspect the selector, pod
labels, readiness state, and EndpointSlices.

## 3. Compare ClusterIP and LoadBalancer.

`ClusterIP` provides an internal virtual endpoint and is appropriate for
internal APIs and databases. `LoadBalancer` requests an external load-balancing
integration when the platform supports it. A LoadBalancer does not replace
application routing or endpoint health checks.

## 4. How does Kubernetes DNS identify a Service?

Within the same namespace, a client can usually use the short name, such as
`mongodb`. A namespace-qualified name is `mongodb.production`, and a common
fully qualified name is
`mongodb.production.svc.cluster.local`. The cluster DNS suffix may differ.

## 5. What is Gateway API?

Gateway API is a more expressive Kubernetes traffic-routing model. Resources
such as `GatewayClass`, `Gateway`, and `HTTPRoute` can separate platform-owned
infrastructure from application-owned routing rules. A compatible
implementation is still required.

## 6. What does NetworkPolicy control?

NetworkPolicy controls permitted network traffic to and from selected pods,
subject to enforcement by the cluster networking implementation. It does not
grant Kubernetes API permissions; that is the responsibility of RBAC.

## 7. How would you troubleshoot an external HTTP 503?

Trace the request in order: public DNS, load balancer, Ingress or Gateway,
backend Service, EndpointSlices, pod readiness, application logs, NetworkPolicy
rules, and downstream dependencies. Verify the first failing layer and then
confirm recovery with a real API or checkout request.

## 8. Why might an application work inside the cluster but fail externally?

Internal Service discovery may work while public DNS, load-balancer health
probes, TLS, firewall rules, host matching, path matching, or Ingress/Gateway
configuration is wrong. Internal success does not validate the external path.

## 9. Why is `Running` not enough to prove traffic readiness?

`Running` describes the container lifecycle state. A failing readiness probe
normally excludes the pod from ready Service endpoints, so the application can
be running but receive no normal Service traffic.

## 10. How should MongoDB be protected?

Keep it private, expose only the required internal Service, restrict ingress to
the API pods with NetworkPolicy, use authentication and TLS as appropriate, and
monitor connection health. Publicly exposing MongoDB is not a valid fix for
service discovery or routing problems.

## Final challenge

Explain how you would investigate a frontend that loads while checkout returns
503. A strong answer identifies the first failing layer, cites concrete
evidence from Services and EndpointSlices, distinguishes DNS from TCP and
application connectivity, and verifies recovery without disabling security.
