# Day 25 - Interview Questions: Docker Compose

## Beginner Level

### Q1. What is Docker Compose?

Docker Compose is a tool for defining and running multi-container applications from a declarative YAML configuration.

### Q2. What is a service in Compose?

A service is a logical application component definition. It specifies details such as the image or build context, ports, environment, volumes, and dependencies.

### Q3. How does a Node API find MongoDB in Compose?

It uses MongoDB's Compose service name through the shared Docker network:

```text
mongodb://mongodb:27017/mern
```

### Q4. What does `docker compose up -d` do?

It creates and starts the services in detached mode so the terminal is returned to the user.

## Intermediate Level

### Q5. What is the difference between a Compose service and a container?

The service is the desired logical configuration. A container is a running instance created from that configuration. One service can be scaled to multiple containers.

### Q6. What does `depends_on` do?

It expresses a relationship and, depending on its form, startup ordering or health-based startup coordination. A simple `depends_on` entry should not be treated as proof that the dependency is ready to accept application traffic.

### Q7. Why is `localhost` commonly wrong in a container-to-container connection string?

Because `localhost` refers to the current container. A peer container should be addressed by its service name on the shared network.

### Q8. How do you persist MongoDB data in Compose?

Mount a named volume at MongoDB's data directory:

```yaml
volumes:
  - mongo-data:/data/db
```

Then declare the named volume at the top level.

### Q9. Why should MongoDB usually not have a host port mapping?

The API can reach it over the private Compose network. Publishing the database port unnecessarily increases exposure and attack surface.

## Advanced Level

### Q10. A Compose stack starts, but the API intermittently cannot connect to MongoDB. What could be happening?

The MongoDB container may have started before the database process was ready. Investigate health checks, dependency conditions, application retry and backoff logic, service names, environment variables, and both services' logs.

### Q11. Does a health check guarantee that the application will never lose its database connection?

No. It reports the result of a defined test at a point in time. The application still needs to handle later outages, reconnects, timeouts, and transient failures.

### Q12. What is the difference between a volume and a backup?

A volume provides persistence for normal container replacement. A backup is an independent copy and recovery process for deletion, corruption, disaster, or operational mistakes.

### Q13. How would you troubleshoot an API that cannot resolve `mongodb`?

1. Confirm both services are running with `docker compose ps`.
2. Confirm the hostname exactly matches the service name.
3. Confirm both services share a network.
4. Inspect the resolved configuration with `docker compose config`.
5. Inspect API and MongoDB logs.
6. Check the API environment variable from inside the container.
7. Test name resolution or connectivity from the API container.

### Q14. What is a production-style entry point for a MERN Compose deployment?

Nginx can be the public entry point, route frontend and API traffic, and keep the API and MongoDB on private internal network paths. The exact topology depends on the deployment requirements.

### Q15. Where should production secrets come from?

They should come from a suitable secret-management or protected configuration mechanism. They should not be hardcoded in source control or baked into an image.

## Strong Short Answers

**Why does `localhost` fail?**  
Because it points to the current container, not the peer service.

**Why use service names?**  
Compose provides internal DNS, and service names remain stable when container IPs change.

**Why use a named volume?**  
To keep database data independent from the replaceable container filesystem.

**Does `depends_on` mean ready?**  
Not by itself. Readiness requires a meaningful health check and resilient application behavior.

**Why not publish MongoDB?**  
The API can use the private network, so exposing the database is usually unnecessary.
