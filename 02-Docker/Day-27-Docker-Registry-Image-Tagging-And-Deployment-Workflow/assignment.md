# Day 27 - Assignment: Docker Registry, Tagging and Deployment

## Objective

Build, version, publish, deploy, and roll back a Docker image using a registry-style workflow. Practice the principle that the artifact tested in staging should be the artifact deployed in production.

---

## Part 1: Build a Versioned Image

From your backend directory, build the API image:

```bash
docker build -t mern-api:1.0 .
```

### Questions

1. What source code is included in the image?
2. Which Dockerfile was used?
3. How can you connect this image to a Git commit?

Use this command to record the current commit when working in a Git repository:

```bash
git rev-parse --short HEAD
```

---

## Part 2: Add Tags

Add a commit-based tag:

```bash
docker tag mern-api:1.0 mern-api:<commit-sha>
```

Add a registry-qualified version tag:

```bash
docker tag \
  mern-api:1.0 \
  registry.example.com/team/mern-api:1.0
```

Inspect the results:

```bash
docker images
```

### Expected observation

The different references can point to the same local image content. Tagging does not rebuild the image.

---

## Part 3: Push to a Registry

Replace the example registry with one you are authorized to use:

```bash
docker login registry.example.com
docker push registry.example.com/team/mern-api:1.0
```

### Security requirements

- Do not commit registry credentials.
- Do not paste tokens into source files.
- Use a protected CI secret or credential store.
- Give CI only the registry permissions it requires.

If you do not have registry access, complete the local build, tagging, and inspection portions.

---

## Part 4: Pull and Deploy

On a deployment machine or a second environment:

```bash
docker pull registry.example.com/team/mern-api:1.0
```

Run the image:

```bash
docker run -d \
  --name mern-api-staging \
  -p 3000:3000 \
  registry.example.com/team/mern-api:1.0
```

Test it:

```bash
curl http://localhost:3000/health
```

Record the image reference that staging used.

---

## Part 5: Design the CI/CD Flow

Write a pipeline design that follows this sequence:

```text
Git commit
    ↓
CI tests
    ↓
Docker build
    ↓
Image security scan
    ↓
Version and commit tags
    ↓
Push to registry
    ↓
Deploy to staging
    ↓
Validation
    ↓
Promote same image to production
```

### Required explanation

Explain why production must pull the image built and tested by CI instead of rebuilding from source.

### Expected answer

Rebuilding can produce a different artifact because dependencies, base images, build context, or timestamps may differ. Promoting the same image preserves consistency between staging and production.

---

## Part 6: Compare Tagging Strategies

Compare these references:

```text
mern-api:latest
mern-api:1.4.2
mern-api:8f31c2a
mern-api@sha256:<digest>
```

For each one, explain:

- whether it is human-friendly
- whether it can move
- how it helps traceability
- whether it is appropriate for production

### Expected conclusion

`latest` is convenient but mutable. A release or commit tag is clearer. A digest gives the strongest exact-content reference.

---

## Part 7: Practice a Rollback

Deploy version `1.4.2`, then simulate a new release with `1.4.3`.

If version `1.4.3` fails validation, roll back:

```bash
docker pull registry.example.com/team/mern-api:1.4.2
docker stop mern-api-staging
docker rm mern-api-staging
docker run -d \
  --name mern-api-staging \
  -p 3000:3000 \
  registry.example.com/team/mern-api:1.4.2
```

Verify:

```bash
curl http://localhost:3000/health
```

### Question

Why is this better than rebuilding old source code?

### Expected answer

The known-good image already exists, was previously tested, and can be redeployed without introducing new build differences.

### Additional consideration

Rollback planning must include database migrations. Reverting the application image does not automatically revert an incompatible database schema change.

---

## Part 8: Practical Assessment

Your Node API has been built as:

```text
mern-api:1.7.0
```

CI has tested it successfully.

Design a deployment plan that includes:

- registry-qualified image name
- image tag and digest strategy
- staging deployment
- production promotion
- monitoring
- rollback to the previous known-good image
- permissions required by CI
- verification after deployment

### Required workflow

```text
Git commit
    ↓
CI
    ↓
Tests
    ↓
Build
    ↓
Scan
    ↓
Registry
    ↓
Staging
    ↓
Validation
    ↓
Production
```

---

## Part 9: MERN Project Update

Document these image names:

```text
registry.example.com/team/mern-frontend:<version>
registry.example.com/team/mern-api:<version>
```

Document your chosen version strategy:

- semantic version
- Git commit SHA
- both semantic version and commit SHA

Create a traceability record:

```text
Image
  ↓
Digest
  ↓
Commit
  ↓
Build
  ↓
Staging deployment
  ↓
Production deployment
```

For rollback documentation, record:

- current version
- previous known-good version
- image digest
- rollback command or process
- post-rollback health verification

---

## Part 10: Submission Checklist

- [ ] API image was built successfully
- [ ] Image has an explicit version tag
- [ ] Image has a commit-based tag or traceability record
- [ ] Registry-qualified image name is documented
- [ ] Image was pushed or the local registry workflow was simulated
- [ ] Image was pulled and run in a staging-style environment
- [ ] `/health` was tested
- [ ] CI/CD build-once/promote workflow was documented
- [ ] `latest` risks were explained
- [ ] Rollback to a previous image was practiced or documented
- [ ] Registry access follows least privilege
- [ ] Database migration rollback concerns were considered
