# DevOps Mentorship Program — Day 21

## Phase 2: Docker & Containers

### Topic: Docker Fundamentals — Images, Containers & the MERN Runtime


## Progress Tracker

### Linux Phase Complete
- Days 1–19 — Linux Foundation
- Day 20 — Linux Phase Revision & Assessment

### Phase 2 Begins
- Day 21 — Docker Fundamentals

**Docker Phase Status:** 1 / ~10 topics complete

---

## 1. Quick Review from the Linux Phase

### Q1. A Node.js process is using 100% CPU. What should you investigate before killing it?
Before killing a process, check the root cause.

Use:
```bash
top
ps aux --sort=-%cpu | head
journalctl -u myapp -n 100
```

Ask:
- Is this a real traffic spike?
- Is there a code loop or infinite retry?
- Is the app waiting on a dependency?
- Is the system overloaded globally?

Killing a process without understanding why it is busy can remove useful evidence and worsen the incident.

### Q2. What is the difference between `df -h` and `du -sh`?
- `df -h` → filesystem capacity and available disk space
- `du -sh` → disk usage for a specific file or directory

Use:
- `df` to identify whether the disk itself is full
- `du` to find which folder is consuming the space

### Q3. Nginx returns 502. What test bypasses Nginx and checks Node directly?
```bash
curl http://localhost:3000/health
```

This is important because a 502 often means Nginx cannot reach the upstream app. Bypassing Nginx lets you verify whether the backend is alive and healthy.

### Q4. Why shouldn't MongoDB normally be publicly exposed?
Because it expands the attack surface and exposes a database to unauthorized access.

Production best practice:
- Restrict database ports to trusted internal networks
- Use VPC/private networking or firewall rules
- Do not expose a database directly to the public internet unless absolutely required

### Q5. What does least privilege mean?
A user, service, or process should only have the minimum permissions needed to do its job.

Example:
- A Node.js app should not run as root
- A database user should not have admin rights unless required
- A deployment script should not have unrestricted system access

---

## 2. Topic Name: Docker Fundamentals

Up to this point, your mental model has mainly been:

```text
Linux Server
    |
    +-- Node.js
    +-- Nginx
    +-- MongoDB
```

You install software directly on the server, configure it, and hope the environment remains consistent.

This creates several operational problems:
- “It works on my machine”
- Different Node versions across environments
- Missing OS packages or library dependencies
- Manual installation drift
- Configuration changes that are difficult to reproduce
- Harder rollbacks and deployments

Docker changes the packaging model.

Instead of manually saying:
> Install Node.js, copy these files, install dependencies, configure env vars, start the process

You package the application into a container image.

That image becomes a reusable, portable artifact that runs consistently across different environments.

---

## 3. Learning Objectives

By the end of this lesson, you should understand:
- What containers are
- What Docker images are
- What a container is
- What a Dockerfile does
- How ports work in Docker
- How containers differ from virtual machines
- How Docker fits into MERN production deployment
- The difference between an image and a running container
- Common Docker interview questions and answers

---

## 4. Container Mental Model

Think of Docker like this:

```text
Dockerfile
    |
    v
Docker Image
    |
    v
Docker Container
    |
    v
Running Application
```

For a Node.js application:

```text
Dockerfile
    |
    v
node-api:1.0
    |
    v
Container
    |
    v
Node.js API
```

The image is the packaged artifact.
The container is a running instance of that image.

### Important distinction
- Image = blueprint / package
- Container = running instance of that blueprint

You can run many containers from the same image.

---

## 5. Container vs Virtual Machine

A virtual machine generally includes:

```text
Hardware
    |
    v
Hypervisor
    |
    v
Guest OS
    |
    v
Libraries
    |
    v
Application
```

A container generally shares the host kernel:

```text
Hardware
    |
    v
Linux Host
    |
    v
Container Runtime
    |
    +---------------------+
    |                     |
    v                     v
Container A           Container B
App                   App
```

### Why this matters
Containers are typically:
- lighter than VMs
- faster to start
- easier to package and move
- more efficient for microservices and app deployment

### Interview answer
> A VM virtualizes an entire machine environment including a guest OS, while a container isolates application processes and runtime dependencies while sharing the host kernel.

---

## 6. Docker Image

An image contains the application environment required to run a container.

For example:
```text
node-api:1.0
```

This image may include:
- Node.js runtime
- Application code
- `package.json`
- Installed dependencies
- Startup command

Images are usually treated as immutable build artifacts.

If a new version is needed:
```text
node-api:1.0
    |
    v
node-api:1.1
```

You do not usually edit a running production image directly.
Instead, you build a new image version and deploy it.

This is one of the key ideas behind modern DevOps and CI/CD.

---

## 7. Docker Container

A container is a running instance of an image.

You might have:
```text
Image: node-api:1.0
    |
    +-----> Container 1
    +-----> Container 2
    +-----> Container 3
```

All three containers can come from the same image.

This is central to scaling and deployment.

Example use case:
- One app image
- Run 3 containers behind a load balancer
- Same code, same runtime, different running instances

---

## 8. Essential Docker Commands

### Check Docker installation
```bash
docker --version
```

### Show running containers
```bash
docker ps
```

### Show all containers, including stopped ones
```bash
docker ps -a
```

### List local images
```bash
docker images
```

### Pull an image from Docker Hub
```bash
docker pull nginx
```

### Run a container
```bash
docker run nginx
```

### Stop a container
```bash
docker stop <container>
```

### Remove a container
```bash
docker rm <container>
```

### Remove an image
```bash
docker rmi <image>
```

### What these mean in real life
- `docker pull` downloads the image
- `docker run` creates and starts a container from an image
- `docker ps` lists the running containers
- `docker logs` shows runtime output
- `docker rm` removes a container
- `docker rmi` removes an image

---

## 9. Docker Ports

Suppose Node.js inside a container listens on:
```text
3000
```

You can publish that port to the host:
```bash
docker run -p 3000:3000 node-api
```

This reads like:
```text
HOST : CONTAINER
3000 : 3000
```

### Architecture example
```text
Browser
    |
    v
Host :3000
    |
    v
Container :3000
    |
    v
Node.js
```

This is how you access the app from outside the container.

---

## 10. Important Distinction: Host Port vs Container Port

This command:
```bash
docker run -p 8080:3000 node-api
```

means:
- Host port = 8080
- Container port = 3000

So the app is reachable from:
```bash
http://localhost:8080
```

and internally the container still listens on 3000.

### Why this matters
This is a very common Docker interview question.

A candidate who understands this distinction is much more likely to handle Docker networking correctly.

---

## 11. Create Your First Dockerfile

Suppose your backend app contains:
```text
backend/
    package.json
    package-lock.json
    server.js
```

Create a file named:
```Dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

This is a very standard Node.js container setup.

---

## 12. Understanding Each Dockerfile Instruction

### `FROM`
```dockerfile
FROM node:22-alpine
```

This selects the base image.

You are saying:
> Start with a Node.js environment.

`alpine` is a lightweight Linux variant that keeps the image smaller.

### Why base images matter
The base image determines:
- OS family
- available packages
- package manager availability
- runtime environment
- image size and security posture

---

### `WORKDIR`
```dockerfile
WORKDIR /app
```

This sets the working directory inside the container.

After this, commands run from `/app` unless otherwise specified.

Example:
- your code goes under `/app`
- install dependencies there
- run the app from there

---

### `COPY`
```dockerfile
COPY package*.json ./
```

This copies dependency manifest files first.

This is important because it improves Docker build caching.

When you change application code later, Docker can reuse the cached dependency installation.

Then:
```dockerfile
COPY . .
```

This copies the full application source into the container.

---

### `RUN`
```dockerfile
RUN npm ci
```

This executes during the image build process.

This is different from runtime behavior.

Build step:
```bash
docker build
```

Runtime step:
```bash
docker run
```

`RUN` builds the environment. `CMD` defines what the container executes at runtime.

---

### `EXPOSE`
```dockerfile
EXPOSE 3000
```

This documents the application port.

However, it does not automatically publish the port to the host.

You still need runtime flags like:
```bash
docker run -p 3000:3000 node-api
```

This distinction is often tested in interviews.

---

### `CMD`
```dockerfile
CMD ["node", "server.js"]
```

This defines the default command that runs when the container starts.

So when you run:
```bash
docker run node-api
```

it runs:
```bash
node server.js
```

This is the default startup command for the container.

---

## 13. Understand the Build vs Run Model

This is one of the most important ideas in Docker.

### Build phase
```bash
docker build -t mern-api:1.0 .
```

During build:
- base image is pulled
- dependencies are installed
- app files are copied
- final image is assembled

### Runtime phase
```bash
docker run -d --name mern-api -p 3000:3000 mern-api:1.0
```

During runtime:
- container starts from the built image
- app process launches
- network port mapping is applied
- app receives traffic

---

## 14. MERN Production Architecture with Docker

Eventually, your application can be designed like this:

```text
                 Internet
                    |
                    v
                 Nginx
                    |
          +---------+---------+
          |                   |
          v                   v
   React Frontend      Node API
                          |
                          v
                       MongoDB
```

In a Dockerized environment, this could become:

```text
Docker Host
    |
    +-- nginx container
    +-- frontend container
    +-- api container
    +-- mongodb container
```

This is the foundation for multi-container deployments.

Later, Docker Compose will let you define this in a single configuration file.

---

## 15. Hands-On Lab: Build a Simple Node API Container

Create a simple app directory:
```bash
mkdir -p ~/devops-course/day21/node-api
cd ~/devops-course/day21/node-api
```

Create a file called `server.js`:
```javascript
const http = require("http");

const server = http.createServer((req, res) => {
  if (req.url === "/health") {
    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(JSON.stringify({ status: "ok" }));
    return;
  }

  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("MERN API running in Docker\n");
});

server.listen(3000, "0.0.0.0", () => {
  console.log("API listening on port 3000");
});
```

### Why `0.0.0.0` matters
Inside a container, the app must listen on a network interface that accepts traffic from the container network.

If you bind to `127.0.0.1`, the container often will not accept connections from outside itself.

This is a common Docker networking issue.

---

### Create a Dockerfile
```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY server.js .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Build the image
```bash
docker build -t mern-api:1.0 .
```

### Check available local images
```bash
docker images
```

### Run the container
```bash
docker run -d --name mern-api -p 3000:3000 mern-api:1.0
```

### Check running containers
```bash
docker ps
```

### Test the health endpoint
```bash
curl http://localhost:3000/health
```

Expected output:
```json
{"status":"ok"}
```

This validates that:
- the container is running
- the app is listening on port 3000
- host port mapping works

---

## 16. Inspect the Container

### View logs
```bash
docker logs mern-api
```

### Follow logs continuously
```bash
docker logs -f mern-api
```

### Inspect metadata
```bash
docker inspect mern-api
```

### Open an interactive shell inside the running container
```bash
docker exec -it mern-api sh
```

Then check:
```bash
ls
ps
```

Exit with:
```bash
exit
```

This connects Docker to your earlier Linux knowledge.

You are still working with:
- processes
- logs
- filesystem
- ports
- networking
- permissions

Docker just adds a containerized execution environment.

---

## 17. How Docker Builds on Linux Knowledge

Docker does not replace Linux.

It adds another layer of abstraction on top of the Linux runtime model.

You still use Linux concepts inside and around containers:
- `ps` to inspect processes
- `ss` to inspect sockets
- `journalctl` or container logs to inspect service logs
- `df` and `du` to monitor storage
- `top` and `free` for memory and CPU
- `chmod` and file ownership for container file security

### Example
If a container fails to start:
```bash
docker logs <container>
```

But to understand the environment, you may still inspect:
```bash
docker exec -it <container> sh
```

From there, you can run Linux commands inside the container environment.

---

## 18. MERN Production Example

Imagine you release a new API version:
```text
mern-api:1.0
```

Then update the code and build:
```text
mern-api:1.1
```

If version 1.1 fails in production:
```text
mern-api:1.1
    |
    v
Problem detected
    |
    v
Rollback to mern-api:1.0
```

This is one reason immutable artifacts are valuable.

A container image acts like a versioned deployment unit.

This later enables:
- CI/CD pipeline integration
- easier rollback
- consistent environments
- reproducible deployment

---

## 19. Interview Preparation

### Beginner Questions

#### Q1. What is Docker?
Docker is a platform for packaging and running applications in isolated containers using images and a container runtime.

#### Q2. What is a Docker image?
A packaged, reusable artifact from which containers are created.

#### Q3. What is a container?
A running instance of an image with isolated process, filesystem, and networking characteristics.

---

### Intermediate Questions

#### Q4. What is the difference between an image and a container?
An image is the packaged artifact; a container is a running instance of that image.

#### Q5. What does this command mean?
```bash
docker run -p 8080:3000 app
```
It maps port 8080 on the host to port 3000 inside the container.

#### Q6. Does `EXPOSE 3000` publish the port automatically?
No. `EXPOSE` documents the port. Publishing requires the runtime flag like `-p`.

---

### Advanced Question

#### Why would you use Docker for a MERN application?
Because Docker packages the application and its dependency runtime into a consistent artifact.

This helps with:
- reproducibility
- environment consistency
- faster deployment
- easier rollback
- CI/CD integration
- improved scaling

A strong answer should also mention:
- reduces “works on my machine” issues
- isolates applications from the host environment
- standardizes a deployment unit

---

## 20. Quiz

1. What creates a Docker image?
- A. `docker start`
- B. `docker build`
- C. `docker exec`
- D. `docker logs`

2. What creates a running container from an image?
- A. `docker run`
- B. `docker image`
- C. `docker build`
- D. `docker pull`

3. What does this mean?
```bash
-p 8080:3000
```
- A. Container 8080 -> host 3000
- B. Host 8080 -> container 3000
- C. Two containers communicate
- D. Docker exposes both ports automatically

4. What does `FROM` specify?
- A. Runtime command
- B. Base image
- C. Container port
- D. Volume

5. What does `WORKDIR /app` do?
- A. Creates a host directory
- B. Sets the working directory inside the image/container
- C. Publishes port 80
- D. Starts Node.js

6. What does `RUN npm ci` execute?
- A. At image build time
- B. Only when Docker stops
- C. On the host after deployment
- D. During DNS resolution

7. What does `CMD` specify?
- A. Default runtime command
- B. Base OS
- C. Build cache
- D. Firewall rule

8. Does `EXPOSE 3000` automatically publish port 3000?
- A. Yes
- B. No

9. Which command shows container logs?
- A. `docker journal`
- B. `docker logs`
- C. `docker output`
- D. `docker trace`

10. Which is lighter in typical deployments?
- A. A full VM with its own guest OS
- B. A container sharing the host kernel

### Answer Key
```text
B A B B A A B B B B
```

---

## 21. Practical Assessment

### Scenario
You have a Node.js API that works locally:
```bash
npm start
```

But it fails when placed inside Docker.

Users observe:
- connection refused
- app not answering on expected port

### How to diagnose it

#### Step 1: Check running containers
```bash
docker ps
```

If the container is not running, investigate the startup logs.

#### Step 2: Check container logs
```bash
docker logs <container>
```

Look for:
- port binding errors
- missing module errors
- runtime exceptions
- startup failure messages

#### Step 3: Open a shell inside the container
```bash
docker exec -it <container> sh
```

Then inspect:
```bash
ps
```

Check whether the Node.js process is actually running.

#### Step 4: Verify port binding
```bash
docker port <container>
```

Ensure the application binds to the correct network interface.

#### Step 5: Test health endpoint locally
```bash
curl http://localhost:3000/health
```

---

### Key interview follow-up
If the app is listening on:
```text
127.0.0.1:3000
```

inside the container, this can prevent traffic from reaching it through the container network interface.

For a simple app, binding to:
```text
0.0.0.0
```

is usually the correct choice.

---

## 22. Month 2 Cumulative Project

### Project Theme
Containerized Production MERN Application

This extends the Month 1 Linux server project.

### Target Architecture
```text
                 Internet
                    |
                    v
                 Nginx
                    |
          +---------+---------+
          |                   |
          v                   v
   React Frontend      Node API
                          |
                          v
                       MongoDB
```

### Month 2 Requirements

#### Backend
- Create a Dockerfile
- Build an image
- Run the container
- Expose the correct port
- Add a health endpoint

#### Frontend
- Build a production version
- Package it in a suitable container setup

#### Database
- Understand how MongoDB should be handled as a stateful service
- Consider persistent storage and backups

#### Operations
Demonstrate:
- `docker build`
- `docker run`
- `docker ps`
- `docker logs`
- `docker exec`
- `docker inspect`

#### Security
Do not put secrets directly into Dockerfiles.

Avoid baking in:
- `MONGO_URI`
- `JWT_SECRET`
- API keys

These will be handled later using safer configuration patterns.

---

## 23. Final Interview Challenge

Answer aloud:
> Explain how you would containerize a Node.js backend for a production MERN application.

### Strong answer structure
1. Create a Dockerfile
2. Select an appropriate Node base image
3. Set the working directory
4. Copy dependency manifests
5. Install dependencies
6. Copy source code
7. Define application port
8. Define startup command
9. Build an immutable image
10. Run the container
11. Publish required port
12. Test the health endpoint
13. Inspect logs and runtime metadata

Then explain:
- the container should not contain production secrets directly
- configuration should be externalized and securely injected later

---

## 24. Key Takeaways

You learned:
- What Docker is
- Images vs containers
- Dockerfiles
- `FROM`
- `WORKDIR`
- `COPY`
- `RUN`
- `EXPOSE`
- `CMD`
- Port publishing
- Container logs and inspection
- Why Docker is valuable for MERN deployments
- How Docker builds on your Linux knowledge

### Core mental model
```text
Dockerfile
    |
    v
Docker Image
    |
    v
Container
    |
    v
Application
```

The broader DevOps progression now looks like:
```text
Linux
    |
    v
Docker
    |
    v
CI/CD
    |
    v
Cloud
    |
    v
Infrastructure as Code
    |
    v
Kubernetes
```

---

## 25. Next Lesson Preview — Day 22

### Topic: Docker Images, Layers & Build Optimization

Next, we will go deeper into:
- image layers
- Docker build cache
- `.dockerignore`
- image size reduction
- dependency caching
- production Dockerfiles
- Node.js/MERN image optimization
- Docker security basics
- interview questions about image size and build speed

This will evolve the Month 2 project. 
