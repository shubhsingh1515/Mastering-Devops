# DevOps Mentorship Program - Day 38

## Phase 2: Docker & Containers

### Docker Registry & Image Lifecycle

**Level:** Intermediate -> Professional  
**Focus:** Docker registries, image promotion, tagging strategy, and production artifact management

> Day 37 connected Docker deployment strategies with rollback. Today we focus on where the release artifacts live, how environments consume them, and how to keep deployments traceable and reversible.

The lifecycle for today's lesson is:

```text
Git commit
   |
   v
Build image
   |
   v
Test and scan
   |
   v
Version image
   |
   v
Push to registry
   |
   v
Promote to staging
   |
   v
Promote the same artifact to production
   |
   v
Health check and monitor
   |
   v
Rollback to a known-good image when necessary
```

---

## 1. Learning Objectives

By the end of this lesson, you should understand:

- What a Docker registry is and why production systems use one.
- The difference between an image, container, repository, registry, tag, and digest.
- The difference between public and private registries.
- How to tag and push an image.
- How a deployment server pulls an exact image.
- Why `latest` is a poor production release strategy.
- How image promotion works between development, staging, and production.
- Why build-once-deploy-many improves reproducibility.
- How image retention affects rollback.
- How to investigate image pull failures.
- How to describe container artifact management in an interview.

---

## 2. What Is a Docker Registry?

A Docker registry is a service that stores and distributes container images. It is the shared location from which build systems, staging servers, and production servers can pull approved artifacts.

A useful comparison is:

```text
Git repository
      |
      +--> source code

Docker registry
      |
      +--> container images
```

Your local machine might contain:

```text
mern-api:v1
```

A registry can store multiple versions of that application:

```text
registry.example.com/mern-api:v1
registry.example.com/mern-api:v2
registry.example.com/mern-api:v3
```

A production server then pulls the exact artifact it has been instructed to run. The production server should normally consume a tested image from the registry instead of rebuilding source code on the production host.

### Why a registry matters

Without a registry, every server would need access to source code and the tools required to build it. That creates inconsistent build environments and makes it difficult to answer a basic operational question: "Which exact artifact is running?"

A registry provides:

- Centralized artifact storage.
- Distribution to multiple environments.
- Access control for private application code.
- Version history and release traceability.
- A source for repeatable deployment and rollback.

---

## 3. Production Architecture

A controlled production path looks like this:

```text
Developer
    |
    v
Git repository
    |
    v
Controlled build environment
    |
    v
Docker image
    |
    v
Private registry
    |
    +-------------> Staging
    |
    +-------------> Production
```

This is a major improvement over:

```text
Developer laptop
       |
       v
Production server
```

The first flow separates source code, build, storage, testing, and runtime responsibilities. It also gives the team a stable artifact that can be inspected, scanned, promoted, and rolled back.

---

## 4. Image, Container, Repository, Registry, Tag, and Digest

These terms are related but not interchangeable.

### Image

An image is the packaged, read-only application artifact. It contains application files, dependencies, metadata, and the instructions needed to create a container.

Example:

```text
mern-api
```

### Container

A container is a created or running instance of an image. The image is the artifact; the container is a runtime instance.

```text
Image:     mern-api:v1.2.0
Container: a running process created from that image
```

### Repository

A repository is a named collection of related image versions.

```text
registry.example.com/mern-api
```

That repository may contain tags such as `v1.0.0`, `v1.1.0`, and `v1.2.0`.

### Registry

A registry is the service that hosts repositories. Examples include Docker Hub, GitHub Container Registry, Azure Container Registry, and self-hosted registry products.

```text
registry.example.com
```

### Tag

A tag is a human-readable reference associated with an image version.

```text
v1.2.0
abc1234
```

Tags are convenient, but a tag can be moved to point at different content. Do not assume every tag is immutable unless your registry policy enforces that rule.

### Digest

A digest is a content-addressable identifier, commonly displayed as a SHA-256 value. It identifies the exact image content.

```text
sha256:0123456789abcdef...
```

An image reference can use a tag:

```text
registry.example.com/mern-api:v1.2.0
```

or a digest:

```text
registry.example.com/mern-api@sha256:0123456789abcdef...
```

A tag is easier for people to read. A digest is stronger when exact immutability matters.

---

## 5. Understanding a Full Image Reference

Consider this reference:

```text
registry.example.com/team/mern-api:v1.5.0
```

It contains:

```text
registry.example.com  -> registry host
team/mern-api         -> repository path
v1.5.0                -> tag
```

A digest reference looks like this:

```text
registry.example.com/team/mern-api@sha256:abc123...
```

There is no mutable version tag in the final reference. The digest identifies the content that the runtime must pull.

---

## 6. Public and Private Registries

### Public registry

A public registry allows images or repositories to be visible to a broad audience, depending on its access settings. Public images are useful for open-source software and intentionally shared base images.

### Private registry

A private registry restricts access to authorized users, build systems, and deployment identities. Private registries are common for proprietary application code and internal services.

Production registry controls should include:

- Authentication.
- Authorization.
- TLS encryption.
- Repository-level permissions.
- Audit logs.
- Vulnerability scanning.
- Retention and deletion policies.
- Credential rotation.

A production server may need permission to pull images, but it usually should not have permission to delete every repository or overwrite release tags.

This follows least privilege:

```text
Deployment identity -> pull approved images
Build identity      -> push new images
Administrator       -> manage repositories and policies
```

---

## 7. Image Tagging Strategy

A production tag should help answer two questions:

1. What release is this?
2. Which source revision produced it?

A useful strategy is to publish both a semantic release tag and a commit tag:

```text
registry.example.com/mern-api:v1.4.0
registry.example.com/mern-api:a83f29c
```

Conceptually:

```text
Release 1.4.0
       |
       +--> v1.4.0
       |
       +--> a83f29c
```

Semantic versioning helps humans understand release order and compatibility. The commit tag provides source traceability.

Other useful metadata may include:

- Build number.
- Branch or release channel.
- Build timestamp.
- Source repository URL.
- Git revision label.
- SBOM or scan result reference.

Do not create tags that hide the source or release identity. A tag such as `production-final-2` is less useful than a tag tied to a commit and release version.

---

## 8. Why `latest` Is Dangerous

Suppose production runs:

```text
mern-api:latest
```

Over time, the tag may move:

```text
Monday:    latest -> v1
Tuesday:   latest -> v2
Wednesday: latest -> v3
```

When someone asks, "What exact version is production running?", the tag alone does not provide a reliable answer. A later `docker pull` may retrieve different content under the same name.

Problems caused by relying only on `latest` include:

- Unclear release identity.
- Difficult incident investigation.
- Accidental upgrades during a pull or restart.
- Ambiguous rollback instructions.
- Weak audit history.
- Different servers running different content.

Use a controlled version tag or digest for production:

```yaml
services:
  api:
    image: registry.example.com/mern-api:v1.5.0
```

For stronger immutability:

```yaml
services:
  api:
    image: registry.example.com/mern-api@sha256:0123456789abcdef...
```

The exact policy depends on the deployment platform, but production should never depend on an unexplained moving tag.

---

## 9. Build Once, Promote the Artifact

The key workflow is:

```text
Build
  |
  v
mern-api:abc123
  |
  v
Push to registry
  |
  +--> Staging
  |
  +--> Production after approval
```

The unsafe alternative is:

```text
Build for staging
       |
       v
Rebuild for production
```

Even when the source appears identical, rebuilds can differ because of dependency resolution, base image changes, timestamps, network state, or build configuration.

With build-once-deploy-many:

1. Build from a known source revision.
2. Run tests and security checks.
3. Push the image to the registry.
4. Deploy the exact image to staging.
5. Validate staging.
6. Promote the exact artifact to production.

The environment configuration can change, but the application image remains the same.

This separates:

```text
Artifact identity from environment configuration
```

That separation is one of the foundations of reproducible delivery.

---

## 10. Tagging, Pushing, and Pulling an Image

Build a local image:

```bash
docker build -t mern-api:1.0.0 ./backend
```

Tag the same image for a registry:

```bash
docker tag mern-api:1.0.0 registry.example.com/mern-api:1.0.0
```

The two names can point to the same local image content:

```text
mern-api:1.0.0
registry.example.com/mern-api:1.0.0
```

Authenticate using the registry's supported mechanism, then push:

```bash
docker push registry.example.com/mern-api:1.0.0
```

On a deployment server, pull the published artifact:

```bash
docker pull registry.example.com/mern-api:1.0.0
```

Run it directly:

```bash
docker run --rm registry.example.com/mern-api:1.0.0
```

Or reference it in production Compose:

```yaml
services:
  api:
    image: registry.example.com/mern-api:1.0.0
```

In this production model, use `image:` rather than `build:` because the image was already built, tested, and published.

---

## 11. Development Compose Versus Production Compose

During development, local source builds are convenient:

```yaml
services:
  api:
    build: ./backend
```

In production, use the tested registry artifact:

```yaml
services:
  api:
    image: registry.example.com/mern-api:v1.5.0
```

The difference is intentional:

```text
Development -> build local source for fast iteration
Production  -> pull a controlled artifact for repeatability
```

Do not put registry passwords in a `Dockerfile` or commit them to a Compose file. Configure authentication through the deployment platform, Docker credential helpers, secret stores, or short-lived identity-based credentials.

---

## 12. Image Digests and Exact Reproducibility

A tag can be moved. A digest is calculated from content. This makes a digest a stronger deployment reference.

```text
Tag
 |
 v
Image manifest
 |
 v
Digest
```

A deployment using a digest says:

```text
Run this exact content, not whatever a tag points to later.
```

A practical release record should capture both the friendly release and the exact digest:

```text
Release: v1.5.0
Commit:  a83f29c
Digest:  sha256:0123456789abcdef...
```

This gives operators a human-readable release name and a machine-verifiable artifact identity.

---

## 13. Image Scanning and Supply Chain Checks

Before production, the image should ideally be scanned for known vulnerabilities and checked against the organization's release policy.

```text
Docker build
    |
    v
Tests
    |
    v
Image scan
    |
    +--> Critical finding -> investigate or block
    |
    +--> Accepted result  -> publish and promote
```

Scanning does not prove that an image is completely secure. It provides automated detection for known issues and supports a repeatable decision process.

Other useful supply-chain checks include:

- Pinning important base image versions.
- Generating a software bill of materials.
- Signing images where supported.
- Verifying image signatures before deployment.
- Recording the build source revision.
- Restricting who can push or overwrite release tags.

---

## 14. Image Retention and Cleanup

A registry may contain many releases:

```text
mern-api:v1
mern-api:v2
mern-api:v3
mern-api:v4
mern-api:v5
```

Deleting old images reduces storage cost, but deleting too aggressively removes rollback options. Retention must balance:

```text
Storage cost
    +
Rollback requirements
    +
Compliance requirements
    +
Incident investigation needs
```

A sensible policy may keep:

- All currently deployed images.
- The last several known-good releases.
- Releases required by an audit or support window.
- Digests referenced by deployment records.

Never delete an image merely because a newer release exists. First verify that the image is no longer a rollback target, is not used by another environment, and is covered by the retention policy.

---

## 15. Rollback with a Registry

Suppose production currently runs `v2` and the previous known-good version is `v1`:

```text
Current:  v2
Previous: v1
```

The registry contains both:

```text
registry.example.com/mern-api:v1
registry.example.com/mern-api:v2
```

If `v2` fails:

```text
Production v2
      |
      v
Failure detected
      |
      v
Pull known-good v1
      |
      v
Deploy v1
      |
      v
Health check
      |
      v
Monitor recovery
```

The rollback should deploy the already-tested artifact. Rebuilding from source during an incident is slower and may produce a different result.

Record the incident:

```text
Current version:  v2
Previous version: v1
Failure:          health endpoint returned 500
Rollback action:  deployed registry.example.com/mern-api:v1
Result:           health endpoint recovered
```

Remember that application rollback and database rollback are separate problems. A new database migration may not be compatible with the old application. Backward-compatible migrations and expand-and-contract patterns make rollback safer.

---

## 16. Troubleshooting Image Pull Failures

A deployment may report:

```text
pull access denied
```

or:

```text
manifest unknown
```

Possible causes include:

- The image name is wrong.
- The tag does not exist.
- The repository path is wrong.
- The registry is unavailable.
- DNS or network access is failing.
- Authentication failed.
- The deployment identity lacks pull permission.
- The image exists in a different registry or region.
- The image architecture is incompatible with the host.

Use this investigation order:

```text
Can the server reach the registry?
          |
          v
Can the deployment authenticate?
          |
          v
Does the repository exist?
          |
          v
Does the tag or digest exist?
          |
          v
Does the identity have pull permission?
          |
          v
Can the host run this image architecture?
```

Do not immediately rebuild the image. First determine whether this is a naming, permissions, connectivity, availability, or compatibility problem.

---

## 17. MERN Production Example

A registry might contain three application repositories:

```text
registry.example.com/
  mern-api
  mern-frontend
  mern-nginx
```

A release may use the same version across all three:

```text
registry.example.com/mern-api:v1.5.0
registry.example.com/mern-frontend:v1.5.0
registry.example.com/mern-nginx:v1.5.0
```

Production Compose could reference them like this:

```yaml
services:
  api:
    image: registry.example.com/mern-api:v1.5.0

  frontend:
    image: registry.example.com/mern-frontend:v1.5.0

  nginx:
    image: registry.example.com/mern-nginx:v1.5.0
```

MongoDB may be self-managed or provided as a managed service. The important image-lifecycle principle remains the same: production pulls known artifacts rather than building from source during deployment.

---

## 18. Complete Artifact Lifecycle

The full release process is:

```text
Git revision
    |
    v
Docker build
    |
    v
Automated tests
    |
    v
Security scan
    |
    v
Tag with release and commit
    |
    v
Push to private registry
    |
    v
Deploy to staging
    |
    v
Health check and approval
    |
    v
Promote exact artifact to production
    |
    v
Record digest and deployment metadata
    |
    v
Retain previous known-good artifact
```

When a release fails:

```text
Production release
       |
       v
Failure or unhealthy signal
       |
       v
Select known-good image
       |
       v
Deploy previous artifact
       |
       v
Health check
       |
       v
Document incident
```

---

## 19. Interview Takeaway

If an interviewer asks, "A production server needs your new Docker image. Walk me through how you would get it there," a strong answer is:

> I would build the image from the intended source revision in a controlled build environment, tag it with a traceable release and commit identifier, run tests and security checks, push it to a private registry, and deploy that exact artifact to staging first. After validation, production would pull the same image reference, preferably using an immutable digest or tightly controlled version tag. I would retain the previous known-good image so rollback is fast and deterministic.

This answer demonstrates:

- Artifact management.
- Security and least privilege.
- Testing.
- Traceability.
- Promotion between environments.
- Health validation.
- Rollback readiness.

---

## 20. Day 38 Summary

Remember this sentence:

> Build the artifact once, store it in a trusted registry, promote the exact artifact through environments, and keep a known-good version available for rollback.

Production rules to remember:

- Build in a controlled environment.
- Test and scan before production.
- Push to a registry.
- Use traceable image versions.
- Prefer immutable references.
- Promote the same artifact.
- Do not rely only on `latest`.
- Retain rollback versions.
- Secure registry access.
- Separate application rollback from database rollback planning.

### Next lesson

Day 39 continues Phase 2 with the next Docker production topic and moves toward the final Docker and Containers assessment.
