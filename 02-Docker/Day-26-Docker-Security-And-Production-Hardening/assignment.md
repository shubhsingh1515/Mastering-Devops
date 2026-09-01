# Day 26 - Assignment: Docker Security & Production Hardening

## Objective

Harden a Node.js application container by reducing privilege, reducing attack surface, controlling runtime access, and improving production readiness.

---

## Part 1: Review the Weak Container

Start with this Dockerfile:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY . .
RUN npm ci

CMD ["node", "server.js"]
```

### Questions

1. Why is this less secure than a hardened container?
2. What is the main risk if the application runs as root?
3. Why is it better to separate runtime secrets from the image?

### Expected answers

- It likely runs as root and may have unnecessary privileges.
- The process can access more of the filesystem and may do more destructive actions if compromised.
- Secrets injected at build time become part of the image and are more easily leaked.

---

## Part 2: Run the Container as Non-Root

Update the Dockerfile to:

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

### Question

Why does this improve security?

### Expected answer

Because the application process runs with a limited user account instead of root, reducing the impact of a container compromise.

---

## Part 3: Add a `.dockerignore`

Create a `.dockerignore` file:

```text
node_modules
.git
.env
npm-debug.log
coverage
```

### Question

Why is this useful?

### Expected answer

It reduces image size, avoids copying local secrets or developer artifacts, and limits the attack surface.

---

## Part 4: Add Hardening Runtime Flags

Build and run:

```bash
docker build -t mern-api:secure .

docker run -d \
  --name mern-api-secure \
  --read-only \
  --cap-drop=ALL \
  --memory=512m \
  --cpus=1 \
  --tmpfs /tmp \
  -p 3000:3000 \
  mern-api:secure
```

### Question

What do these settings aim to accomplish?

### Expected answer

- `--read-only` prevents unauthorized root filesystem writes.
- `--cap-drop=ALL` removes Linux capabilities not explicitly required.
- `--memory` and `--cpus` bound resource consumption.
- `--tmpfs /tmp` provides a writable temporary area without making the whole filesystem writable.

---

## Part 5: Test the Health Endpoint

```bash
curl http://localhost:3000/health
```

### Question

Why should a production app have a health endpoint?

### Expected answer

Health checks allow the platform or orchestrator to determine whether the application is actually ready and healthy, not merely running.

---

## Part 6: Troubleshooting Read-Only Filesystem Issues

If the app fails, inspect:

```bash
docker logs mern-api-secure
docker exec -it mern-api-secure sh
```

Then ask:

- What does the application try to write at runtime?
- Is the write path a temporary directory, a cache folder, or a log path?
- Can that path be mounted read-write intentionally instead of disabling the security measure completely?

### Expected answer

The app may need explicit writable locations. Security settings should be precise rather than all-or-nothing.

---

## Part 7: Secure Network Design

Your Compose stack is:

```yaml
services:
  api:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      MONGO_URI: mongodb://mongodb:27017/mern

  mongodb:
    image: mongo
    ports:
      - "27017:27017"
```

### Question

Why is publishing MongoDB to the host insecure or unnecessary in many cases?

### Expected answer

The API can reach MongoDB through the private Docker network using the `mongodb` service name. The database is usually not meant to be publicly exposed.

---

## Part 8: Secrets Review

### Question

Which of these is a bad practice?

```yaml
environment:
  JWT_SECRET: super-secret
```

### Expected answer

It is bad practice to embed production secrets directly in Compose or Dockerfiles. Use a secret-management system or secure runtime injection mechanism instead.

---

## Part 9: Production Hardening Checklist

Check off each item that applies:

- [ ] Container runs as a non-root user
- [ ] `.dockerignore` excludes unnecessary files
- [ ] Image is minimized and current
- [ ] No secrets are baked into the image
- [ ] Unnecessary Linux capabilities are dropped
- [ ] Root filesystem is read-only where possible
- [ ] Only required writable paths are mounted
- [ ] Resource limits are configured
- [ ] MongoDB is not unnecessarily exposed
- [ ] Health checks are defined
- [ ] Logs and runtime behavior are monitored

---

## Part 10: Final Reflection

Write a short paragraph explaining how Docker security follows the principle of least privilege and defense in depth.

Your answer should mention at least these ideas:

- non-root execution
- minimal image surface
- reduced capabilities
- read-only filesystem where appropriate
- restricted network exposure
- secret management
- resource boundaries
- health checks
