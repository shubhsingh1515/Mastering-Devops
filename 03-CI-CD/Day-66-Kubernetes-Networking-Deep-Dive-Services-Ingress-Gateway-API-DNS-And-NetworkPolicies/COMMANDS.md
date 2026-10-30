# Day 66 — Kubernetes Networking Commands

Use these commands in a development or staging cluster first. Replace
placeholders with real values, use the correct namespace, and never print
Secret values.

## Inspect Services and selectors

```bash
kubectl get svc -n production
kubectl describe svc mern-api -n production
kubectl get svc mern-api -n production \
  -o jsonpath='{.spec.selector}'
kubectl get pods -n production --show-labels
```

Compare the Service selector with the labels on the intended pods. A Service
can exist successfully while selecting no pods.

## Inspect EndpointSlices

```bash
kubectl get endpointslices -n production \
  -l kubernetes.io/service-name=mern-api
kubectl describe endpointslice -n production \
  -l kubernetes.io/service-name=mern-api
kubectl get endpoints mern-api -n production
```

EndpointSlices show whether the Service has usable addresses and whether those
addresses are ready. Empty or unready endpoints explain many HTTP 503 errors.

## Test Kubernetes DNS

```bash
kubectl exec -it <api-pod> -n production -- \
  nslookup mongodb.production.svc.cluster.local
kubectl exec -it <api-pod> -n production -- \
  getent hosts mongodb.production.svc.cluster.local
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=100
```

Use the cluster DNS domain configured by your environment. Minimal images may
not include `nslookup` or `getent`; use an approved temporary diagnostic pod
instead of changing the production image only for troubleshooting.

## Test TCP connectivity

```bash
kubectl exec -it <api-pod> -n production -- \
  sh -c 'nc -vz mongodb 27017'
kubectl run network-debug --rm -it --restart=Never \
  --image=busybox:1.36 -n production -- \
  nslookup mongodb.production.svc.cluster.local
```

DNS success proves only name resolution. Continue checking endpoints, ports,
NetworkPolicies, database listeners, authentication, and TLS.

## Inspect Ingress and Gateway API

```bash
kubectl get ingress -n production
kubectl describe ingress mern-ingress -n production
kubectl get ingressclass
kubectl get gatewayclass
kubectl get gateway -n production
kubectl get httproute -n production
kubectl describe httproute shop-route -n production
```

Verify the hostname, path match, backend Service name, backend port, TLS
status, and the health of the controller or Gateway implementation.

## Inspect NetworkPolicies

```bash
kubectl get networkpolicy -n production
kubectl describe networkpolicy mongodb-ingress -n production
kubectl get pods -n production \
  -l app=mongodb --show-labels
kubectl get pods -n production \
  -l app=mern-api --show-labels
```

When a default-deny policy is present, confirm that application traffic,
cluster DNS, health checks, monitoring, and required external dependencies are
explicitly allowed.

## Verify the external request path

```bash
kubectl get svc -n ingress-nginx
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx \
  -l app.kubernetes.io/component=controller --tail=100
curl -I https://shop.example.com
curl -i https://shop.example.com/api/health
```

The final checks should include a representative business request, such as a
checkout flow, not only a successful TCP connection or a `Running` pod state.

## Safe operating rules

1. Capture the failing request, timestamp, hostname, path, and response code.
2. Trace from DNS to load balancer, Ingress or Gateway, Service, endpoints, and
   pod before changing configuration.
3. Use EndpointSlices and readiness state rather than assuming `Running` means
   traffic-ready.
4. Do not expose MongoDB publicly to solve an internal connectivity problem.
5. Apply one targeted change at a time and verify the user-facing result.
