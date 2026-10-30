# Day 66 — Kubernetes Networking Deep Dive: Services, Ingress, Gateway API, DNS & NetworkPolicies
**Date:** Friday, October 16, 2026
**Phase 4:** DevSecOps & Production Engineering
**Duration:** 25–30 minutes
**Level:** Intermediate → Advanced
**Focus:** Kubernetes networking, MERN production architecture, troubleshooting, and DevOps interview preparation.

## 🎯 Today's Mission
Yesterday, on **Day 65**, you learned how to troubleshoot `CrashLoopBackOff`, `OOMKilled`, `Pending` pods, `ImagePullBackOff`, DNS failures, and Services with missing endpoints.

Today, we'll go deeper into Kubernetes networking and understand how traffic travels through your production infrastructure.

Imagine your MERN application is deployed to Kubernetes:

- Users can open your React frontend, but API requests fail.
- The Express API cannot resolve the MongoDB Service.
- A Kubernetes Service exists but doesn't route traffic to any pods.
- Your application works inside the cluster but isn't accessible externally.
- A NetworkPolicy blocks legitimate traffic.
- Your cloud load balancer returns HTTP 502 or 503 errors.
You need to understand the entire request path to diagnose these failures.

**Today's central question:**

> How does traffic travel from an external user to a React frontend, an Express API, and MongoDB in Kubernetes, and how do you secure and troubleshoot every connection?

### Today's learning plan
TimeActivity0–4 minKubernetes networking architecture4–8 minServices and service discovery8–12 minDNS and internal connectivity12–17 minIngress and Gateway API17–21 minNetworkPolicies and traffic security21–24 minProduction troubleshooting and interview questions24–27 minReview questions and quiz27–30 minPractical assessment

---

# 1. 🧠 Kubernetes Networking Architecture
Consider a production e-commerce application.

```
                  INTERNET
                     |
                     v
              DNS: shop.example.com
                     |
                     v
             Cloud Load Balancer
                     |
                     v
             Ingress / Gateway
                     |
             +-------+-------+
             |               |
             v               v
       React Frontend     Express API
                               |
                       +-------+-------+
                       |               |
                       v               v
                    MongoDB          Redis
```

Each component serves a different purpose.

ComponentResponsibilityDNSResolves hostnames to the appropriate destinationLoad balancerDistributes incoming traffic across available targetsIngress controller or Gateway implementationRoutes external HTTP traffic according to configured rulesKubernetes ServiceProvides a stable virtual endpoint for a set of workloadsPod networkingEnables communication with individual podsNetworkPolicyRestricts permitted traffic to and from selected pods

The key is understanding that **a Kubernetes Service, an Ingress, a Gateway, and a cloud load balancer are not interchangeable components**.

They operate at different layers of the traffic path.

## Example: A customer checks out
A customer opens:

`https://shop.example.com`

The React application sends a request to:

`https://shop.example.com/api/orders`

A typical request path is:

```
Browser
   |
   v
Public DNS
   |
   v
Cloud Load Balancer
   |
   v
Ingress / Gateway
   |
   v
Express API Service
   |
   v
Ready Express API Pod
   |
   v
MongoDB Service
   |
   v
MongoDB Pod
```

The actual path depends on your architecture. For example, the frontend may be served through a CDN, and MongoDB may run outside Kubernetes.

For today's examples, we'll assume the frontend, API, and database are inside the cluster.

---

# 2. 🔌 Kubernetes Services: Stable Networking for Pods
Pods are replaceable.

A pod might be deleted during a deployment, recreated after a failure, or moved to another node.

Its IP address can change.

Suppose your API has three replicas:

```
Express API Service
       |
       +------> API Pod A: 10.1.2.10
       |
       +------> API Pod B: 10.1.3.15
       |
       +------> API Pod C: 10.1.4.20
```

If Pod A is replaced, the application should not need to discover its new IP manually.

A Kubernetes Service provides a stable virtual endpoint for accessing the selected pods.

## 2.1 ClusterIP Service
A `ClusterIP` Service is the default Kubernetes Service type. It provides an internal virtual IP for cluster networking.

Example:

```
apiVersion: v1
kind: Service
metadata:
  name: mern-api
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: mern-api
  ports:
    - name: http
      port: 80
      targetPort: 5000
```

Suppose the Express containers listen on port `5000`.

The Service exposes port `80` and forwards traffic to port `5000` on selected pods.

The selector:

```
selector:
  app: mern-api
```

must match the appropriate pod labels.

For example:

```
labels:
  app: mern-api
```

If the labels do not match, the Service may have no backend endpoints.

### Interview question
**Why use a Service instead of connecting directly to a pod IP?**

A strong answer:

"Pod IPs are not stable application endpoints because pods can be replaced or rescheduled. A Kubernetes Service provides a stable virtual endpoint and routes traffic to the selected backend pods."

## 2.2 Other Service types
Service typePurposeTypical use`ClusterIP`Internal virtual endpointExpress API or MongoDB`NodePort`Exposes a port on cluster nodesSpecific cluster networking or testing scenarios`LoadBalancer`Requests an external load-balancing integration where supportedExposing an application through a cloud load balancerHeadless ServiceUses `clusterIP: None` rather than a normal virtual IPStateful applications and direct endpoint discovery

For most production MERN architectures, internal APIs and databases commonly use `ClusterIP` Services.

A database usually should not be exposed publicly merely because the application needs to connect to it.

---

# 3. 🌐 Kubernetes DNS and Service Discovery
On Day 65, you learned how to investigate DNS failures.

Today, let's understand how Kubernetes service discovery works.

Suppose your MongoDB Service is called `mongodb` and exists in the `production` namespace.

An Express pod in the same namespace can usually connect using:

```
mongodb:27017
```

A pod in another namespace can use a namespace-qualified name:

```
mongodb.production:27017
```

The fully qualified service name commonly looks like:

```
mongodb.production.svc.cluster.local
```

The cluster DNS domain can be different from `cluster.local`, so verify your environment when troubleshooting.

## 3.1 Example MERN configuration
Your Express backend might use:

```
const mongoose = require("mongoose");

async function connectDatabase() {
  const uri = process.env.MONGODB_URI;

  if (!uri) {
    throw new Error("MONGODB_URI is not configured");
  }

  await mongoose.connect(uri);
}

module.exports = connectDatabase;
```

A securely provisioned configuration could contain a connection string resembling:

```
mongodb://mongodb:27017/shop
```

This connection string is illustrative. Production deployments should use authentication and appropriate transport security, and credentials should be delivered securely rather than committed to Git.

## 3.2 Troubleshooting DNS
Suppose the API reports:

```
getaddrinfo ENOTFOUND mongodb
```

Investigate in this order:

1. Does the `mongodb` Service exist?
2. Is it in the expected namespace?
3. Is the hostname correct?
4. Is cluster DNS functioning?
5. Does the pod have working DNS configuration?
6. Are NetworkPolicies blocking DNS traffic?
Useful commands:

```
kubectl get svc -n production
```

```
kubectl get pods -n kube-system
```

If diagnostic tools are available inside your pod:

```
kubectl exec -it <api-pod> -n production -- \
  nslookup mongodb.production.svc.cluster.local
```

The lookup command requires `nslookup` to be installed in the container.

For minimal production images, use an approved diagnostic pod if necessary.

### Critical distinction
DNS resolution is not the same as network connectivity.

If the hostname resolves successfully but the TCP connection fails, investigate:

- Service endpoints.
- Target ports.
- NetworkPolicies.
- Database listener configuration.
- Database availability.
- Authentication and TLS.
**Interview takeaway:** Diagnose name resolution, routing, and application connectivity as separate layers.

---

# 4. 🚪 Ingress: Routing External HTTP Traffic
Your application needs to be accessible to customers outside the Kubernetes cluster.

A common solution is Kubernetes Ingress.

Ingress defines HTTP and HTTPS routing rules, typically based on hostnames and URL paths.

However, **creating an Ingress resource alone does not make it work**. You also need an Ingress controller or another implementation that processes those rules.

## 4.1 MERN routing example
You want these routes:

```
https://shop.example.com/
        |
        v
React Frontend

https://shop.example.com/api/
        |
        v
Express API
```

An Ingress resource can define this routing.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mern-ingress
  namespace: production
spec:
  ingressClassName: nginx
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: mern-api
                port:
                  number: 80

          - path: /
            pathType: Prefix
            backend:
              service:
                name: mern-frontend
                port:
                  number: 80
```

This example assumes:

- An appropriate Ingress controller is installed.
- An IngressClass named `nginx` exists.
- The `mern-api` Service exposes port `80`.
- The `mern-frontend` Service exposes port `80`.
- Both Services are in the `production` namespace.
The controller's configuration determines additional behavior, including TLS termination and any path-rewriting rules.

For example, `/api/orders` may reach the Express backend with the same path unless the controller is explicitly configured to rewrite it.

### Important production consideration
Ingress implementations differ.

Do not assume that annotations or advanced routing features supported by one controller work identically in another.

Also, the Kubernetes Ingress API is stable but frozen; the Kubernetes project recommends the Gateway API for future development of more expressive traffic-routing capabilities.

---

# 5. 🚦 Gateway API: A More Expressive Routing Model
The Kubernetes Gateway API provides resources for defining and managing network traffic routing.

Important resources include:

ResourcePurposeGatewayClassIdentifies a class of Gateway implementationGatewayDefines a traffic-handling entry pointHTTPRouteDefines HTTP routing rulesReferenceGrantAllows certain cross-namespace references where supported and required

The Gateway API separates infrastructure configuration from application routing more explicitly.

For example:

```
Platform Team
     |
     v
Gateway
     |
     +----------------------+
                            |
Application Team             |
     |                      |
     v                      |
HTTPRoute ------------------+
     |
     +----> React Service
     |
     +----> Express API Service
```

A platform team might manage the Gateway and its infrastructure, while application teams manage permitted HTTPRoute resources.

This separation can be useful in larger organizations.

## Ingress versus Gateway API
FeatureIngressGateway APIBasic HTTP routingSupportedSupportedHost and path routingSupportedSupportedSeparation of infrastructure and application configurationMore limited in the core APIMore explicitAdvanced routing capabilitiesOften controller-specificExpressed through richer API resources and supported extensionsController or implementation requiredYesYes

Neither resource is a complete implementation by itself.

Before adopting Gateway API, confirm that your chosen implementation supports the resources and features you intend to use.

### Interview question
**What is the difference between Ingress and a Gateway API HTTPRoute?**

A strong answer:

"Ingress provides a relatively simple API for HTTP and HTTPS routing. Gateway API offers a more expressive model with resources such as GatewayClass, Gateway, and HTTPRoute, allowing clearer separation between infrastructure ownership and application routing. Both require a compatible implementation to process their configuration."

---

# 6. 🔐 NetworkPolicies: Secure the Traffic Path
On Day 63, you learned that NetworkPolicies restrict communication between workloads.

Today, let's apply that principle to the full MERN architecture.

Suppose you want:

```
External Users
      |
      v
Frontend
      |
      v
Express API
      |
      v
MongoDB
```

Your intended communication rules might be:

SourceDestinationRequired trafficIngress controllerFrontendHTTPIngress controllerExpress APIHTTPExpress APIMongoDBMongoDB protocol over TCPExpress APIRedisRedis protocol over TCP, if usedApplication podsCluster DNSDNS, as required

A NetworkPolicy can restrict traffic to selected pods.

## Example: Restrict MongoDB ingress
Assume the MongoDB pods have the label `app: mongodb`, while the Express API pods have `app: mern-api`.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mongodb-ingress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: mongodb
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: mern-api
      ports:
        - protocol: TCP
          port: 27017
```

This policy allows ingress to selected MongoDB pods from matching API pods in the same namespace on TCP port `27017`, subject to the cluster's NetworkPolicy enforcement.

It does not automatically restrict MongoDB's outbound traffic or define the complete network security posture of the application.

### Production warning
When introducing default-deny policies, account for every required communication path, including DNS, monitoring, health checks, and external dependencies.

NetworkPolicy enforcement depends on the cluster networking implementation.

**Interview takeaway:** A working Service and correct DNS entry do not guarantee that network traffic is permitted.

---

# 7. 🚨 Production Troubleshooting: The API Returns HTTP 503
Imagine your production application has these symptoms:

```
React Frontend: Accessible
Express API:    HTTP 503
MongoDB:        Running
```

The database is running, but users cannot complete checkout.

Use this sequence.

## Step 1: Check the API Service

```
kubectl get svc mern-api -n production
```

Verify that the Service exists and exposes the expected port.

## Step 2: Check EndpointSlices

```
kubectl get endpointslices -n production \
  -l kubernetes.io/service-name=mern-api
```

If no ready backend endpoints are available, inspect:

- Service selectors.
- Pod labels.
- Pod readiness.
- Deployment health.
- Readiness probe failures.

## Step 3: Check the API pods

```
kubectl get pods -n production
```

If the pods are running but not ready, inspect their descriptions and logs.

```
kubectl describe pod <api-pod> -n production
```

## Step 4: Check the routing layer
If the Service has ready endpoints, inspect the Ingress or Gateway configuration and implementation logs.

Check:

- Hostname rules.
- URL path rules.
- Backend Service names.
- Backend Service ports.
- TLS configuration.
- Controller or Gateway health.

## Step 5: Check network restrictions
Confirm that the required traffic is allowed by applicable NetworkPolicies.

Also verify that the Express API can reach MongoDB.

## Step 6: Verify recovery
Check:

- Successful API requests.
- Checkout completion.
- HTTP error rates.
- API latency.
- Ready backend endpoints.
A pod being `Running` does not prove that the application is ready to receive traffic.

Likewise, a database being `Running` does not prove that the Express API can reach it.

---

# 8. 🎤 Interview Preparation: Five Important Questions

### Question 1: What is the difference between ClusterIP and LoadBalancer?
**Strong answer:**

"ClusterIP provides an internal virtual endpoint for accessing selected pods within the cluster's networking environment. LoadBalancer requests an external load-balancing integration when supported by the platform. For a typical MERN application, the Express API and MongoDB often use ClusterIP Services, while external access is provided through a load balancer and an HTTP routing layer."

### Question 2: Why can a Service exist without working endpoints?
**Strong answer:**

"A Service can exist even if its selector matches no pods or no selected pods are ready. I'd inspect the Service selector, pod labels, readiness state, and EndpointSlices before investigating higher-level routing."

### Question 3: What is the difference between Ingress and a Service?
**Strong answer:**

"A Service provides a stable endpoint for accessing a set of pods. Ingress defines HTTP and HTTPS routing rules that are processed by an Ingress controller. An external request may travel through a load balancer and Ingress before reaching the backend Service."

### Question 4: Why might an application work inside Kubernetes but fail externally?
**Strong answer:**

"Internal connectivity may be working while external DNS, the cloud load balancer, Ingress or Gateway configuration, TLS, firewall rules, or external routing is misconfigured. I'd trace the request path from the external entry point to the backend Service."

### Question 5: How do you secure Kubernetes network communication?
**Strong answer:**

"I'd expose only the necessary external entry points, use NetworkPolicies to restrict pod communication, keep databases private, configure appropriate TLS, and verify that Services route only to intended workloads. I'd also monitor traffic and test the policies to ensure that required application paths continue working."

---

# 9. 🔁 Review Questions From Earlier Lessons
Answer these four questions before checking the solutions.

**Q1 — Day 63: Kubernetes Security**

What is the difference between RBAC and NetworkPolicy?

**Q2 — Day 64: Health Probes**

Why can a pod be `Running` but not receive normal Service traffic?

**Q3 — Day 65: Troubleshooting**

What should you investigate if a Service has no ready endpoints?

**Q4 — Day 62: Supply-Chain Security**

Why doesn't a valid container image signature guarantee that the application is secure?

<details>
<summary><strong>Reveal the answers</strong></summary>**1.** RBAC controls which identities can perform actions on Kubernetes resources. NetworkPolicy governs permitted network traffic to and from selected pods.

**2.** The container may be running while its readiness probe fails. Kubernetes normally excludes an unready pod from the ready endpoints used for Service traffic.

**3.** Check the Service selector, pod labels, readiness state, and EndpointSlices. Also verify that the intended backend pods exist and are functioning.

**4.** A signature provides evidence about artifact integrity and signing identity. It does not prove the application has no vulnerabilities or that its runtime configuration is secure.

</details>
---

# 10. 📝 Today's Quiz
**Time limit: 3 minutes.**

### Q1. What is the primary purpose of a Kubernetes ClusterIP Service?
A. Expose a database publicly
B. Provide an internal virtual endpoint for selected pods
C. Build container images
D. Generate TLS certificates automatically

### Q2. What happens if a Service selector matches no pods?
A. Kubernetes automatically creates matching pods
B. The Service may have no backend endpoints
C. DNS is automatically disabled cluster-wide
D. The Service becomes a Gateway

### Q3. What is normally required for an Ingress resource to process traffic?
A. A compatible Ingress controller or implementation
B. A NodePort on every pod
C. A public MongoDB endpoint
D. An HPA

### Q4. Which Kubernetes API is designed to provide a more expressive traffic-routing model than the core Ingress API?
A. Gateway API
B. ConfigMap API
C. Job API
D. Secret API

### Q5. Which hostname commonly identifies a Service called `mongodb` in the `production` namespace?
A. `mongodb.production.svc.cluster.local`
B. `production.mongodb.public`
C. `mongodb.localhost`
D. `production.cluster.mongodb`

### Q6. A hostname resolves, but the TCP connection fails. What should you investigate next?
A. Only the Git branch name
B. Service endpoints, ports, NetworkPolicies, and destination health
C. Only React CSS
D. Image signing alone

### Q7. What does a NetworkPolicy primarily control?
A. CPU allocation
B. Permitted network traffic to and from selected pods
C. Deployment revision history
D. Database schema migrations

### Q8. Why should MongoDB normally remain private in a production MERN architecture?
A. It should accept only the intended connections, rather than unnecessary public access
B. It cannot operate with Services
C. It must always run on the same node as React
D. It does not support authentication

### Q9. Your API Service has ready endpoints, but external requests fail. What should you inspect?
A. Ingress or Gateway routing, load balancer, DNS, and TLS
B. Only MongoDB indexes
C. Only container CPU limits
D. Git commit messages

### Q10. What is the best way to troubleshoot a failed production request?
A. Restart every pod immediately
B. Trace the request path and collect evidence at each networking layer
C. Disable all NetworkPolicies
D. Expose MongoDB publicly

## Answer key
QuestionAnswer1B2B3A4A5A6B7B8A9A10B

**Score guide**

- **9–10:** Excellent networking fundamentals.
- **7–8:** Good; review the missed concepts.
- **5–6:** Revisit Services, DNS, and routing.
- **Below 5:** Repeat today's lesson before advancing.

---

# 11. 💻 Practical Exercise — Trace a MERN Request
**Duration:** 5–7 minutes.

Your production architecture is:

```
shop.example.com
       |
       v
Load Balancer
       |
       v
Ingress
       |
       v
mern-api Service
       |
       v
Express API Pods
       |
       v
mongodb Service
       |
       v
MongoDB Pods
```

Customers report that the frontend loads, but checkout requests fail.

### Task A: Inspect the Services

```
kubectl get svc -n production
```

Check that both `mern-api` and `mongodb` exist and expose the intended ports.

### Task B: Inspect the backend endpoints

```
kubectl get endpointslices -n production \
  -l kubernetes.io/service-name=mern-api
```

```
kubectl get endpointslices -n production \
  -l kubernetes.io/service-name=mongodb
```

Check whether the Services have the expected ready endpoints.

### Task C: Inspect external routing

```
kubectl get ingress -n production
```

If your cluster uses Gateway API, inspect its Gateway and HTTPRoute resources as well.

Verify hostnames, paths, backend Services, and ports.

### Task D: Test DNS
From an approved diagnostic pod, resolve:

```
mern-api.production.svc.cluster.local
```

and:

```
mongodb.production.svc.cluster.local
```

Use the actual cluster DNS domain if it differs.

### Task E: Verify network policy
Inspect existing policies:

```
kubectl get networkpolicies -n production
```

Determine whether the required communication paths are permitted.

### Practical deliverable
Write a short incident note containing:

1. The failing request path.
2. The first component where the request fails.
3. The evidence that identified the problem.
4. The corrective action.
5. The checks used to verify recovery.

---

# 12. 🧪 Practical Assessment — The Broken Production Checkout
Your production MERN application reports:

```
Frontend: Accessible
API Service: Exists
API Pods: Running
API Ready Replicas: 3
MongoDB Service: Exists
MongoDB Endpoints: Empty
Checkout: Failing
```

You have five minutes to explain your investigation.

### Task 1 — What do you check first?
Inspect the MongoDB Service selector, pod labels, and readiness state.

A Service can exist without having any usable backend endpoints.

### Task 2 — What if the MongoDB pods have the wrong labels?
Correct the underlying workload labels or Service selector through a controlled configuration change.

Avoid manually changing live resources without updating the source configuration used by your deployment pipeline.

### Task 3 — What if MongoDB endpoints exist but the API still cannot connect?
Investigate:

- DNS resolution.
- Service and target ports.
- NetworkPolicies.
- Database listener configuration.
- Authentication and TLS.
- MongoDB health and connection limits.

### Task 4 — What if the API is accessible inside the cluster but not externally?
Inspect:

- Public DNS.
- Cloud load balancer.
- Ingress or Gateway status.
- Host and path routing.
- TLS configuration.
- External firewall and network settings.

### Task 5 — How do you verify recovery?
Confirm that the expected MongoDB endpoints exist, the API can connect to the database, checkout requests succeed, and user-facing error rates and latency return to acceptable levels.

### Assessment scoring
AreaPointsRequest-path analysis20Service and endpoint diagnosis20DNS and connectivity troubleshooting20Ingress/Gateway investigation20Recovery verification20**Total****100**

**90–100:** Strong production networking judgment.
**75–89:** Good; refine the order of investigation.
**60–74:** Review Service discovery and endpoint troubleshooting.
**Below 60:** Repeat the assessment after reviewing today's lesson.

---

# 13. 🏆 October Cumulative DevOps Project

## Project: Production-Ready MERN Application on Kubernetes
Continue your cumulative project by integrating the networking knowledge from Day 66 with the previous lessons.

Your platform should now incorporate:

- SLOs and alerting.
- Incident response and RCA.
- Backup and disaster recovery.
- Secrets management and IAM.
- Software supply-chain security.
- Kubernetes runtime security.
- Health probes and autoscaling.
- Production troubleshooting.
- Service discovery and external traffic routing.

### Target architecture

```
                 External Users
                       |
                       v
                  Public DNS
                       |
                       v
               Cloud Load Balancer
                       |
                       v
                Ingress / Gateway
                       |
                +------+------+
                |             |
                v             v
          React Service   Express Service
                              |
                       +------+------+
                       |             |
                       v             v
                  MongoDB Service  Redis Service
```

### New deliverables
Add or update these files:

FilePurpose`SERVICE-DISCOVERY.md`Document Service names, namespaces, ports, and selectors`INGRESS-ROUTING.md`Describe external hostname and path routing`GATEWAY-API.md`Document Gateway API adoption if applicable`NETWORK-POLICY.md`Specify permitted application traffic`NETWORK-TROUBLESHOOTING.md`Document DNS, endpoint, and connectivity diagnosis`DEPLOYMENT-RUNBOOK.md`Include post-deployment networking validation

### Acceptance criteria
Demonstrate in a controlled environment that:

1. External HTTP traffic reaches the intended application.
2. React can make requests to the Express API.
3. Express can resolve and connect to MongoDB.
4. Services select the intended backend pods.
5. NetworkPolicies restrict unauthorized communication.
6. Routing failures can be diagnosed systematically.
7. The application remains observable and recoverable during incidents.
**Project goal:** Build a production platform where networking is deliberate, restricted, observable, and testable—not dependent on accidental connectivity.

---

# 14. 🎤 Final Interview Challenge
**Question:**

> Your Kubernetes-hosted MERN application works inside the cluster but fails for external users. The frontend loads, checkout returns HTTP 503, and MongoDB appears healthy. Explain how you would troubleshoot the entire request path.
Structure your answer:

```
1. Establish user impact
2. Verify external DNS
3. Inspect the load balancer
4. Check Ingress or Gateway configuration
5. Verify the Express Service
6. Inspect EndpointSlices and pod readiness
7. Check NetworkPolicies
8. Test internal DNS resolution
9. Validate MongoDB connectivity
10. Verify recovery using application metrics
```

### Model interview answer
"I would first establish the scope of the outage and determine whether the problem affects all users or only specific routes. Since the frontend loads but checkout fails, I'd trace the API request through external DNS, the load balancer, and the Ingress or Gateway routing layer. I'd then inspect the Express Service, its selectors, and EndpointSlices to confirm that ready backend pods are available.

"If the API is reachable internally, I'd investigate the next failing connection, including MongoDB DNS resolution, Service endpoints, ports, and NetworkPolicies. I'd use application logs and connectivity tests to distinguish routing failures from database or application errors. After making a targeted fix, I'd verify checkout success, HTTP error rates, latency, and the health of the relevant Services."

---

# 🧠 Day 66 — Final Takeaways
Remember these seven principles:

1. **Services provide stable virtual endpoints for pods.**
2. **Kubernetes DNS enables service discovery using names and namespaces.**
3. **Ingress defines HTTP routing but requires an implementation to process it.**
4. **Gateway API offers a more expressive traffic-routing model.**
5. **NetworkPolicies restrict communication only when supported by the cluster's networking implementation.**
6. **A Service can exist without having ready backend endpoints.**
7. **Successful networking requires DNS, routing, endpoints, ports, policies, and application health to work together.**
Your production request path is:

```
External User
      |
      v
DNS Resolution
      |
      v
Load Balancer
      |
      v
Ingress / Gateway
      |
      v
Kubernetes Service
      |
      v
Ready Application Pod
      |
      v
Internal Service Discovery
      |
      v
MongoDB / Redis
      |
      v
Successful Business Operation
```

### ⭐ Interview takeaway
**Kubernetes networking troubleshooting is about tracing a request through every layer, finding the first point of failure, and fixing the underlying cause without weakening security.**

## Next syllabus lesson — Day 67
**Kubernetes Storage and Stateful Applications: PersistentVolumes, PersistentVolumeClaims, StorageClasses, StatefulSets, and MongoDB Production Data Protection.**

You'll learn how Kubernetes manages persistent application data, why containers should not store important database data in ephemeral filesystems, and how to design reliable storage for a production MERN application.