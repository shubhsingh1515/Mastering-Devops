# DevOps Mentorship Program - Day 27

## 🐳 Phase 2: Docker & Containers

### 🟢 Topic: Docker Registry, Image Tagging & Deployment Workflow

**Estimated time:** 25-30 minutes  
**Docker phase:** 6 / approximately 10 topics

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
Day 27 — Docker Registry, Image Tagging & Deployment Workflow  

**Docker Phase:** 6 / ~10 topics

---

## 🔁 1. Review From Earlier Lessons

### Q1. Why should a Node container not use `localhost` to connect to MongoDB in another container?

Because `localhost` refers to the Node container itself. On the shared Docker network, use the MongoDB service name:

```text
mongodb://mongodb:27017/mern
```

### Q2. Why should production containers normally run as non-root?

To apply least privilege and reduce the impact of an application compromise.

### Q3. What does this mapping mean?

```bash
-p 8080:3000
```

```text
Host port 8080
       ↓
Container port 3000
```

### Q4. Why is a Docker volume not a backup?

A volume provides persistence, but a backup is an independent recovery mechanism for events such as deletion, corruption, or host loss.

### Q5. Why should `package*.json` usually be copied before source code?

To improve Docker layer-cache reuse and avoid reinstalling dependencies when only application code changes.

---

## 1. 📚 The Problem: Where Does Your Image Go?

You have learned the basic flow:

```text
Dockerfile
    ↓
Docker image
    ↓
Container
```

That works on one computer, but a production team needs to move the image to another machine. Where does production get it?

The answer is a **container registry**.

The complete flow becomes:

```text
Source code
    ↓
Docker build
    ↓
Docker image
    ↓
Container registry
    ↓
Production server
    ↓
Container
```

A registry is a service that stores, distributes, and manages container images. Examples include Docker Hub, GitHub Container Registry, Azure Container Registry, and private registry products.

The registry separates image creation from image execution. CI can build and publish an image, while a deployment platform pulls that tested image later.

---

## 2. 🎯 Learning Objectives

By the end of this lesson, you should understand:

- What a container registry is
- Why image tags matter
- Why `latest` is often a poor production reference
- How to tag an image
- How to push and pull images
- What immutable deployment means
- How Docker fits into CI/CD
- How to promote an image from staging to production
- How to roll back a deployment
- How registry security protects the software supply chain
- How this applies to a MERN application

---

## 3. 📦 Image Naming

Suppose the application is called `mern-api`.

A simple local image name is:

```text
mern-api:1.0
```

A registry-qualified image name could be:

```text
registry.example.com/mern-api:1.0
```

Or it could include a team or namespace:

```text
registry.example.com/team/mern-api:1.0
```

The important parts are:

```text
registry.example.com/team/mern-api:1.0
│                  │             │
│                  │             └── tag
│                  └──────────────── repository path
└────────────────────────────────── registry
```

A registry-qualified name tells Docker where the image belongs and which repository and tag should be used.

---

## 4. 🏷️ Why Tags Matter

You could deploy:

```text
mern-api:latest
```

But what does `latest` mean at a specific point in time?

```text
Today:     latest → version A
Tomorrow:  latest → version B
```

The tag can move to different image content. A deployment referencing `latest` may therefore run different code at different times.

For production, prefer explicit versioning such as:

```text
mern-api:1.4.2
```

or a commit-based tag:

```text
mern-api:8f31c2a
```

A useful strategy is to publish both a human-readable release tag and a commit tag:

```text
mern-api:1.4.2
mern-api:8f31c2a
```

The release tag helps people understand the version. The commit tag helps trace the image back to source control.

Tags improve communication, but tags may still be moved or overwritten. For the strongest production guarantee, use an immutable image digest.

---

## 5. 🧠 Immutable Deployment

Imagine production runs:

```text
mern-api:1.4.2
```

You know which release is intended. A new release can then be deployed explicitly:

```text
1.4.2
  ↓
1.4.3
```

If errors increase after the release:

```text
1.4.3 ❌
  ↓
rollback
  ↓
1.4.2 ✅
```

This is easier to reason about than a mutable `latest` tag changing underneath a server.

An immutable deployment means that the deployment reference points to a fixed artifact. The tested artifact is not silently replaced by different content.

---

## 6. 🔐 Image Digests

Tags are human-friendly:

```text
mern-api:1.4.2
```

An image also has a content-addressed digest:

```text
sha256:...
```

The digest identifies the exact image content. Conceptually:

```text
Friendly tag
      ↓
Release version
      ↓
Exact image digest
```

A production deployment can use a digest reference such as:

```text
registry.example.com/team/mern-api@sha256:<digest>
```

A digest is stronger than a mutable tag because changing a tag does not change the content represented by an existing digest.

Use tags for usability and digests for controlled, exact deployments where the platform supports them.

---

## 7. 🏷️ Tag a Local Image

Suppose you built this local image:

```bash
docker build -t mern-api:1.0 .
```

Add a registry-qualified tag:

```bash
docker tag \
  mern-api:1.0 \
  registry.example.com/team/mern-api:1.0
```

Now `docker images` may show both names pointing to the same image content:

```text
mern-api:1.0
        │
        └────── registry.example.com/team/mern-api:1.0
```

Tagging does not rebuild the image. It adds another reference to the same local image content.

Check the references:

```bash
docker images
```

The repository and tag are used later by `docker push` to select the destination.

---

## 8. 📤 Push an Image to a Registry

Authenticate to the registry:

```bash
docker login registry.example.com
```

Push the image:

```bash
docker push registry.example.com/team/mern-api:1.0
```

The flow is:

```text
Developer or CI
      ↓
Docker image
      ↓
Registry repository
```

After the push completes, another machine with permission can pull the image.

Never place registry passwords or access tokens in a Dockerfile or source repository. Use a secure credential store or the CI platform's protected secret mechanism.

---

## 9. 📥 Pull and Run an Image

On a deployment server:

```bash
docker pull registry.example.com/team/mern-api:1.0
```

Then run it:

```bash
docker run \
  -d \
  --name mern-api \
  -p 3000:3000 \
  registry.example.com/team/mern-api:1.0
```

The build and run responsibilities are now separate:

```text
Build in CI or development
          ↓
Push artifact to registry
          ↓
Pull artifact on deployment server
          ↓
Run the exact artifact
```

This separation is a major DevOps principle. Production does not need the source code or a compiler just to run an already-built image.

---

## 10. 🏗️ CI/CD Deployment Flow

A typical pipeline looks like this:

```text
Developer
    │
    ▼
Git push
    │
    ▼
CI pipeline
    │
    ├── Run tests
    ├── Build Docker image
    ├── Scan image
    └── Push image
           │
           ▼
        Registry
           │
           ▼
        Staging
           │
       Validation
           │
           ▼
       Production
```

Production should not build a different image from the same source again. It should pull the artifact that CI built, scanned, and tested.

A pipeline commonly records:

- Git commit SHA
- image repository
- image tag
- image digest
- build timestamp
- test results
- deployment environment

This metadata creates traceability from production back to the exact source change.

---

## 11. 🧠 Build Once, Promote the Artifact

### Bad workflow

```text
Build for staging
    ↓
Deploy staging
    ↓
Build again for production
    ↓
Deploy production
```

The two builds might differ because of dependency resolution, build context, timestamps, base-image changes, or configuration.

### Better workflow

```text
Build once
    ↓
Test the image
    ↓
Scan the image
    ↓
Push to registry
    ↓
Deploy to staging
    ↓
Approve and validate
    ↓
Promote the same image
    ↓
Deploy to production
```

This is **artifact promotion**. The artifact tested in staging is the artifact deployed in production.

Environment-specific configuration, such as database URLs and secrets, should be injected at runtime. The application image should remain the same.

---

## 12. 🌍 MERN Example

Suppose the Node API commit is:

```text
8f31c2a
```

CI builds and pushes:

```text
registry.example.com/team/mern-api:8f31c2a
```

Staging runs that image:

```text
8f31c2a
```

Tests pass. Production then runs the same image:

```text
8f31c2a
```

The exact same artifact moves through the environments:

```text
Git commit 8f31c2a
      ↓
CI build
      ↓
Image tag 8f31c2a
      ↓
Staging
      ↓
Production
```

For a complete MERN system, you may publish separate images:

```text
registry.example.com/team/mern-frontend:1.4.2
registry.example.com/team/mern-api:1.4.2
```

MongoDB is commonly operated as a managed database or as a separately controlled stateful service rather than being treated like a disposable application image.

---

## 13. 🧪 Versioning Strategy

You can use semantic versions:

```text
1.0.0
1.0.1
1.1.0
2.0.0
```

Or commit-based tags:

```text
8f31c2a
9ab44ef
```

A practical production strategy can use both:

```text
mern-api:1.4.2
mern-api:8f31c2a
```

Record the digest used by the deployment as well.

The most important goal is traceability:

```text
Production deployment
        ↓
Image digest
        ↓
Image tag
        ↓
Git commit
        ↓
Source change
```

You should be able to answer:

> Exactly which source code is running in production?

Avoid relying on a tag that can be overwritten without an audit trail.

---

## 14. 🔄 Rollback

Suppose production currently runs:

```text
1.4.2
```

You deploy:

```text
1.4.3
```

Then API errors increase. Roll back by deploying the previous known-good image:

```text
1.4.3 ❌
   ↓
1.4.2 ✅
```

Example:

```bash
docker pull registry.example.com/team/mern-api:1.4.2
docker stop mern-api
docker rm mern-api
docker run -d \
  --name mern-api \
  -p 3000:3000 \
  registry.example.com/team/mern-api:1.4.2
```

A real orchestrator may provide safer rolling or declarative rollback commands, but the principle is the same: select the previous known-good artifact.

Do not normally rebuild old source just to roll back. The old image should already be preserved in the registry according to a retention policy.

Rollback must also consider database schema compatibility. An application image can be rolled back quickly, but an irreversible database migration may require a separate recovery plan.

---

## 15. 🚨 Why `latest` Can Cause Problems

Imagine two servers both configured with:

```text
mern-api:latest
```

A new image is pushed:

```text
latest → version 2
```

Depending on when each server pulled the tag, the servers may now be running different actual versions.

Prefer:

```text
Server A → mern-api:1.4.2
Server B → mern-api:1.4.2
```

Or use an exact digest. The desired state then identifies a specific artifact instead of a moving label.

`latest` can still be convenient for experiments, but it is a weak production deployment reference because it reduces determinism, auditability, and rollback clarity.

---

## 16. 🔐 Registry Security

A registry is part of the software supply chain. Protect it like production infrastructure.

Consider:

- authentication
- authorization
- image scanning
- repository access control
- retention policies
- audit logs
- TLS
- secret protection
- signed or verified artifacts where supported
- protection against accidental tag overwrites

Apply least privilege to CI:

```text
CI
 ↓
Can push images
```

CI should not automatically have unrestricted production administrator access. Separate permissions for building, publishing, approving, and deploying when practical.

Use private repositories for proprietary images and remove credentials from logs. Rotate credentials and prefer short-lived tokens or workload identity when the platform supports them.

---

## 17. 🧠 Supply Chain Mental Model

The deployment chain is:

```text
Developer
    ↓
Source repository
    ↓
CI
    ↓
Dependencies
    ↓
Docker build
    ↓
Image
    ↓
Security scan
    ↓
Registry
    ↓
Deployment
```

Every stage is part of the software supply chain.

Security is not only:

> Is the production server secure?

It is also:

> Can I trust the artifact being deployed?

This is why image provenance, vulnerability scanning, controlled registry access, immutable references, and deployment traceability matter.

---

## 18. 🧪 Hands-On Lab

Build your API image:

```bash
docker build -t mern-api:1.0 .
```

Tag it for a registry:

```bash
docker tag \
  mern-api:1.0 \
  registry.example.com/team/mern-api:1.0
```

Inspect the image:

```bash
docker images
```

If you have access to a real registry, authenticate:

```bash
docker login registry.example.com
```

Push it:

```bash
docker push registry.example.com/team/mern-api:1.0
```

On another machine or deployment environment, pull it:

```bash
docker pull registry.example.com/team/mern-api:1.0
```

Run and test it:

```bash
docker run \
  -d \
  --name mern-api \
  -p 3000:3000 \
  registry.example.com/team/mern-api:1.0

curl http://localhost:3000/health
```

If you do not have access to a registry, practice the build, tag, image inspection, and versioning steps locally.

Do not use fake registry commands with real credentials. Replace `registry.example.com/team` with a registry and repository that you are authorized to use.

---

## 19. 💼 Interview Preparation

### Beginner

#### Q1. What is a Docker registry?

A service that stores and distributes container images.

#### Q2. What is a Docker tag?

A human-readable reference associated with image content, commonly used to identify a version or variant.

#### Q3. Why shouldn't production blindly use `latest`?

Because the tag can move to different image content, making deployments less deterministic and harder to audit or roll back.

---

### Intermediate

#### Q4. Explain build once, deploy many.

Build the application artifact once, test that artifact, store it in a registry, and promote the same artifact through staging and production rather than rebuilding for each environment.

#### Q5. How would you roll back a Docker deployment?

Deploy the previous known-good image version or digest from the registry instead of rebuilding old source code.

#### Q6. Why are image digests useful?

They identify exact image content and provide a stronger immutable reference than a mutable tag.

---

### Advanced Interview Scenario

> Staging passed testing, but production is running a slightly different Docker image. How could that happen?

Possible causes:

```text
Rebuilt image for production
       ↓
Different dependency resolution
       ↓
Different build context
       ↓
Different base image
       ↓
Mutable tag such as latest
```

Better architecture:

```text
CI
 ↓
Build exact image
 ↓
Test exact image
 ↓
Push exact image
 ↓
Staging
 ↓
Promote same image
 ↓
Production
```

This removes a major class of "worked in staging" inconsistencies.

---

## 20. 📝 Quiz

### 1. What is the purpose of a Docker registry?

A. Run Linux processes  
B. Store and distribute container images  
C. Replace Dockerfiles  
D. Manage CPU

### 2. Which is generally more deterministic for production?

A. `latest`  
B. A specific version tag or immutable digest  
C. Random container ID  
D. Local source directory

### 3. What does `docker push` do?

A. Uploads an image to a registry  
B. Starts a container  
C. Builds an image  
D. Creates a volume

### 4. What does `docker pull` do?

A. Deletes an image  
B. Downloads an image from a registry  
C. Builds a Dockerfile  
D. Creates a network

### 5. Why is build-once/promote useful?

A. It ensures environments use the same tested artifact  
B. It removes testing  
C. It avoids registries  
D. It requires rebuilding production

### 6. What is an image digest?

A. A content-addressed identifier for image content  
B. A container password  
C. A port number  
D. A Docker network

### 7. What is a common rollback strategy?

A. Rebuild random source code  
B. Redeploy the previous known-good image  
C. Delete the registry  
D. Change MongoDB's port

### 8. Why should registry credentials follow least privilege?

A. To limit damage if credentials are compromised  
B. To make images smaller  
C. To speed up Docker  
D. To improve DNS

### 9. Which flow is preferable?

A. Build separately for staging and production  
B. Build once, test, then promote the same artifact  
C. Build directly on production  
D. Build manually on every server

### 10. Which gives the strongest exact-content reference?

A. Mutable `latest` tag  
B. Image digest  
C. Container name  
D. Hostname

### Answer Key

1. B  
2. B  
3. A  
4. B  
5. A  
6. A  
7. B  
8. A  
9. B  
10. B

---

## 21. 🧪 Practical Assessment - Production Deployment Design

Your Node API has been built as:

```text
mern-api:1.7.0
```

CI has tested it successfully.

### Design the deployment workflow

Required flow:

```text
Git commit
    ↓
CI
    ↓
Tests
    ↓
Docker build
    ↓
Security scan
    ↓
Registry
    ↓
Staging
    ↓
Validation
    ↓
Production
```

Your production deployment should reference:

```text
mern-api:1.7.0
```

or, for maximum immutability:

```text
registry.example.com/team/mern-api@sha256:<digest>
```

Rollback should select a known-good artifact:

```text
1.7.0
  ↓
1.6.4
```

It should not normally rebuild old source code unless there is a specific reason to do so.

### Questions

1. How will you connect the image to its Git commit?
2. How will you prevent production from rebuilding a different artifact?
3. How will you retain the previous image for rollback?
4. What permissions should CI have in the registry?
5. How will you verify the production image after deployment?

---

## 22. 🏗️ Month 2 Cumulative Project Update

Your containerized MERN project now evolves into a deployment pipeline:

```text
Developer
    │
    ▼
Git
    │
    ▼
CI
    │
    ├── Test
    ├── Build frontend
    ├── Build API image
    ├── Scan
    └── Tag
         │
         ▼
      Registry
         │
      ┌──┴──┐
      ▼     ▼
   Staging Production
```

Document the following:

### Image naming

```text
frontend:<version>
mern-api:<version>
```

### Version strategy

Choose and document one or both:

- semantic version
- Git commit SHA

### Traceability

Maintain enough metadata to connect:

```text
Image
  ↓
Commit
  ↓
Build
  ↓
Deployment
```

### Rollback

Document:

- current version
- previous known-good version
- rollback command or process
- post-rollback verification

---

## 23. 🎤 Final Interview Challenge

Answer aloud:

> Explain how you would deploy a Dockerized MERN application from Git commit to production.

A strong answer is:

```text
Developer pushes code
        ↓
CI runs tests
        ↓
Build Docker images
        ↓
Scan images
        ↓
Tag with immutable version
        ↓
Push to registry
        ↓
Deploy exact image to staging
        ↓
Run validation
        ↓
Promote same image to production
        ↓
Monitor
        ↓
Rollback to previous known-good image if necessary
```

Then explain why:

> I would avoid rebuilding specifically for production because that can introduce differences between the artifact tested in staging and the artifact actually deployed.

That is a strong production DevOps answer.

---

## 24. 📖 Today's Key Takeaways

You learned:

✅ Docker registries  
✅ Image tags  
✅ Versioned images  
✅ `docker tag`  
✅ `docker push`  
✅ `docker pull`  
✅ Immutable deployments  
✅ Image digests  
✅ Build-once/promote workflow  
✅ CI/CD artifact flow  
✅ Rollbacks  
✅ Registry security  
✅ MERN deployment traceability

Your deployment model is now:

```text
              Git
               │
               ▼
              CI
               │
        ┌──────┼──────┐
        ▼      ▼      ▼
      Test    Build   Scan
               │
               ▼
            Registry
               │
               ▼
            Staging
               │
            Validate
               │
               ▼
          Production
               │
          ┌────┴────┐
          │         │
        Monitor   Rollback
                    │
                    ▼
              Known-good image
```

### Core principle

> The artifact tested should be the artifact deployed.

---

## ⏭️ Next Lesson

Day 28 — Docker Logging, Monitoring & Resource Management

Topics to cover:

- Container logs
- Logging drivers and concepts
- Application logs versus container logs
- CPU and memory inspection
- Disk usage
- Health checks
- Container restart behavior
- MERN production incidents
- Docker observability
- Container failure and resource exhaustion interview scenarios
