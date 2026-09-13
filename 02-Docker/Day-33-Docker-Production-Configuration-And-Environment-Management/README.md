# DevOps Mentorship Program - Day 33

## Phase 2: Docker & Containers

### Docker Production Configuration and Environment Management

**Level:** Intermediate to Professional    
**Focus:** Environment-specific configuration, `.env`, Docker Compose interpolation, runtime versus build-time configuration, and production-safe MERN deployments.

> Today continues directly from Day 32, where Nginx was placed in front of the Dockerized MERN stack. The goal is to keep application artifacts independent from environment-specific configuration.

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain why configuration should be separated from Docker images.
- Describe the difference between configuration and secrets.
- Use `.env` files with Docker Compose safely.
- Understand Compose variable interpolation and `docker compose config`.
- Distinguish backend runtime configuration from frontend build-time configuration.
- Configure a MERN application for development, staging, and production.
- Prevent real credentials from being committed to Git.
- Validate required Node.js environment variables and fail fast.
- Troubleshoot a container that starts with an invalid MongoDB URI.
- Explain immutable artifacts and configuration injection in a DevOps interview.

---

## 2. The Architecture We Are Building Toward

```text
                         Internet
                             |
                             v
                      +---------------+
                      |     Nginx     |
                      |    :80/:443   |
                      +-------+-------+
                              |
                              v
                      +---------------+
                      |    Node API   |
                      |   api:3000    |
                      +-------+-------+
                              |
                              v
                      +---------------+
                      |    MongoDB    |
                      | mongodb:27017|
                      +-------+-------+
                              |
                              v
                       Persistent volume
```

Nginx is public. The API and MongoDB communicate over a private Docker network. MongoDB data lives in a persistent volume. Day 33 adds a fourth concern: the values that change between environments must be supplied separately from the application image.

A production deployment should conceptually look like this:

```text
                         Same image
                       mern-api:abc123
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
            Dev            Staging          Prod
        Config A         Config B         Config C
        Secrets A        Secrets B        Secrets C
```

The tested application artifact stays the same. Only its environment-specific configuration and secrets change.

---

## 3. Why Configuration Management Matters

Suppose you build this image:

```text
mern-api:1.0
```

You need to run it in development, staging, and production. A weak approach is to create three separate images:

```text
mern-api-development
mern-api-staging
mern-api-production
```

That couples the artifact to a deployment environment. It also means each environment may be running code built differently, which makes testing and promotion less reliable.

A stronger approach is:

```text
Build once -> Test -> Promote the same artifact
```

Then inject values at deployment time:

```text
Application image
      +
Environment configuration
      +
Secret values
      =
Running service
```

This is an important DevOps principle:

> Separate the application artifact from environment configuration.

Benefits include:

- One artifact can be promoted from staging to production.
- Production database addresses do not need to be baked into an image.
- Configuration changes do not require rebuilding application code.
- Rollbacks can use a previously tested image with the current environment configuration.
- The image can be scanned and signed independently from deployment settings.

---

## 4. Configuration Versus Secrets

Configuration and secrets are related, but they do not have the same protection requirements.

### Normal configuration

Examples:

```text
NODE_ENV=production
PORT=3000
LOG_LEVEL=info
API_TIMEOUT=5000
```

These values can still affect application behavior, but they may not be confidential.

### Secrets

Examples:

```text
MONGO_PASSWORD
JWT_SECRET
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
API_SECRET
```

Secrets require stronger controls: restricted access, rotation, auditing, encryption at rest, and careful handling in logs and support tickets.

A useful mental model is:

```text
Configuration -> environment/configuration system
Secrets       -> secret-management system
```

A local `.env` file is convenient for development. It is not automatically a secure production secret manager. Anyone who can read that file can read its values, and careless commands can print them.

---

## 5. How `.env` and Docker Compose Work

Docker Compose can substitute variables from a local `.env` file into `compose.yaml` or `docker-compose.yml`.

Example `.env`:

```dotenv
NODE_ENV=development
PORT=3000
MONGO_URI=mongodb://mongodb:27017/mern
```

Example Compose service:

```yaml
services:
  api:
    build: ./backend
    environment:
      NODE_ENV: ${NODE_ENV}
      PORT: ${PORT}
      MONGO_URI: ${MONGO_URI}
```

The flow is:

```text
.env file
   |
   v
Compose interpolation
   |
   v
Container environment
   |
   v
Node.js process.env
```

Inside Node.js, the application reads values like this:

```javascript
const mongoUri = process.env.MONGO_URI;
const port = Number(process.env.PORT || 3000);
```

### Important distinction

Compose interpolation happens when Compose processes the configuration. The `.env` file is not magically copied into the image. The values are resolved and supplied to the container according to the Compose file.

### Default values

Compose supports defaults:

```yaml
environment:
  NODE_ENV: ${NODE_ENV:-development}
  PORT: ${PORT:-3000}
```

For required values, use a mandatory interpolation expression:

```yaml
environment:
  MONGO_URI: ${MONGO_URI:?MONGO_URI must be provided}
```

This makes Compose stop early when the value is missing instead of starting a broken service.

---

## 6. `.env` Is Not a Secret Manager

This statement is important in interviews and real deployments:

> A `.env` file is a configuration input format, not a production-grade secret-management system.

For local development, `.env` is practical because it keeps personal values out of the Compose file. For production, use the secret mechanism provided by your platform, such as a CI/CD secret store, cloud secret manager, orchestrator secret, or managed configuration service.

Never assume a value is safe merely because it came from `.env`. It can still be exposed by:

- Committing the file to Git.
- Printing `process.env` in application logs.
- Running `docker inspect` and sharing the output.
- Running `docker exec <container> env` in a recorded terminal session.
- Including secrets in image layers, shell history, tickets, or screenshots.

---

## 7. `.gitignore` and `.env.example`

A repository should generally ignore local environment files:

```gitignore
.env
.env.*
!.env.example
```

The exception keeps a safe template available to other developers.

Example `.env.example`:

```dotenv
NODE_ENV=development
PORT=3000
MONGO_URI=mongodb://mongodb:27017/mern
JWT_SECRET=replace-me-for-local-development
```

The example file should document variable names and safe placeholder values. It must not contain real passwords, tokens, private keys, or production connection strings.

A normal local setup is:

```text
.env.example -> committed template
.env         -> local, ignored values
```

Before committing, check the repository:

```bash
git status --short
git check-ignore -v .env
```

If a real secret was ever committed, deleting the file in a later commit is not enough. Treat the secret as exposed, rotate it, and remove it from repository history using an approved process.

---

## 8. Development, Staging, and Production Configuration

The values can vary by environment while the image remains unchanged.

```text
Development:
  NODE_ENV=development
  LOG_LEVEL=debug
  MONGO_URI=mongodb://mongodb:27017/mern

Staging:
  NODE_ENV=staging
  LOG_LEVEL=info
  MONGO_URI=<staging-managed-value>

Production:
  NODE_ENV=production
  LOG_LEVEL=warn
  MONGO_URI=<production-managed-value>
```

The exact storage mechanism can differ:

- Development: local `.env` file.
- Staging: CI/CD variables or a staging secret store.
- Production: managed configuration and secret services with access control and rotation.

Do not confuse separate configuration with separate code. The application should be built and tested as one artifact whenever possible.

---

## 9. Build-Time Versus Runtime Configuration

This is one of the most important concepts in this lesson.

### Backend runtime configuration

A Node.js backend reads environment variables while the process starts or runs:

```javascript
const mongoUri = process.env.MONGO_URI;
```

The same image can receive different values when it starts:

```text
mern-api image
   |
   +--> Dev:   MONGO_URI=A
   +--> Stage: MONGO_URI=B
   +--> Prod:  MONGO_URI=C
```

Changing the container environment changes what the backend process reads on its next start. The image itself does not need to change.

### Frontend build-time configuration

A typical React/Vite frontend may contain:

```javascript
const apiUrl = import.meta.env.VITE_API_URL;
```

During `npm run build`, the value is commonly substituted into generated JavaScript and static assets:

```text
React source
     |
     v
npm run build
     |
     v
JavaScript bundle with embedded value
     |
     v
Nginx serves static files
```

Changing `VITE_API_URL` after the bundle has already been built does not normally rewrite the browser assets. This is why frontend configuration is often build-time configuration.

### Runtime frontend configuration

If one identical frontend artifact must be promoted through multiple environments, use an explicit runtime mechanism. One pattern is a generated `config.js` file:

```javascript
window.APP_CONFIG = {
  API_URL: "https://api.example.com"
};
```

The HTML loads this file before the application bundle, and the application reads `window.APP_CONFIG.API_URL`. A deployment step can generate or inject this non-secret value without rebuilding the entire frontend.

Never place private secrets in browser configuration. Anything delivered to a browser is visible to the user.

### Strong interview answer

> A backend process reads environment variables at runtime, so startup values can differ while the image remains the same. A typical React production build embeds frontend environment values into static assets during compilation, so changing the container environment afterward does not automatically change what the browser uses.

---

## 10. Production-Style Compose Configuration

A simplified Compose design can look like this:

```yaml
services:
  nginx:
    build: ./nginx
    ports:
      - "80:80"
    depends_on:
      - api
    networks:
      - mern-network

  api:
    build: ./backend
    environment:
      NODE_ENV: ${NODE_ENV:?NODE_ENV must be provided}
      PORT: 3000
      MONGO_URI: ${MONGO_URI:?MONGO_URI must be provided}
    networks:
      - mern-network

  mongodb:
    image: mongo:8
    volumes:
      - mongo-data:/data/db
    networks:
      - mern-network

volumes:
  mongo-data:

networks:
  mern-network:
```

Notice the intended exposure model:

```text
Public:
  Nginx -> host port 80/443

Private:
  Nginx -> api:3000
  API   -> mongodb:27017
```

The API does not need a published host port if Nginx is its only client. MongoDB should not have a public `27017:27017` mapping unless there is a specific, controlled operational reason.

`depends_on` controls startup ordering, but it does not prove readiness. The API should still handle MongoDB startup timing through health checks, retries, or a clear startup failure.

---

## 11. `docker compose config`: The First Diagnostic Tool

Use:

```bash
docker compose config
```

This renders the resolved Compose configuration after variable substitution. It helps answer questions such as:

- Did Compose load the expected `.env` file?
- Was `${MONGO_URI}` substituted?
- Did a variable become blank?
- Is the service connected to the expected network?
- Is a port published unexpectedly?

Useful variants:

```bash
docker compose config --services
docker compose config --environment
```

Be careful when sharing output. Resolved configuration can contain secrets. Prefer redacting sensitive values before putting output into tickets or chat.

A good troubleshooting sequence is:

```text
Check .env
   |
Check Compose references
   |
docker compose config
   |
Inspect service state and logs
   |
Test the application path
```

---

## 12. Inspecting Container Environment Safely

For a non-sensitive variable:

```bash
docker compose exec api printenv NODE_ENV
```

To inspect a specific container:

```bash
docker inspect <container>
```

To view all environment variables:

```bash
docker compose exec api env
```

Do not casually dump all environment variables in production. Prefer checking one known non-secret variable at a time, and redact sensitive values in any captured output.

Remember that environment variables are not automatically encrypted merely because they are inside a running container. Use a suitable secret mechanism for production and limit who can inspect the process environment.

---

## 13. Node.js Configuration Validation

An API should fail fast when required configuration is missing. Otherwise, it may start successfully and fail later with a confusing database or authentication error.

Simple validation:

```javascript
const required = ["NODE_ENV", "MONGO_URI"];

for (const variable of required) {
  if (!process.env[variable]) {
    console.error(`${variable} is required`);
    process.exit(1);
  }
}
```

A more useful validator can check values and types without printing secrets:

```javascript
const required = ["NODE_ENV", "MONGO_URI"];

for (const variable of required) {
  if (!process.env[variable]) {
    throw new Error(`${variable} is required`);
  }
}

const port = Number(process.env.PORT || 3000);
if (!Number.isInteger(port) || port < 1 || port > 65535) {
  throw new Error("PORT must be a valid TCP port");
}

const allowedEnvironments = ["development", "staging", "production"];
if (!allowedEnvironments.includes(process.env.NODE_ENV)) {
  throw new Error("NODE_ENV must be development, staging, or production");
}
```

A clear startup failure is preferable to a partially configured service:

```text
MONGO_URI is required
Container exits
```

This is especially valuable in automated deployments, where a failed release should be visible immediately.

---

## 14. Configuration Troubleshooting Scenario

### Symptom

The API container is `Up`, but its logs show a MongoDB connection error such as `MongoServerSelectionError`.

### Do not assume

Do not immediately conclude that MongoDB itself is broken. The API may have received the wrong URI.

### Common mistake

```dotenv
MONGO_URI=mongodb://localhost:27017/mern
```

Inside the API container, `localhost` means the API container itself, not the MongoDB container.

### Correct Compose address

```dotenv
MONGO_URI=mongodb://mongodb:27017/mern
```

`mongodb` is the Compose service name, and Docker's internal DNS resolves it on the shared network.

### Investigation

```bash
docker compose config
docker compose ps -a
docker compose logs --tail 100 api
docker compose logs --tail 100 mongodb
docker compose exec api printenv MONGO_URI
docker network ls
docker network inspect <project>_mern-network
```

Check the full path:

```text
.env value
   |
Compose substitution
   |
API container environment
   |
Node process.env.MONGO_URI
   |
MongoDB service DNS and port
```

This combines Day 23 networking knowledge with today's configuration-management concepts.

---

## 15. Common Mistakes

### Mistake 1: Committing `.env`

Real credentials may become part of the repository history. Ignore local files and rotate any exposed secret.

### Mistake 2: Treating `.env` as security

`.env` is a convenient input file, not access control, encryption, auditing, or secret rotation.

### Mistake 3: Using `localhost` for another Compose service

Use the service name, such as `mongodb:27017`, for container-to-container communication.

### Mistake 4: Baking production configuration into an image

This couples the artifact to one environment and may expose sensitive values in layers.

### Mistake 5: Printing all environment variables

Logs, screenshots, and support tickets can expose secrets. Inspect only what is necessary.

### Mistake 6: Rebuilding independently for every environment

Prefer build once and promote the tested image when the application and platform allow it.

### Mistake 7: Assuming frontend variables are runtime variables

Typical React/Vite values are embedded during the frontend build. Use a deliberate runtime configuration pattern when needed.

### Mistake 8: Assuming `docker compose up` proves readiness

A container can be running while the application is unable to serve requests or connect to dependencies. Use health checks and application-level validation.

### Mistake 9: Supplying empty values silently

Use required Compose interpolation such as `${MONGO_URI:?MONGO_URI must be provided}` or validate in the application.

---

## 16. Production Design Challenge

Suppose the registry contains one image:

```text
mern-api:7f81a2c
```

Design the deployment like this:

```text
                         mern-api:7f81a2c
                                |
                +---------------+---------------+
                |               |               |
                v               v               v
              DEV            STAGE            PROD
                |               |               |
             Config A        Config B        Config C
                |               |               |
             Secrets A       Secrets B       Secrets C
```

Your design should answer:

1. Where does each environment's configuration live?
2. Where do secrets live?
3. Who can read or change production values?
4. How are values injected during deployment?
5. How do you prevent secrets from appearing in logs?
6. How do you roll back the application image without rebuilding it?
7. How do you distinguish a bad image from a bad configuration?

A strong answer says the image is immutable and promoted unchanged. Environment configuration and secrets are injected by the deployment system, validated before startup, and protected by access control.

---

## 17. Summary

The most important ideas from Day 33 are:

```text
Application image
       +
Environment configuration
       +
Secrets
       =
Running service
```

```text
Backend:
  process.env -> runtime configuration

Frontend:
  environment variable -> build -> static bundle
```

```text
.env with real secrets -> do not commit
.env.example           -> safe committed template
```

```text
Build once -> test -> promote the same artifact
```

For Compose troubleshooting, start with:

```bash
docker compose config
docker compose ps
docker compose logs api
```

For the MERN network, remember:

```text
API -> mongodb:27017
Nginx -> api:3000
```

Tomorrow's lesson builds on this configuration foundation with Compose production patterns, health checks, restart behavior, and deployment resilience.
