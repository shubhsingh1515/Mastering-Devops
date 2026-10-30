# Day 66 Quiz — Kubernetes Networking

Answer the questions before checking the key.

## Questions

1. What is the primary purpose of a `ClusterIP` Service?
   - A. Publicly expose MongoDB
   - B. Provide an internal virtual endpoint for selected pods
   - C. Build a container image
   - D. Generate certificates

2. What can happen when a Service selector matches no pods?
   - A. Kubernetes creates matching pods
   - B. The Service has no backend endpoints
   - C. Cluster DNS is disabled
   - D. The Service becomes a Gateway

3. What is required for an Ingress resource to process traffic?
   - A. A compatible Ingress controller or implementation
   - B. A NodePort on every pod
   - C. A public database
   - D. An HPA

4. Which API provides a more expressive traffic-routing model?
   - A. Gateway API
   - B. ConfigMap API
   - C. Job API
   - D. Secret API

5. What is a common fully qualified name for `mongodb` in `production`?
   - A. `mongodb.production.svc.cluster.local`
   - B. `production.mongodb.public`
   - C. `mongodb.localhost`
   - D. `production.cluster.mongodb`

6. DNS resolves but TCP connection fails. What should you inspect?
   - A. Git branch name
   - B. Endpoints, ports, policies, and destination health
   - C. React CSS only
   - D. Image signature only

7. What does NetworkPolicy primarily control?
   - A. CPU allocation
   - B. Permitted network traffic to and from selected pods
   - C. Deployment history
   - D. Database schema

8. Why should MongoDB usually remain private?
   - A. It should accept only intended connections
   - B. Services cannot expose databases
   - C. It must share a node with React
   - D. It cannot use authentication

9. Ready API endpoints exist, but external requests fail. What should you
   inspect?
   - A. DNS, load balancer, Ingress or Gateway, TLS, and routing
   - B. MongoDB indexes only
   - C. CPU limits only
   - D. Commit messages

10. What is the best troubleshooting approach?
    - A. Restart every pod
    - B. Trace the request path and collect evidence at each layer
    - C. Disable all NetworkPolicies
    - D. Expose MongoDB publicly

## Answer key

| Question | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|
| Answer | B | B | A | A | A | B | B | A | A | B |

## Score guide

- **9–10:** Excellent networking fundamentals.
- **7–8:** Good; review the missed concepts.
- **5–6:** Revisit Services, DNS, and routing.
- **Below 5:** Repeat the lesson before advancing.
