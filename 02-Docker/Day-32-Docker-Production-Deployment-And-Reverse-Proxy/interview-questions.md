# Day 32 - Interview Questions: Docker Reverse Proxy and Production Deployment

## Beginner Level

### Q1. What is a reverse proxy?

A server that receives client requests and forwards them to backend services. Nginx can be the public entry point while application services remain private.

### Q2. Why use Nginx in front of Node.js?

Nginx can provide reverse proxying, TLS termination, static-file serving, routing, headers, request handling, and load balancing.

### Q3. What is the correct address for Nginx to reach an API service named `api`?

```text
http://api:3000
```

Nginx and the API are separate containers, so `localhost` inside Nginx points to Nginx itself.

### Q4. Why should MongoDB usually remain private?

The API can reach it through the internal Docker network. Publishing the database unnecessarily increases exposure and attack surface.

---

## Intermediate Level

### Q5. What does `502 Bad Gateway` mean?

It generally means the proxy could not obtain a valid response from its upstream. Possible causes include an unavailable API, wrong service name, network failure, wrong port, invalid Nginx configuration, or an unhealthy backend.

### Q6. Why are service names preferable to container IPs?

Service names provide stable logical discovery through Docker DNS. Container IPs can change when services are recreated.

### Q7. Why might the API have no `ports` section in Compose?

If Nginx is the only service that needs to reach the API, Nginx can use `api:3000` over the private network. The API does not need a host-published port.

### Q8. What is the difference between a host port and a container port?

A host port is exposed on the Docker host. A container port is where the process listens inside the container. Container-to-container communication normally uses the service name and container port.

### Q9. Why build React before serving it through Nginx?

The production frontend normally needs static assets, not a development server or the complete Node build environment. A multi-stage build keeps build tools out of the runtime image.

---

## Advanced Level

### Q10. How would you troubleshoot a production `502`?

1. Check Compose service state.
2. Read Nginx logs.
3. Read API logs.
4. Validate Nginx configuration.
5. Confirm Nginx and API share a network.
6. Resolve `api` from inside Nginx.
7. Test `http://api:3000/health` from Nginx.
8. Confirm the API listening port and interface.
9. Check API dependencies and readiness.
10. Retest the public request path.

### Q11. Is a `502` definitely an Nginx bug?

No. It can originate in Nginx configuration, Docker networking, DNS/service discovery, API availability, the listening port, application health, or a dependency failure.

### Q12. How does this architecture support scaling?

Nginx provides a stable public boundary. Multiple API instances can later sit behind Nginx or a load balancer, with health-based routing and rolling deployment strategies.

### Q13. How would you test the upstream independently of Nginx?

From inside the Nginx container, resolve and request the API:

```bash
docker compose exec nginx getent hosts api
docker compose exec nginx wget -qO- http://api:3000/health
```

This separates upstream connectivity from public proxy behavior.

### Q14. What does a complete request path look like?

```text
Browser -> Nginx -> api:3000 -> mongodb:27017
```

The response returns through the reverse path. Each boundary can be tested independently.

### Q15. Why should `depends_on` not be treated as readiness?

It expresses a dependency or startup relationship, but a service may still be initializing or unable to serve requests. Health checks and application retry behavior are needed.

## Scenario Question

> Nginx returns `502`, but the API container is `Up`. What do you check?

A strong answer:

```text
Nginx logs
    |
    v
API logs
    |
    v
Network membership
    |
    v
DNS resolution for api
    |
    v
API port and listener
    |
    v
API health endpoint
    |
    v
MongoDB dependency
```

I would not assume that `Up` means healthy. I would test the request path from Nginx to the API and then from the API to MongoDB.

## Short Strong Answers

**Why not `localhost` in Nginx?**  
Because it refers to the Nginx container, not the API container.

**What does `api:3000` provide?**  
Docker service-name discovery and direct internal access to the API container port.

**Why keep MongoDB private?**  
The API is its intended client, so public exposure is unnecessary risk.

**How do you debug 502?**  
Trace client, Nginx, network, DNS, API port, health, and database in order.
