# DevOps Mentorship Program - Day 44

## Phase 3: CI/CD

### Docker Registry Integration with GitHub Actions
 
**Level:** Intermediate -> Professional  
**Focus:** Automatically building, scanning, tagging, and pushing MERN Docker images from CI

Yesterday's revision consolidated the CI/CD foundation. Today we continue the sequence by connecting the CI pipeline to a Docker registry.

The pipeline is changing from a validation-only pipeline into one that produces a durable, deployable production artifact:

```text
Git
  |
  v
GitHub Actions
  |
  v
Test -> Build -> Docker image -> Security scan
                                      |
                                      v
                               Registry login
                                      |
                                      v
                                 Docker push
                                      |
                                      v
                             Deployable artifact
```

---

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain why CI pipelines publish Docker images to a registry.
- Describe how registry authentication works conceptually.
- Tag images with a Git commit SHA and release version.
- Build and push Docker images from GitHub Actions.
- Explain why `latest` should not be the only production tag.
- Separate image build, security scanning, authentication, and push stages.
- Protect registry credentials and apply least privilege.
- Troubleshoot failed image pushes and missing image tags.
- Explain how registry integration fits into a MERN production project.
- Describe the complete CI-to-registry flow in an interview.

---

## 2. Where We Are Now

Previously, the pipeline ended after building an image on the temporary CI runner:

```text
Git push
  |
  v
GitHub Actions
  |
  v
npm ci -> Lint -> Tests -> Build -> Docker build
                                      |
                                      v
                            Image exists on runner
```

The runner may disappear when the job finishes. The image is therefore not yet available to a deployment system.

Today the pipeline stores the image in a registry:

```text
Git push
  |
  v
CI validation
  |
  v
Docker build
  |
  v
Image scan
  |
  v
Registry
  |
  +--> Staging
  |
  +--> Production
```

The registry becomes the permanent source of deployable images. This is the same **build once, store the artifact, and promote that exact artifact** principle introduced earlier.

---

## 3. Why Push Images to a Registry?

Without a registry, the result of the build is trapped on the CI runner:

```text
CI runner -> Docker image -> runner disappears
```

With a registry, the image becomes a durable artifact that can be downloaded by staging, production, or another deployment platform:

```text
CI runner -> Docker image -> Registry -> Staging -> Production
```

A registry provides:

- Centralized storage for container images.
- Distribution to deployment environments.
- Versioned tags and immutable digests.
- Access control and repository permissions.
- Artifact traceability.
- Retention and cleanup policies.
- A handoff point between CI and CD.
- Optional vulnerability scanning, signing, and verification features.

The registry is therefore part of the software supply chain, not merely a file store.

---

## 4. MERN Production Flow

The target architecture is:

```text
Developer
    |
    v
  Git push
    |
    v
GitHub Actions
    |
    +----------+----------+
    v          v          v
  Lint       Tests      Build
    |          |          |
    +----------+----------+
               |
               v
         Docker build
               |
               v
          Image scan
               |
          +----+----+
          |         |
       FAIL       PASS
          |         |
          v         v
        Stop   Registry login
                    |
                    v
                Docker push
                    |
                    v
             Versioned artifact
                    |
                +---+---+
                v       v
             Staging Production
```

The new component is the registry handoff:

```text
Docker image -> Registry login -> Docker push -> Registry
```

---

## 5. Docker Image Naming and Tags

Assume the registry hostname is:

```text
registry.example.com
```

An API image repository might be:

```text
registry.example.com/mern-api
```

A complete image reference includes a tag:

```text
registry.example.com/mern-api:8f31c2a
```

The components are:

```text
registry.example.com  -> Registry hostname
mern-api              -> Repository name
8f31c2a               -> Image tag
```

Other valid references might include:

```text
registry.example.com/mern-api:v1.4.0
registry.example.com/mern-api:8f31c2a
registry.example.com/mern-api:latest
```

The tag should help operators identify which source revision or release the image represents.

---

## 6. Git SHA Tagging

Suppose Git creates this commit:

```text
8f31c2a
```

CI can build and push:

```text
registry.example.com/mern-api:8f31c2a
```

Production can then report:

```text
Running image: registry.example.com/mern-api:8f31c2a
Source commit: 8f31c2a
```

This is valuable during incidents. Instead of saying, "Production is running the latest version," the team can identify the exact source revision.

A Git SHA tag supports this traceability chain:

```text
Image tag -> Git commit -> Pull request -> CI run -> Deployment
```

Use the full SHA when practical. A shortened SHA is easier to read, but it must remain unique within the repository's relevant history.

---

## 7. What About `latest`?

You may publish both a specific tag and a moving convenience tag:

```text
registry.example.com/mern-api:8f31c2a
registry.example.com/mern-api:latest
```

However, `latest` should not be the sole production identity.

For example:

```text
Monday: latest -> v1
Tuesday: latest -> v2
```

The command `docker pull mern-api:latest` means something different on each day. This makes auditing and rollback more difficult.

For production traceability, prefer:

- A Git commit SHA.
- A release version such as `v1.4.0`.
- The image digest when immutable identity is required.

Tags are useful human-readable pointers. Digests provide content-addressed identity:

```text
registry.example.com/mern-api@sha256:<digest>
```

---

## 8. Registry Authentication

A private registry must know who is pushing the image. Conceptually:

```text
GitHub Actions
      |
      v
Protected credentials or workload identity
      |
      v
Registry authentication
```

Never place credentials directly in source code, a Dockerfile, a workflow literal, or a README:

```yaml
# Never do this
password: my-secret-password
```

Instead, use protected GitHub Actions secrets or an identity mechanism supported by the registry:

```text
GitHub Secrets or federated identity
              |
              v
       Workflow authentication
              |
              v
             Registry
```

The authentication method must be limited to the permissions required by the workflow.

---

## 9. Least-Privilege Registry Access

If the pipeline only publishes images, it may only need permission to push images. It should not automatically receive permission to:

- Delete repositories.
- Modify unrelated repositories.
- Access production databases.
- Change deployment infrastructure.
- Read secrets unrelated to the image publication job.

The principle is:

```text
CI identity -> Minimum required registry permissions
```

Use separate identities for separate responsibilities when possible. For example, a build workflow may push to a registry, while a deployment identity may pull from it.

---

## 10. Docker Login

The interactive command is conceptually:

```bash
docker login registry.example.com
```

In CI, credentials should be supplied through protected inputs and `--password-stdin` so the password is not exposed as a command-line argument:

```yaml
- name: Log in to registry
  run: |
    echo "${{ secrets.REGISTRY_PASSWORD }}" \
      | docker login registry.example.com \
        --username "${{ secrets.REGISTRY_USERNAME }}" \
        --password-stdin
```

The security boundary is:

```text
Secret or identity
      |
      v
Protected workflow input
      |
      v
Docker login
      |
      v
Registry authentication
```

The exact login command varies by registry. The concept remains the same: authenticate before pushing, keep credentials outside source code, and grant only the permissions required.

---

## 11. GitHub Actions Build Example

The following is a simplified example for a backend located in `backend/`. It installs dependencies, tests the application, and builds an image tagged with the commit SHA.

```yaml
name: Build and Publish

on:
  push:
    branches:
      - main

permissions:
  contents: read

env:
  REGISTRY: registry.example.com
  IMAGE_NAME: mern-api

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
          cache-dependency-path: backend/package-lock.json

      - name: Install dependencies
        run: npm ci
        working-directory: backend

      - name: Test
        run: npm test
        working-directory: backend

      - name: Build Docker image
        run: |
          docker build \
            -t "$REGISTRY/$IMAGE_NAME:${{ github.sha }}" \
            ./backend
```

At the end of the build step, the image exists locally on the runner:

```text
CI runner -> registry.example.com/mern-api:<commit-sha>
```

It is not yet available to the deployment system. Login and push are still required.

---

## 12. Login and Push Steps

Add registry authentication before pushing:

```yaml
      - name: Log in to registry
        run: |
          echo "${{ secrets.REGISTRY_PASSWORD }}" \
            | docker login "$REGISTRY" \
              --username "${{ secrets.REGISTRY_USERNAME }}" \
              --password-stdin

      - name: Push image
        run: |
          docker push "$REGISTRY/$IMAGE_NAME:${{ github.sha }}"
```

The resulting flow is:

```text
Authenticate -> Build -> Scan -> Push
```

When the push succeeds, the registry contains an image such as:

```text
registry.example.com/mern-api:8f31c2a
```

The deployment system can now pull the exact artifact created by CI.

> Replace the example registry hostname, repository name, and secret names with values from your registry. Do not commit real credentials.

---

## 13. Scan Before Push

For a trusted release artifact, required security checks should normally run before the image enters the release repository:

```text
Docker build
      |
      v
Image scan
      |
      v
Required policy passes?
      |
  +---+---+
  |       |
 No      Yes
  |       |
  v       v
Stop  Registry login
          |
          v
       Docker push
```

This prevents the trusted repository from filling with images that policy would never allow into production. Some organizations intentionally push first to a quarantine repository, scan there, and promote only passing images. The important point is to define what a trusted artifact means and enforce that policy.

A production-oriented ordering is:

```text
Checkout
  |
  v
npm ci -> Lint -> Tests -> Application build
                                      |
                                      v
                               Docker build
                                      |
                                      v
                                Image scan
                                      |
                                  PASS?
                                 /     \
                               NO       YES
                               |          |
                             Stop   Registry login
                                          |
                                          v
                                      Docker push
```

---

## 14. Build Once, Deploy Many

After the image is pushed, CD should pull the existing image:

```text
Registry
   |
   v
Pull mern-api:8f31c2a
   |
   v
Staging
   |
   v
Health checks and smoke tests
   |
   v
Approval or policy
   |
   v
Production
```

CD should not rebuild the application. Rebuilding in each environment can produce different artifacts because of:

- A changed base image.
- Changed dependencies.
- Different build tools.
- Different environment variables.
- External downloads at build time.
- Different runner or deployment environments.

The reproducible approach is:

```text
CI -> One tested image -> Registry -> Staging -> Production
```

The same image reference, and preferably the same digest, should move through the environments.

---

## 15. Artifact Traceability and Rollback

Suppose production reports HTTP 500 errors. Operations can inspect the running image:

```text
Production image:
registry.example.com/mern-api:8f31c2a
```

They can then trace:

```text
8f31c2a
  |
  v
Git commit
  |
  v
Pull request
  |
  v
CI run and scan results
  |
  v
Deployment record
```

A registry should retain previous known-good versions so rollback does not require rebuilding:

```text
Current:  registry.example.com/mern-api:8f31c2a
Rollback: registry.example.com/mern-api:7aa9210
```

The previous image should remain available according to the team's retention policy.

---

## 16. Common Push Failures

### `denied: requested access to the resource is denied`

Possible causes include:

- Incorrect registry hostname.
- Incorrect repository name.
- Invalid credentials.
- Insufficient push permission.
- An expired or incorrect token.
- A repository that does not exist.
- A registry namespace that does not match the authenticated account.

Troubleshoot in this order:

```text
Build succeeded?
      |
      v
Registry URL correct?
      |
      v
Login successful?
      |
      v
Repository name correct?
      |
      v
Credentials valid and unexpired?
      |
      v
Push permission granted?
```

Do not rebuild immediately. The Docker build may be completely correct; the failure may only concern authentication or authorization.

### `manifest unknown`

This usually means the requested image or tag does not exist in the registry.

For example:

```text
CI pushed:       mern-api:8f31c2a
Deployment pulls: mern-api:8f31c2b
```

Trace the same value through every stage:

```text
Git SHA -> CI tag -> Registry tag -> Deployment tag
```

All four must agree. Also verify the registry hostname, repository path, image architecture, and whether the push job actually completed.

---

## 17. Practice Materials

The practical work is separated into companion files, matching the structure used in previous lessons:

- [Assignment](assignment.md): Build, scan, authenticate, and push the SHA-tagged image.
- [Commands](commands.md): Local Docker, registry, scanning, inspection, and troubleshooting commands.
- [Interview Questions](interview-questions.md): Registry, security, traceability, and deployment questions.
- [Quiz](quiz.md): Knowledge check, answer key, score guide, and reflection questions.

---

## 18. Day 44 Summary

CI produces and validates an artifact. The registry stores and distributes that artifact to deployment environments.

```text
Git
  |
  v
GitHub Actions
  |
  v
npm ci -> Lint -> Tests -> Build
                         |
                         v
                   Docker build
                         |
                         v
                   Security scan
                         |
                         v
                   Registry login
                         |
                         v
                    Docker push
                         |
                         v
                    mern-api:<SHA>
```

Remember these production principles:

- Build once.
- Test before publishing.
- Scan before promotion.
- Use traceable image tags.
- Protect registry credentials.
- Apply least privilege.
- Deploy the exact CI artifact.
- Retain previous versions for rollback.

### Interview takeaway

When asked how CI and Docker registries work together, remember:

```text
CI       -> Creates and validates the artifact
Registry -> Stores and distributes the artifact
CD       -> Deploys the exact artifact
```

---

## Next Lesson: Day 45

Next we will build on registry publishing with CI/CD environments and staging deployment:

```text
Registry
   |
   v
Staging
   |
   v
Environment configuration
   |
   v
Health checks
   |
   v
Smoke tests
   |
   v
Promotion decision
```

This moves the MERN project from publishing an image to safely deploying and validating that image.