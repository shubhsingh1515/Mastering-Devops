# DevOps Mentorship Program - Day 26

## 🐳 Phase 2: Docker & Containers

### 🟢 Topic: Docker Security & Production Hardening

**Docker phase:** 5 / approximately 10 topics

---

## 📊 Progress Tracker

✅ Linux Phase Complete  
Days 1-19 — Linux Foundation  
Day 20 — Linux Phase Revision & Assessment  

🐳 Docker Phase  
Day 21 — Docker Fundamentals  
Day 22 — Images, Layers & Build Optimization  
Day 23 — Docker Networking & Persistent Storage  
Day 24 — Weekly Revision & Assessment  
Day 25 — Docker Compose  
Day 26 — Docker Security & Production Hardening  

**Docker Phase:** 5 / ~10 topics

---

## 🔁 1. Review From Earlier Lessons

### Q1. Why shouldn't Node use `localhost` to connect to MongoDB in separate containers?

Because `localhost` inside the Node container refers to the Node container itself. On a shared Docker network, the Node service should resolve MongoDB by its service name:

```text
mongodb://mongodb:27017/mern
```

### Q2. What does a Docker volume provide?

A volume provides persistent storage that can survive the lifecycle of an individual container.

### Q3. Does `depends_on` guarantee MongoDB is ready?

No. `depends_on` only expresses dependency and startup ordering. It does not guarantee that the database is ready to accept connections. Health checks and application-level retry logic are also required.

### Q4. Why copy `package*.json` before the application source in a Node Dockerfile?

This improves Docker build cache reuse for dependency installation layers.

### Q5. What does `-p 8080:3000` mean?

```text
Host port 8080
      ↓
Container port 3000
```

---

## 1. 📚 Why Container Security Matters

A container is isolated, but isolation does not mean immunity.

A compromised app can be much more dangerous if the container has:

- root privileges
- unnecessary Linux capabilities
- access to the host filesystem
- sensitive secrets embedded in the image
- excessive CPU or memory consumption

Production goal:

```text
Minimal image
     ↓
Minimal privileges
     ↓
Minimal capabilities
     ↓
Minimal filesystem access
     ↓
Minimal network exposure
     ↓
Minimal secrets exposure
```

This is the Docker equivalent of the Linux least-privilege principle.

---

## 2. 🔐 Run Containers as Non-Root

A weak Dockerfile might look like this:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY . .

RUN npm ci

CMD ["node", "server.js"]
```

Depending on the image, the process may run with elevated privileges.

A better approach is:

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

Now the application runs as the `node` user rather than as `root`:

```text
Container
   ↓
node user
   ↓
Node.js
```

This reduces the impact of a compromise.

### Interview wording

> Running as non-root reduces the impact of a container compromise; it does not make the application secure by itself.

---

## 3. 🧠 Why Non-Root Helps

Imagine an application vulnerability allows an attacker to execute:

```bash
touch /some/file
```

If the application runs as root, the impact is much greater. If the process runs as a restricted user, its permissions are limited.

The principle is simple:

```text
Attacker
   ↓
Compromised Node process
   ↓
Limited permissions
```

This is a great point to mention in interviews: restricted identity reduces blast radius.

---

## 4. 🔑 Secrets

Never do this:

```dockerfile
ENV JWT_SECRET=my-secret
```

And never copy a `.env` file into the image:

```dockerfile
COPY .env .
```

This embeds secrets inside a reusable image, which is a major security risk.

A better mental model is:

```text
Docker image
   ↓
Reusable, environment-independent artifact
```

while:

```text
Runtime configuration
   ↓
Environment-specific settings
   ↓
Secrets
```

is supplied separately at runtime.

---

## 5. 🧪 Example

Instead of baking values like:

- `MONGO_URI`
- `JWT_SECRET`
- `API_KEY`

into the image, the app should read them from runtime configuration.

For local Compose-based development, you might use:

```yaml
environment:
  NODE_ENV: development
  MONGO_URI: mongodb://mongodb:27017/mern
```

For production, use the deployment platform's secret-management workflow, such as:

- Docker secrets
- Kubernetes secrets
- cloud secret stores
- environment injection managed by the platform

### Principle

> Build once; inject environment-specific secrets at runtime.

---

## 6. 🛡️ Linux Capabilities

Linux traditionally associated many powerful operations with root. Modern Linux separates privileges into capabilities.

A container does not need every capability.

You can inspect a running container with:

```bash
docker inspect <container>
```

Then look at its security metadata.

A hardening strategy can involve dropping unnecessary capabilities.

Example:

```bash
docker run \
  --cap-drop=ALL \
  ...
```

This removes most capabilities from the container.

However, do not apply this blindly. Some software genuinely requires specific capabilities.

### Production approach

```text
Determine requirements
     ↓
Drop unnecessary capabilities
     ↓
Test
     ↓
Add only required capability
```

---

## 7. 📂 Read-Only Filesystem

Some applications do not need to modify their root filesystem during normal operation.

You can use:

```bash
docker run --read-only myapp
```

This makes the container root filesystem read-only.

This can reduce the attacker's ability to modify files after a compromise.

However, applications may still need writable locations for:

- temporary files
- runtime caches
- logs
- uploads

In those cases, explicitly provide writable volumes or temporary mount points.

---

## 8. 🧠 Security Is About Explicit Access

Instead of:

```text
Container
   ↓
Write anywhere
```

prefer:

```text
Container
   │
   ├── Read application files
   │
   └── Write only where necessary
```

This follows the same reasoning behind Linux permissions:

- Who?
- What access?
- Where?
- Why?

The principle is explicit and minimal access.

---

## 9. 🌐 Network Hardening

The previous working architecture looked like:

```text
Internet
   ↓
Nginx
   ↓
Node
   ↓
MongoDB
```

That should not become:

```text
Internet
   ├── Nginx
   ├── Node :3000
   └── MongoDB :27017
```

Instead, prefer:

```text
Internet
   │
   ▼
Nginx
   │
   ▼
Private Docker network
   │
   ├── Node
   │
   └── MongoDB
```

Only expose what external clients actually require.

---

## 10. 🚫 Don't Publish Every Port

Avoid a configuration like:

```yaml
ports:
  - "27017:27017"
```

unless there is a justified reason.

The API can talk to MongoDB internally with:

```text
mongodb:27017
```

without making the database externally reachable from the host.

Similarly, if Nginx is the public reverse proxy, the Node application may not need a public host-port mapping at all.

Example:

```text
Internet
   ↓
Nginx :443
   ↓
Docker network
   ↓
Node :3000
```

---

## 11. 📏 Resource Limits

Containers can consume resources. A faulty Node process can keep growing and eventually affect the host:

```text
Node
  ↓
Memory growth
  ↓
Host pressure
  ↓
Other services affected
```

Resource limits help establish boundaries.

Example:

```bash
docker run \
  --memory=512m \
  --cpus=1 \
  myapp
```

The exact limit should be based on real workload observation rather than arbitrary guesses.

---

## 12. 🧠 Why Resource Limits Are a Security Concern

Security is not only about unauthorized access.

Consider:

```text
One compromised container
        ↓
Consumes all memory
        ↓
Host becomes unstable
        ↓
Other services fail
```

This is both a performance and reliability issue. Good resource boundaries reduce blast radius and protect the entire host.

---

## 13. 🩺 Health Checks

A production container should have a way to determine whether the application is actually healthy.

For a Node API, a health endpoint might be:

```text
GET /health
```

which returns:

```json
{
  "status": "ok"
}
```

A health check can test that endpoint.

Important concept:

```text
Container running
   ≠
Application healthy
```

This is the same distinction you learned earlier when thinking about Linux services and process health.

---

## 14. 🔍 Image Security

Before deploying an image such as `node-api:1.0`, ask:

- Is the base image trusted?
- Is it up to date?
- Are there known vulnerabilities?
- Does it contain unnecessary packages?
- Are development dependencies included?
- Are secrets embedded in the image?
- Is the image being scanned before deployment?

Typical production pipeline:

```text
Dockerfile
   ↓
Build
   ↓
Image scan
   ↓
Tests
   ↓
Registry
   ↓
Deployment
```

Image scanning tools vary by environment, but the principle matters more than any single tool name.

---

## 15. 🧹 Minimize the Image

Earlier, you learned image optimization. Security and optimization overlap.

Reduce:

- unnecessary packages
- unnecessary dependencies
- build tools
- test artifacts
- source files not needed at runtime
- secrets

A smaller attack surface generally means:

```text
Fewer components
   ↓
Fewer potential vulnerabilities
```

But do not minimize the image at the cost of compatibility or missed patches. Security updates still matter.

---

## 16. 🏗️ Secure MERN Architecture

A production-oriented design is:

```text
                         Internet
                            │
                         443 HTTPS
                            │
                            ▼
                          Nginx
                            │
                  Private Docker Network
                     ┌───────┴───────┐
                     ▼               ▼
                 Frontend        Node API
                                     │
                                     ▼
                                  MongoDB
                                     │
                                     ▼
                               Persistent data
```

### Security controls

#### Nginx
- TLS termination
- Public entry point

#### Node API
- non-root user
- minimal image
- no baked secrets
- resource limits

#### MongoDB
- private network only
- persistent volume
- backup strategy

---

## 17. 🧪 Practical Lab

Take your backend Dockerfile and harden it.

### Start with this Dockerfile

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

Add a `.dockerignore` file:

```text
node_modules
.git
.env
npm-debug.log
coverage
```

Build it:

```bash
docker build -t mern-api:secure .
```

Run with added hardening controls:

```bash
docker run -d \
  --name mern-api-secure \
  --read-only \
  --cap-drop=ALL \
  --memory=512m \
  --cpus=1 \
  -p 3000:3000 \
  mern-api:secure
```

Then test:

```bash
curl http://localhost:3000/health
```

### Important note

Do not assume these restrictions work for every Node application without testing.

If the app needs writable temporary files or a specific capability, configure those explicitly and validate behavior.

---

## 18. 🔍 Inspect the Result

Check the container status:

```bash
docker ps
```

Inspect the running container:

```bash
docker inspect mern-api-secure
```

Check the runtime user:

```bash
docker exec mern-api-secure whoami
```

You want to see a non-root user.

Check logs:

```bash
docker logs mern-api-secure
```

If the application fails because the filesystem is read-only, that is valuable information. Ask:

> What does the application actually need to write?

Then provide only the minimal writable location instead of disabling security controls entirely.

---

## 19. 💼 Interview Preparation

### Beginner

#### Q1. Why run a container as a non-root user?

To reduce the privileges available to the application and limit the impact of a potential compromise.

#### Q2. Should secrets be included in a Docker image?

No. Secrets should be supplied through runtime configuration or a secret management mechanism.

#### Q3. What does `--read-only` do?

It makes the container's root filesystem read-only, reducing unauthorized modification of the filesystem.

---

### Intermediate

#### Q4. What are Linux capabilities?

They are fine-grained privileges that split traditionally powerful root operations into smaller permissions that can be granted or removed selectively.

#### Q5. Why would you use `--cap-drop=ALL`?

To remove unnecessary Linux capabilities and reduce the container's privilege surface, then add back only the capabilities the application genuinely requires.

#### Q6. Why are resource limits useful?

They prevent a single container from consuming excessive CPU or memory and reduce the operational blast radius.

---

### Advanced Interview Scenario

> Your security team says your Node.js container has too many privileges. How would you harden it?

Strong answer:

```text
Run as non-root
     ↓
Minimize base image
     ↓
Remove unnecessary packages
     ↓
Drop unnecessary capabilities
     ↓
Use a read-only filesystem where practical
     ↓
Mount only required writable paths
     ↓
Remove unnecessary network exposure
     ↓
Protect secrets
     ↓
Apply resource limits
     ↓
Scan the image
     ↓
Monitor runtime behavior
```

Do not say:

> "I'll just use Alpine."

That is only one optimization. Container security is a defense-in-depth problem.

---

## 20. 📝 Quiz

### 1. What is the main reason to run containers as non-root?

A. Faster startup  
B. Reduced privilege and smaller compromise impact  
C. Smaller images  
D. Automatic encryption

### 2. Should `JWT_SECRET` be baked into a Docker image?

A. Yes  
B. No

### 3. What does `--cap-drop=ALL` do?

A. Deletes all containers  
B. Removes Linux capabilities from the container  
C. Disables networking  
D. Deletes volumes

### 4. What does `--read-only` protect?

A. The root filesystem from writes  
B. The Docker registry  
C. DNS  
D. MongoDB backups

### 5. Why might a read-only filesystem break an application?

A. Some applications need to write temporary/runtime data  
B. Node cannot use TCP  
C. Docker requires write access everywhere  
D. Nginx cannot start

### 6. What is the relationship between image minimization and security?

A. Fewer unnecessary components can reduce attack surface  
B. Small images cannot be hacked  
C. Image size automatically encrypts data  
D. Image size replaces authentication

### 7. Why shouldn't MongoDB be unnecessarily published to the host?

A. It doesn't need to be reachable externally when Node can use the private Docker network  
B. MongoDB doesn't use TCP  
C. Docker blocks port 27017  
D. Nginx requires it

### 8. What does a container health check tell you?

A. Whether the application meets a defined health condition  
B. Whether the host is physically secure  
C. Whether the database has backups  
D. Whether the image has no vulnerabilities

### 9. Why use resource limits?

A. To prevent uncontrolled resource consumption from affecting the host and other services  
B. To encrypt containers  
C. To create Docker networks  
D. To replace monitoring

### 10. What is the best security philosophy for containers?

A. Give every container root and restrict nothing  
B. Defense in depth and least privilege  
C. Expose every port  
D. Put credentials in Dockerfiles

### Answer Key

1. B  
2. B  
3. B  
4. A  
5. A  
6. A  
7. A  
8. A  
9. A  
10. B

---

## 21. 🧪 Practical Assessment - Harden This Container

### Starting configuration

```yaml
services:
  api:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      JWT_SECRET: super-secret
      MONGO_URI: mongodb://mongodb:27017/mern

  mongodb:
    image: mongo
    ports:
      - "27017:27017"
```

### Identify the security problems.

#### Problem 1

```yaml
JWT_SECRET: super-secret
```

Do not put real secrets directly in the Compose configuration.

#### Problem 2

```yaml
ports:
  - "27017:27017"
```

MongoDB is unnecessarily exposed if the API is the only client that needs access.

#### Problem 3

The API should run as a non-root user where practical.

#### Problem 4

The image should be minimized and scanned before deployment.

#### Problem 5

Consider additional hardening such as:

- Linux capabilities
- read-only filesystem
- resource limits
- health checks

based on actual application requirements.

---

## 22. 🏗️ Month 2 Cumulative Project Update

Your containerized MERN application now needs a security layer.

### Required architecture

```text
                    Internet
                       │
                       ▼
                    Nginx
                       │
                Private network
                       │
               ┌───────┴───────┐
               ▼               ▼
          Frontend           Node API
                                │
                                ▼
                             MongoDB
                                │
                                ▼
                           Persistent data
```

### Security requirements

#### Node
- non-root
- minimal image
- no baked secrets
- health check
- resource limits

#### MongoDB
- private network
- no unnecessary host port
- persistent storage
- backup strategy

#### Overall
- minimal public exposure
- least privilege
- minimal image
- secret separation
- resource boundaries
- runtime monitoring

---

## 23. 🎤 Final Interview Challenge

Answer aloud:

> How would you harden a Dockerized Node.js application before putting it into production?

A strong answer should include:

1. Use a trusted, maintained base image
2. Minimize image contents
3. Use a `.dockerignore` file
4. Install production dependencies only
5. Run as non-root
6. Drop unnecessary capabilities
7. Use a read-only filesystem where practical
8. Limit writable paths
9. Never bake secrets into the image
10. Avoid unnecessary public ports
11. Apply CPU and memory limits
12. Add health checks
13. Scan images
14. Monitor logs and runtime behavior
15. Keep images and dependencies updated

Then explain:

> No single control makes a container secure. The objective is to reduce privileges, attack surface, exposure, and blast radius at every layer.

---

## 24. 📖 Today's Key Takeaways

You learned:

✅ Non-root containers  
✅ Docker secrets principles  
✅ Linux capabilities  
✅ `--cap-drop`  
✅ Read-only filesystems  
✅ Writable-path design  
✅ CPU/memory limits  
✅ Health checks  
✅ Image security  
✅ Minimal attack surface  
✅ MERN container hardening  
✅ Defense in depth

Your security model is now:

```text
              Container Security
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    Identity       Filesystem     Network
       │             │             │
   Non-root      Read-only      Minimal ports
       │          where possible      │
       ▼             ▼             ▼
  Capabilities   Writable paths   Private network
       │
       └──────────────┬──────────────┘
                      ▼
                  Resources
                      │
                 CPU / Memory
                      │
                      ▼
                   Secrets
                      │
                      ▼
                   Scanning
```

---

## ⏭️ Next Lesson

Day 27 — Docker Registry, Image Tagging & Deployment Workflow

Topics to cover:

- Docker registries
- Image tags
- `latest` vs immutable version tags
- Pushing and pulling images
- Private registries
- CI/CD image flow
- MERN deployment workflow
- Image promotion from development → staging → production
- Rollbacks
- Registry security
- Docker interview scenarios

The next Sunday will be the weekly revision, quiz, and practical assessment rather than a new topic.
