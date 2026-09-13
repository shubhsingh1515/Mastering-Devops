# Day 33 - Assignment: Production Configuration Management

## Objective

Update your Day 32 MERN deployment so that environment-specific values are externalized, local secrets are not committed, the API validates required configuration, and the complete request path can be diagnosed.

## Part 1: Create Environment Templates

Create `.env.example` in the project root:

```dotenv
NODE_ENV=development
PORT=3000
MONGO_URI=mongodb://mongodb:27017/mern
JWT_SECRET=replace-me-for-local-development
```

Create a local `.env` with values suitable for your machine. Do not commit it.

## Part 2: Update `.gitignore`

Add:

```gitignore
.env
.env.*
!.env.example
```

Prove the rule works:

```bash
git check-ignore -v .env
```

## Part 3: Configure Compose

Update the API service so it receives configuration from Compose:

```yaml
api:
  build: ./backend
  environment:
    NODE_ENV: ${NODE_ENV:?NODE_ENV must be provided}
    PORT: ${PORT:-3000}
    MONGO_URI: ${MONGO_URI:?MONGO_URI must be provided}
  networks:
    - mern-network
```

Keep MongoDB private. Do not publish `27017` unless you have a documented reason.

## Part 4: Verify Substitution

Run:

```bash
docker compose config
```

Confirm that the API receives the expected values. Redact secrets before sharing output.

## Part 5: Add Application Validation

Add startup validation to the Node API. It must:

- Require `NODE_ENV`.
- Require `MONGO_URI`.
- Validate that `PORT` is a valid TCP port.
- Accept only `development`, `staging`, or `production` for `NODE_ENV`.
- Fail with a clear error instead of silently using an unsafe production default.

Expected missing-variable behavior:

```text
MONGO_URI is required
Container exits
```

Do not print the value of any secret.

## Part 6: Start and Verify

Run:

```bash
docker compose up -d --build
docker compose ps
docker compose logs --tail 100 api
curl http://localhost/api/health
```

Document this request path:

```text
Client -> Nginx -> api:3000 -> mongodb:27017
```

## Part 7: Break the Configuration Intentionally

Change the local value to:

```dotenv
MONGO_URI=mongodb://localhost:27017/mern
```

Restart the API:

```bash
docker compose up -d api
docker compose logs --tail 100 api
```

Explain why `localhost` is wrong inside the API container. Restore:

```dotenv
MONGO_URI=mongodb://mongodb:27017/mern
```

Restart and verify recovery.

## Part 8: Document Frontend Configuration

Add a README section called `Configuration Management` that explains:

- Development configuration.
- Staging configuration.
- Production configuration.
- Required environment variables.
- Secret handling.
- Backend runtime configuration.
- Frontend build-time configuration.
- A runtime `config.js` approach for a single promoted frontend artifact.

## Part 9: Deliverables

Submit:

1. `.env.example` with safe placeholders.
2. Updated `.gitignore`.
3. Updated Compose configuration.
4. Node startup validation.
5. A successful `docker compose config` result.
6. Evidence that the API health path works.
7. A short troubleshooting note for the intentionally broken Mongo URI.
8. The README `Configuration Management` section.

## Success Criteria

- `.env` is not tracked by Git.
- `.env.example` contains no real secrets.
- Compose resolves required variables correctly.
- The API uses `mongodb:27017`, not `localhost:27017`, for MongoDB.
- Missing required configuration causes a clear startup failure.
- The same image can conceptually run with different environment values.
- The difference between backend runtime and frontend build-time configuration is documented.
