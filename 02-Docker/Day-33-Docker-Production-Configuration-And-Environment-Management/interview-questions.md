# Day 33 - Interview Questions: Docker Configuration and Environment Management

## Beginner Level

### Q1. What is environment-specific configuration?

Configuration that changes according to where an application runs, such as development, staging, or production. Examples include database endpoints, logging levels, feature flags, and service URLs.

### Q2. What is a `.env` file?

A file containing environment-variable assignments that can be loaded by tooling or an application. It is convenient for local development, but it is not automatically a secure secret manager.

### Q3. Why should `.env` usually be ignored by Git?

It may contain passwords, tokens, private keys, or machine-specific values. Committing it can expose secrets in the repository and its history.

### Q4. Why provide `.env.example`?

It documents the expected variables and gives developers a safe starting template without exposing real credentials.

## Intermediate Level

### Q5. What does `docker compose config` do?

It renders and validates the resolved Compose model after interpolation. It is useful for checking whether `.env` values were loaded and substituted correctly.

### Q6. Why should production configuration not be baked into a Docker image?

It couples the image to one environment, makes promotion and rollback harder, and can expose sensitive values in image layers or metadata.

### Q7. What is the correct MongoDB URI inside a Compose network?

```text
mongodb://mongodb:27017/mern
```

`mongodb` is the Compose service name. `localhost` inside the API container refers to the API container itself.

### Q8. What is the difference between configuration and secrets?

Configuration affects behavior but may not be confidential. Secrets are confidential values that require stronger access controls, secure storage, auditing, and rotation.

### Q9. Why might the API have no published host port?

If Nginx is the only public entry point, it can reach the API through `api:3000` on the private Docker network. Publishing the API port is unnecessary exposure.

## Advanced Level

### Q10. Why can backend environment variables change at container startup?

The backend process reads `process.env` at runtime. The same image can be started with different environment values in different deployments.

### Q11. Why can React environment variables often not be changed after the image is built?

Typical React/Vite builds substitute environment values into static JavaScript during compilation. Changing the container environment later does not rewrite those already-generated assets.

### Q12. How can one frontend artifact use different API URLs?

Use a deliberate runtime configuration strategy, such as a deployment-generated `config.js` loaded before the frontend bundle. Never put secrets in browser-delivered configuration.

### Q13. What should happen when `MONGO_URI` is missing?

The application should fail fast with a clear error such as `MONGO_URI is required`. Starting with an invalid or arbitrary production default creates a less diagnosable failure.

### Q14. Is a running container necessarily healthy?

No. `Up` only indicates that the container process has not exited. The application may still be unable to connect to MongoDB or serve requests. Health checks and endpoint tests are needed.

### Q15. How would you troubleshoot a MongoDB connection failure?

1. Run `docker compose config`.
2. Check `docker compose ps -a`.
3. Read API and MongoDB logs.
4. Confirm `MONGO_URI` uses `mongodb`, not `localhost`.
5. Confirm both services share a network.
6. Resolve the service name from the API container.
7. Check MongoDB readiness and credentials.
8. Retest the API health endpoint.

### Q16. How would you protect production secrets?

Store them in a managed secret system or protected CI/CD variables, restrict access, inject them at deployment time, rotate them, and prevent them from appearing in logs, images, source code, and diagnostic output.

## Scenario Question

> The API container is `Up`, but logs show `MongoServerSelectionError`. What do you investigate first?

A strong answer:

```text
Compose interpolation
       |
MONGO_URI inside the API
       |
Docker DNS for mongodb
       |
MongoDB port and readiness
       |
Credentials and application logs
```

I would first run `docker compose config` and verify that the resolved URI points to `mongodb:27017`. Then I would inspect service state, network membership, and logs.

## Strong Short Answers

**Why not `localhost` for MongoDB?**  
Because `localhost` refers to the current container, not another Compose service.

**What is the immutable-artifact principle?**  
Build and test one application artifact, then inject environment-specific configuration when deploying it.

**Is `.env` secure by itself?**  
No. It is only a convenient configuration file.

**Why use `.env.example`?**  
It documents required variables without exposing real values.

**Backend versus frontend configuration?**  
Backend values are usually read at runtime; frontend values are often embedded during the build.

**What is the first Compose configuration diagnostic?**  
`docker compose config`.
