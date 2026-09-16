# DevOps Mentorship Program - Day 37

## Phase 2: Docker & Containers

### Docker Production Deployment Strategies & Rollback

**Level:** Intermediate → Professional  
**Focus:** Safe image promotion, deployment, rollback, and release strategy for a Dockerized MERN application

> Today is Monday, so we are back to a new topic after yesterday's Sunday revision.
>
> We have now covered the major Docker building blocks:
>
> Containers
>    ↓
> Images
>    ↓
> Dockerfiles
>    ↓
> Compose
>    ↓
> Networking
>    ↓
> Volumes
>    ↓
> Image Optimization
>    ↓
> Security
>    ↓
> Registries
>    ↓
> Logging / Monitoring
>    ↓
> Nginx
>    ↓
> Configuration
>    ↓
> Health Checks
>    ↓
> Production Operations
>
> Today we connect those concepts into an actual deployment and rollback workflow.

---

## 1. 🎯 Learning Objectives

By the end of today's lesson, you should understand:

- What a production Docker deployment actually looks like.
- Why images should be immutable deployment artifacts.
- Why `latest` is dangerous as a deployment strategy.
- How versioned image tags work.
- How to deploy a new MERN API version.
- How to perform a rollback.
- What “build once, deploy many” means.
- Why health checks matter during deployment.
- How to reduce deployment risk.
- How to explain Docker deployment strategy in interviews.

---

## 2. 🧠 The Production Problem

Suppose your current application is:

```bash
mern-api:v1
```

Your developers release:

```bash
mern-api:v2
```

The naive approach is:

```text
Build v2
   ↓
Deploy v2
   ↓
Hope everything works
```

That is not enough for production.

A better process is:

```text
Code
 ↓
Build image
 ↓
Test image
 ↓
Push image
 ↓
Deploy exact image
 ↓
Health check
 ↓
Monitor
 ↓
Keep or rollback
```

This is the foundation of modern container deployment.

### Why this matters

A production deployment is not a random restart; it is a controlled release of a verified application artifact.

---

## 3. 📦 Docker Image as a Release Artifact

Think of an image as a packaged application artifact.

For example:

```bash
mern-api:2026.09.14
```

or:

```bash
mern-api:abc1234
```

where `abc1234` could represent a Git commit.

### Architecture

```text
Git commit
    │
    ▼
Docker build
    │
    ▼
mern-api:abc1234
    │
    ▼
Registry
    │
    ├── Staging
    │
    └── Production
```

The same image can move through environments.

### Why this is important

An image is not just a local build artifact. It becomes the actual release candidate that is tested, stored, promoted, and deployed.

---

## 4. ⭐ Build Once, Deploy Many

This is one of today's most important concepts.

### Bad approach

```text
Development
    ↓
Build image A

Staging
    ↓
Build image B

Production
    ↓
Build image C
```

Even if all three builds come from the same source, differences can creep in.

### Better approach

```text
Source
  ↓
Build ONE image
  ↓
Test
  ↓
Registry
  ↓
Staging
  ↓
Production
```

For example:

```bash
mern-api:8f31c2a
```

is the exact artifact tested in staging.

Then production deploys:

```bash
mern-api:8f31c2a
```

not a newly rebuilt image.

### Why this matters

If different environments rebuild the same app separately, you introduce drift:

- different dependency states
- different base image layers
- different environment configuration
- different build timestamps
- hidden variation between environments

A real production workflow wants the exact same artifact to move through the pipeline.

---

## 5. 🏷️ Image Tags

You might see:

```bash
mern-api:latest
mern-api:v1.0.0
mern-api:v1.1.0
mern-api:8f31c2a
```

These have different operational properties.

### `latest`

Convenient:

```bash
docker pull mern-api:latest
```

But what exactly is `latest`?

It changes.

That makes troubleshooting and rollback harder.

### Version tag

```bash
mern-api:v1.2.0
```

Much clearer.

You know which release you're deploying.

### Commit tag

```bash
mern-api:8f31c2a
```

Excellent traceability.

You can associate the image with a specific source revision.

### Production takeaway

Use explicit version or commit tags in deployment workflows.

`latest` is convenient but risky in production.

---

## 6. 🔒 Immutable Image References

An even stronger reference is an image digest:

```bash
mern-api@sha256:<digest>
```

The digest identifies the exact image content.

### Conceptually

```text
Tag
 ↓
Image
 ↓
Digest
```

For production, immutable references are valuable because:

> “Deploy version 8f31c2a”

is clearer than:

> “Deploy whatever `latest` currently means.”

### Why it matters in operations

- reproducibility
- precise rollback target
- easier incident diagnosis
- stronger release accountability

---

## 7. 🧱 MERN Deployment Flow

Imagine your repository:

```text
mern-app/
├── frontend/
├── backend/
├── nginx/
├── docker-compose.yml
└── README.md
```

A production release might be:

```text
Developer
   │
   ▼
Git commit
   │
   ▼
Docker build
   │
   ▼
mern-api:abc123
   │
   ▼
Registry
   │
   ▼
Staging
   │
   ▼
Tests
   │
   ▼
Production
```

This is the broad flow production teams aim for.

---

## 8. 🐳 Building the Image

Example:

```bash
docker build -t mern-api:abc123 ./backend
```

Check:

```bash
docker images
```

Then run:

```bash
docker run --rm mern-api:abc123
```

Or through Compose:

```bash
docker compose up -d
```

The exact production command depends on how your deployment configuration is structured.

### Important idea

The image is the deployed artifact. It should be built intentionally and treated as a release object, not a tossaway local build.

---

## 9. 📤 Push to Registry

After testing:

```bash
docker tag mern-api:abc123 registry.example.com/mern-api:abc123
```

Then:

```bash
docker push registry.example.com/mern-api:abc123
```

Your registry now contains:

```text
registry
   │
   ├── mern-api:abc123
   ├── mern-api:def456
   └── mern-api:789abc
```

Each release is independently addressable.

### Why this matters

You can deploy version `abc123`, then later move to `def456`, and still have the previous version available for rollback.

---

## 10. 🚀 Deploying a New Version

Suppose production currently runs:

```bash
mern-api:abc123
```

New release:

```bash
mern-api:def456
```

### Deployment

```text
Production
    │
    ├── Current → abc123
    │
    ▼
Pull def456
    │
    ▼
Start def456
    │
    ▼
Health check
    │
    ├── FAIL → rollback
    │
    └── PASS
          │
          ▼
       monitor
```

This is dramatically safer than blindly replacing the old version.

### Why this is safer

The process includes a health gate and a rollback path before the deployment is considered complete.

---

## 11. ❤️ Health Checks Become Deployment Gates

This connects directly to Day 34–36.

Suppose:

```bash
api:v2
```

starts successfully.

But:

```bash
GET /health
```

returns:

```text
500
```

Should you send production traffic to it?

No.

Your deployment process should recognize:

```text
Container started
        ≠
Application ready
```

Therefore:

```text
New version
    ↓
Health check
    ↓
Healthy?
```

becomes a deployment decision point.

### Production principle

Running is not the same as ready.

---

## 12. 🔄 Rollback

Imagine:

```text
v1 → working
v2 → deployed
```

Then users report:

```text
500 Internal Server Error
```

Your rollback strategy should be simple.

Return to:

```text
v1
```

### Architecture

```text
v1
 │
 │ working
 ▼
Deploy v2
 │
 │ failure
 ▼
Rollback
 │
 ▼
v1
```

This is why keeping previous image versions is so important.

---

## 13. ⚡ Fast Rollback

A good rollback should not require:

```text
Git checkout
   ↓
Rebuild
   ↓
Retest
   ↓
Push
   ↓
Deploy
```

Instead:

```text
Production
   ↓
Current: v2
   ↓
Rollback
   ↓
Known-good v1
```

The old image already exists.

This is another benefit of immutable artifacts.

### Key idea

Fast rollback is a release-management design goal, not an accidental outcome.

---

## 14. 🧠 Rollback Does Not Mean Database Rollback

This is an advanced but extremely important point.

Suppose:

```text
API v1
   ↓
Database schema v1
```

Then you deploy:

```text
API v2
   ↓
Database schema v2
```

Now API v2 fails.

You cannot automatically assume:

```text
Rollback API
+
Rollback database
```

is safe.

Database migrations may be irreversible.

For example:

```text
v2 migration
DROP old_column
```

If you deploy v2 and then immediately deploy v1:

```text
v1 expects old_column
```

but:

```text
old_column no longer exists
```

Your rollback may fail.

### This is why production database migrations require careful compatibility planning.

---

## 15. ⭐ Interview Question

### Q: How do you design database migrations for safe application rollbacks?

A strong answer:

> “I avoid migrations that make the previous application version immediately incompatible. I prefer backward-compatible or expand-and-contract migration strategies, where schema changes are introduced in stages so old and new application versions can coexist during deployment.”

That is a strong production-level answer.

---

## 16. 🔁 Expand-and-Contract Migration

Imagine you need to rename:

```text
username
```

to:

```text
display_name
```

### Dangerous approach

```text
Rename immediately
      ↓
Old API breaks
```

### Safer approach

#### Phase 1 — Expand

Add:

```text
display_name
```

while keeping:

```text
username
```

#### Phase 2

Deploy application capable of using both.

```text
username
display_name
```

#### Phase 3

Migrate data.

#### Phase 4 — Contract

Only after all old versions are gone:

```text
Remove username
```

### Conceptually

```text
Expand
  ↓
Compatible application
  ↓
Migrate
  ↓
Remove old structure
```

This becomes very important as you move toward Kubernetes and advanced CI/CD.

---

## 17. 🟢 Deployment Strategies

There are several ways to release a new application version.

### Recreate

```text
Stop old
   ↓
Start new
```

Simple, but may cause downtime.

### Rolling

```text
v1 v1 v1
 ↓
v2 v1 v1
 ↓
v2 v2 v1
 ↓
v2 v2 v2
```

Traffic gradually moves to the new version.

We will study this much more deeply in Kubernetes.

### Blue-Green

```text
             Load Balancer
                  │
          ┌───────┴───────┐
          ▼               ▼
       BLUE v1          GREEN v2
          │               │
       Current           New
```

Test green.

Then switch traffic.

```text
BLUE
  X

GREEN
  ✓
```

Rollback can be very fast:

```text
GREEN → BLUE
```

### Canary

Send a small percentage of traffic to the new version:

```text
v1 → 95%
v2 → 5%
```

If healthy:

```text
v1 → 75%
v2 → 25%
```

Then:

```text
v1 → 0%
v2 → 100%
```

These advanced strategies come later in the curriculum.

---

## 18. 🏭 What Compose Can and Cannot Do

For your current learning environment, Docker Compose is excellent for:

- local environments
- small deployments
- multi-container application management
- development/staging
- simple single-host deployments

But do not pretend Compose alone gives you:

- Kubernetes-level orchestration
- automatic sophisticated rolling deployments
- cluster scheduling
- self-healing across many nodes

That is one reason Kubernetes exists.

### Lesson goal

For now, your goal is to understand the underlying deployment principles before orchestration is introduced.

---

## 19. 🔍 Production Troubleshooting

Suppose:

```text
v2 deployed
```

but users see:

```text
500 errors
```

Use evidence.

### Step 1

Check:

```bash
docker compose ps
```

### Step 2

Check logs:

```bash
docker compose logs --tail 100 api
```

### Step 3

Check health:

```bash
docker inspect <api-container>
```

### Step 4

Check Nginx:

```bash
docker compose logs --tail 100 nginx
```

### Step 5

Check resource usage:

```bash
docker stats
```

### Step 6

Compare versions:

```text
Current → v2
Previous → v1
```

### Step 7

If root cause is confirmed and rollback is safer:

```text
v2
 ↓
rollback
 ↓
v1
```

Then investigate v2 separately.

### Key lesson

Production debugging is evidence-driven, not guess-driven.

---

## 20. 🛡️ Deployment Best Practices

1. Use immutable/versioned images  
   Prefer:
   ```bash
   mern-api:abc123
   ```
   over relying exclusively on:
   ```bash
   mern-api:latest
   ```

2. Build once  
   Do not rebuild the artifact differently for production.

3. Test before production  
   At minimum:
   ```text
   Build
    ↓
   Unit tests
    ↓
   Integration tests
    ↓
   Image test
    ↓
   Staging
    ↓
   Production
   ```

4. Keep previous versions  
   Rollback requires something to roll back to.

5. Use health checks  
   Do not send traffic to an unhealthy application.

6. Make rollback easy  
   A rollback should be an operational procedure, not an emergency invention.

7. Record deployment metadata  
   Know:
   - Version
   - Commit
   - Build time
   - Image digest
   - Environment
   - Deployment time

---

## 21. ❌ Common Mistakes

### Mistake 1

Deploying `latest` without knowing which exact image it refers to.

### Mistake 2

Rebuilding the image on the production server.

Prefer producing the artifact in a controlled build process and deploying that exact artifact.

### Mistake 3

Deleting the previous image immediately after deployment.

You may need it for rollback.

### Mistake 4

Rolling back application code without considering database compatibility.

### Mistake 5

Calling a deployment successful because:

```text
Container = Up
```

instead of checking:

```text
Application = Healthy
```

### Mistake 6

Using:

```bash
restart
```

as your rollback strategy.

Restarting `v2` does not turn `v2` into `v1`.

---

## 22. 🎤 Interview Preparation

### Beginner

#### Q1. What is a Docker image tag?

A human-readable reference associated with an image, such as `v1.2.0` or a commit identifier.

#### Q2. Why is `latest` risky in production?

Because it is mutable and does not uniquely identify a specific release.

### Intermediate

#### Q3. What does “build once, deploy many” mean?

Build a tested artifact once and promote that exact artifact through environments instead of rebuilding it separately for each environment.

#### Q4. What is rollback?

Returning a deployment to a known-good previous application version.

### Advanced

#### Q5. Why isn't application rollback always enough?

Because the application may depend on database/schema changes that aren't compatible with the previous version.

#### Q6. How would you make rollback safer?

Use:

- versioned/immutable images
- known-good previous artifacts
- backward-compatible migrations
- health checks
- deployment metadata
- tested rollback procedures

#### Q7. What would you check before deciding to rollback?

A strong answer:

> “I’d first confirm the issue is associated with the new release, inspect application and proxy logs, check health and resource metrics, determine the user impact, and compare with the previous known-good version. If rollback reduces risk, I’d execute the documented rollback while preserving evidence for root-cause analysis.”

That is a production-minded answer.

---

## 23. 🔁 Review Questions From Earlier Lessons

### Q1 — Networking

Inside the API container, why is:

```bash
mongodb://mongodb:27017
```

normally correct?

Because `mongodb` is the Docker service name available through the Compose network.

### Q2 — Nginx

What commonly causes:

```text
502 Bad Gateway
```

between Nginx and Node?

Incorrect upstream configuration, service discovery/network problems, wrong port, unavailable API, or an unhealthy backend.

### Q3 — Configuration

Why should production configuration generally not be baked into the Docker image?

It couples the artifact to one environment and can expose sensitive information.

### Q4 — Health

What's the difference between:

```text
Container Up
```

and:

```text
Application Healthy
```

A running container only indicates container/process state; health checks can determine whether a defined application health condition actually passes.

### Q5 — Persistence

Does a Docker volume automatically provide disaster recovery?

No. Persistence and backup/disaster recovery are separate concerns.

---

## 24. 📝 Quiz — 10 Questions

### Q1
Which is generally the strongest production image reference?

A. `latest`  
B. Container name  
C. Specific version/commit tag or immutable digest  
D. Random generated name

### Q2
What does “build once, deploy many” mean?

A. Build the same source separately on every server  
B. Build one tested artifact and promote it through environments  
C. Never test the image  
D. Rebuild production manually

### Q3
Why are previous image versions retained?

A. To increase disk usage  
B. To enable traceability and rollback  
C. To disable networking  
D. To replace health checks

### Q4
A new API container is Up but `/health` fails. Should production traffic be considered safe?

A. Yes  
B. No  
C. Only if Nginx is running  
D. Only if MongoDB is running

### Q5
Why can database migrations complicate rollback?

A. Databases cannot be accessed by Docker  
B. New schema changes may be incompatible with the previous application version  
C. Docker automatically deletes databases  
D. Nginx controls MongoDB schema

### Q6
What is a blue-green deployment?

A. Two database volumes  
B. Two application environments/versions where traffic can be switched between them  
C. Two Docker networks with random names  
D. A Docker image optimization method

### Q7
What is a canary deployment?

A. Deploying a new version to a small portion of traffic before broader rollout  
B. Deploying only to development  
C. Deleting the previous version  
D. Running MongoDB twice

### Q8
Which is the best rollback approach?

A. Rebuild from memory  
B. Deploy a previously tested known-good image  
C. Restart the failed container repeatedly  
D. Delete production data

### Q9
Which deployment sequence is safest?

A. Deploy → hope → investigate  
B. Build → test → deploy → health check → monitor → rollback if needed  
C. Delete old → build on production → deploy  
D. Restart → restart → restart

### Q10
What is a major advantage of immutable image references?

A. They make containers permanent  
B. They provide reproducibility and precise release identification  
C. They remove the need for testing  
D. They automatically back up databases

### ✅ Answer Key

1 → C  
2 → B  
3 → B  
4 → B  
5 → B  
6 → B  
7 → A  
8 → B  
9 → B  
10 → B

### Score

- 9–10 → 🟢 Excellent
- 7–8 → 🟡 Good
- 5–6 → 🟠 Review deployment concepts
- <5 → 🔴 Revisit Docker production fundamentals

---

## 25. 💻 Coding / Scripting Exercise

Create:

```bash
deploy.sh
```

The script should accept an image version:

```bash
./deploy.sh abc123
```

Conceptually it should:

1. Validate version
2. Pull exact image
3. Update deployment configuration
4. Start/recreate API
5. Check service status
6. Check health
7. Report success/failure

### Example flow

```text
Deploying mern-api:abc123

Pulling image...
✓

Starting service...
✓

Checking health...
✓

Deployment successful.
```

If health fails:

```text
Deployment failed.
```

### Recommended action

```text
Rollback to previous known-good version.
```

### Bonus

Store:

```bash
PREVIOUS_VERSION
CURRENT_VERSION
```

so the script can support:

```bash
./rollback.sh
```

This is excellent preparation for future CI/CD work.

---

## 26. 🧪 Hands-On Lab

Use your existing MERN application.

### Step 1 — Build a release

```bash
docker build -t mern-api:v1 ./backend
```

### Step 2 — Create a second release

Make a small application change.

Then:

```bash
docker build -t mern-api:v2 ./backend
```

### Step 3 — Run v1

```bash
mern-api:v1
```

Verify:

```bash
curl http://localhost/api/health
```

### Step 4 — Deploy v2

Replace the image reference in your deployment configuration.

Start:

```bash
docker compose up -d
```

### Step 5 — Verify

```bash
docker compose ps
```

and:

```bash
docker compose logs api
```

and:

```bash
curl http://localhost/api/health
```

### Step 6 — Simulate a bad release

Make v2 intentionally fail its health check.

Deploy it.

Observe:

```text
v2
 ↓
unhealthy
```

### Step 7 — Rollback

Return to:

```bash
v1
```

Then verify:

```bash
curl http://localhost/api/health
```

You have now performed your first controlled deployment/rollback exercise.

---

## 27. 📋 Mini Assignment

Create:

```bash
DEPLOYMENT.md
```

Document:

- Release
- How do we build the image?
- Versioning
- How is an image identified?
- Deployment
- How is the image deployed?
- Health
- How do we know it's healthy?
- Rollback
- How do we return to the previous version?
- Database
- Are database migrations backward-compatible?
- Incident response
- When should we rollback?

---

## 28. 🏆 Monthly Cumulative Project — Production Release Requirement

For this month's MERN project, add:

### Release Management

Your project should now support:

```text
Git commit
    │
    ▼
Docker build
    │
    ▼
Versioned image
    │
    ▼
Registry
    │
    ▼
Staging
    │
    ▼
Health validation
    │
    ▼
Production
    │
    ┌────┴────┐
    │         │
  Healthy   Failed
    │         │
    ▼         ▼
 Monitor   Rollback
```

### Required

- [ ] Versioned image tags
- [ ] Exact image references
- [ ] Previous image retained
- [ ] Health check
- [ ] Deployment procedure
- [ ] Rollback procedure
- [ ] Deployment documentation
- [ ] Database migration strategy

---

## 29. 🏆 Project Interview Challenge

Explain your deployment strategy in under two minutes.

Your answer should cover:

1. How the image is built
2. How it is versioned
3. Where it is stored
4. How staging is validated
5. How production is deployed
6. How health is verified
7. How rollback works
8. How database migrations are handled

A strong answer sounds like:

> “We build an immutable Docker image from a specific Git revision and tag it with the commit identifier. We push that exact artifact to our registry and deploy the same image to staging and production. Before accepting traffic, we validate application health. We retain the previous known-good image so rollback doesn't require rebuilding. Database migrations are designed to remain backward-compatible during deployment so that an application rollback doesn't immediately break against the newer schema.”

That is interview-quality Docker deployment knowledge.

---

## 30. 📖 Recommended Documentation

Review the official documentation for:

- Docker image tags
- Docker image digests
- Docker registry operations
- Docker Compose deployment
- Docker health checks
- Docker restart policies

Also review your previous notes on:

- `docker build`
- `docker tag`
- `docker push`
- `docker pull`
- `docker compose`
- `docker inspect`
- `docker compose logs`

---

## 31. 🧠 Day 37 Summary

Today's core principle:

> A production deployment should be a controlled movement of a known artifact, not a manual rebuild-and-hope operation.

### Remember:

```text
Git commit
    ↓
Build
    ↓
Test
    ↓
Immutable image
    ↓
Registry
    ↓
Staging
    ↓
Health
    ↓
Production
    ↓
Monitor
    ↓
Rollback if necessary
```

### Most important interview concepts

```text
Build once, deploy many
        +
Immutable artifacts
        +
Health-gated deployment
        +
Fast rollback
        +
Backward-compatible migrations
```

And remember:

```text
Application rollback
       ≠
Database rollback
```

That distinction separates basic Docker knowledge from production engineering thinking.

---

## 🔮 Day 38 — Next Lesson

Tomorrow we will continue the Docker production track by going deeper into Docker image lifecycle, registry management, release versioning, and artifact promotion, tying today’s deployment strategy into a complete container release workflow.

Then we will finish the remaining Docker assessment before moving to the next phase of the curriculum.
