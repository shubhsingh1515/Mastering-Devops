# Day 31 - Assignment: Production-Ready Container Review

## Objective

Review and improve the Day 30 MERN Docker project so that its images and runtime behavior are reproducible, secure, observable, and easier to operate.

---

## Part 1: Review the Dockerfile

Start with this Dockerfile:

```dockerfile
FROM node:22
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]
```

Identify at least five problems.

### Expected areas

- unnecessarily large or unreviewed base image
- source copied before dependency manifests
- less reproducible dependency installation
- possible root execution
- missing production-only dependency selection
- no `.dockerignore` evidence
- possible accidental secret or artifact inclusion
- no clear health or shutdown design

---

## Part 2: Improve the Backend Dockerfile

Create a production-oriented version:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY . .

USER node

EXPOSE 3000

CMD ["node", "server.js"]
```

Adapt the command and runtime files to your application. Do not copy only `server.js` if the API requires routes, models, configuration modules, or compiled output.

Build it:

```bash
docker build --progress=plain -t mern-api:production-review .
```

Inspect it:

```bash
docker images
docker history mern-api:production-review
```

Record the image size and build behavior.

---

## Part 3: Add `.dockerignore`

Create:

```text
node_modules
.git
.env
coverage
npm-debug.log
```

Add other files that are unnecessary or sensitive for your project. Explain why `.dockerignore` reduces build context and accidental secret inclusion but does not replace runtime secret management.

---

## Part 4: Verify Non-Root Execution

Run the image and check identity:

```bash
docker run -d --name mern-api-review -p 3000:3000 mern-api:production-review
docker exec mern-api-review whoami
docker exec mern-api-review id
```

Expected result: the application should run as a restricted user where practical.

Document any file-permission changes required to make non-root execution work.

---

## Part 5: Add and Test Health

Implement:

```text
GET /health
```

Return HTTP 200 and:

```json
{
  "status": "ok"
}
```

Test:

```bash
curl http://localhost:3000/health
```

Add a Compose or Docker health check. Confirm that its command exists in the image and that it tests a meaningful condition.

---

## Part 6: Implement Graceful Shutdown

Add a `SIGTERM` handler to the Node application.

The handler should:

1. log shutdown start
2. stop accepting new requests
3. finish active requests where possible
4. close MongoDB connections
5. close the HTTP server
6. exit cleanly

Test:

```bash
docker stop mern-api-review
docker logs mern-api-review
```

Record the shutdown messages and explain the sequence from signal to process exit.

Also consider a timeout so a stuck dependency cannot prevent termination forever.

---

## Part 7: Test Immutability

Do not install packages or edit configuration manually inside the running container.

Instead:

1. Change the Dockerfile or application source.
2. Build a new image tag.
3. Run a replacement container.
4. Test it.
5. Remove the old container only after verification.

Explain why this is more reproducible than modifying a live container.

---

## Part 8: Apply Runtime Controls

Test the application with:

```bash
docker run -d \
  --name mern-api-hardened \
  --read-only \
  --cap-drop=ALL \
  --tmpfs /tmp \
  --memory=512m \
  --cpus=1 \
  -p 3000:3000 \
  mern-api:production-review
```

If it fails, determine exactly what it needs to write or which capability it needs. Document the smallest exception rather than disabling every control.

---

## Part 9: Review Configuration and Secrets

Confirm that:

- the image does not contain `.env`
- credentials are not in Dockerfile instructions
- production configuration is injected at runtime
- the same image can run in staging and production
- image tags identify the source revision

Use safe placeholder configuration in documentation. Never commit real credentials.

---

## Part 10: Monitor and Inspect

Run:

```bash
docker logs --tail 100 mern-api-review
docker stats --no-stream
docker inspect mern-api-review
docker top mern-api-review
```

Record:

- CPU usage
- memory usage and limit
- health status
- restart policy
- user identity
- mounts and networks
- log behavior

---

## Part 11: Production Review Checklist

Mark each item and add evidence:

```text
[ ] Package files are copied before source
[ ] Lockfile-based installation is used
[ ] Production dependencies are selected
[ ] Image base is appropriate and maintained
[ ] .dockerignore exists
[ ] Secrets are excluded from images and Git
[ ] Application runs as non-root
[ ] Runtime image excludes build tooling where practical
[ ] Health endpoint exists
[ ] Health check command exists in the image
[ ] SIGTERM is handled gracefully
[ ] Container uses an explicit runtime command
[ ] Restart policy is documented
[ ] CPU and memory limits are considered
[ ] Logs go to stdout/stderr
[ ] Image tag is traceable
[ ] Running containers are not manually modified
[ ] MongoDB data is persistent
[ ] MongoDB is not unnecessarily exposed
[ ] Rollback image is retained
```

The goal is to identify remaining work, not to pretend every control is automatically appropriate.

---

## Part 12: Interview Deliverable

Write a short production review of your Day 30 project answering:

1. Why did you select the base images?
2. How does your Dockerfile use cache effectively?
3. How are secrets supplied?
4. Which process runs as PID 1?
5. How does the API handle `SIGTERM`?
6. How does Docker determine application health?
7. What happens if the API crashes repeatedly?
8. What resource limits are configured and why?
9. How can you identify the exact image in production?
10. How do you replace the container without losing persistent data?

---

## Submission Checklist

- [ ] Dockerfile reviewed and improved
- [ ] `.dockerignore` created
- [ ] Image rebuilt and inspected
- [ ] Image history reviewed
- [ ] Non-root user verified
- [ ] Health endpoint tested
- [ ] Health check configured
- [ ] Graceful shutdown implemented
- [ ] `docker stop` behavior verified
- [ ] Runtime hardening tested
- [ ] Resource usage inspected
- [ ] Secrets reviewed
- [ ] Immutability workflow documented
- [ ] Production review checklist completed
- [ ] Interview deliverable written
