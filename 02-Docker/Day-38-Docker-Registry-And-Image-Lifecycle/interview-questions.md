# Day 38 - Interview Questions: Docker Registry & Image Lifecycle

## Beginner Level

### Q1. What is a Docker registry?

A Docker registry is a service that stores and distributes container images. Build systems push images to it and deployment environments pull images from it.

### Q2. What is the difference between an image and a container?

An image is the packaged application artifact. A container is a created or running instance of that image.

### Q3. What is a repository?

A repository is a named collection of related images, usually containing multiple tags or release versions.

### Q4. What is an image tag?

A tag is a human-readable reference such as `v1.5.0`, `latest`, or a Git commit identifier such as `a83f29c`.

### Q5. What does `docker push` do?

It uploads a locally tagged image to a registry repository.

### Q6. What does `docker pull` do?

It downloads an image from a registry so a Docker host can run it.

## Intermediate Level

### Q7. Why is `latest` not ideal for production?

`latest` is commonly mutable. It can point to different image content over time, which makes the running release difficult to identify and rollback less deterministic.

### Q8. What is an image digest?

A digest is a content-addressable identifier, usually a SHA-256 value, that identifies exact image content.

### Q9. What does build-once-deploy-many mean?

It means an image is built and tested once, pushed to a registry, and then promoted through staging and production without rebuilding it for each environment.

### Q10. Why should production use `image:` instead of `build:` in Compose?

Production should consume a tested artifact from a registry. Using `build:` there can rebuild source on the production host and reduce reproducibility.

### Q11. Why use both a semantic version and a commit tag?

The semantic version is easy for people to understand, while the commit tag connects the image directly to the source revision that produced it.

### Q12. What permissions should a production deployment identity have?

It should normally have the minimum permissions needed to authenticate and pull approved images. It should not automatically have broad permissions to push, delete, or administer all repositories.

## Advanced Level

### Q13. Why can rebuilding the same source produce a different image?

Dependency resolution, base image changes, build timestamps, network content, tool versions, and build configuration can differ. Building once and promoting the resulting artifact avoids this drift.

### Q14. How do image digests improve deployment safety?

A digest points to exact content, so a later tag movement cannot silently change what the deployment pulls.

### Q15. What should an image retention policy consider?

It should balance storage cost, rollback requirements, compliance, incident investigation, currently deployed images, and the number of known-good releases the team must retain.

### Q16. What are common causes of `pull access denied`?

Wrong registry or repository, wrong tag, failed authentication, missing pull permission, unavailable registry, DNS or network failure, and an image that does not exist.

### Q17. What is the difference between an application rollback and a database rollback?

An application rollback changes the running image. A database rollback changes persisted data or schema and may be unsafe or impossible if migrations are destructive. The two must be planned separately.

### Q18. What is image promotion?

Image promotion is moving or approving the same tested artifact for use in the next environment, such as staging and then production, without rebuilding it.

## Scenario Questions

### Q19. Production runs `mern-api:latest`. An incident occurs. What is your first concern?

Determine the exact image digest and source revision actually running on each host. A moving tag alone is not sufficient release evidence.

### Q20. Staging passed, but production cannot pull the image. How do you investigate?

Check registry connectivity, DNS, authentication, repository and tag spelling, image existence, deployment identity permissions, registry availability, and host architecture. Do not rebuild until the pull problem is understood.

### Q21. A new image passes its container start check but its health endpoint returns 500. Should it be promoted?

No. The image is not ready to receive production traffic. Inspect logs, configuration, dependency connectivity, and the health-check contract before promotion.

### Q22. Why retain the previous production image?

It provides a known-good recovery target and preserves the ability to investigate what changed between releases.

## Strong Answer Template

### How would you manage Docker artifacts for production?

> I would build the image from a known Git revision in a controlled environment, run tests and vulnerability checks, and publish it to a private registry. I would tag it with a semantic release and commit identifier, record its digest, and deploy that same artifact to staging and production. Production would use an immutable digest or a tightly controlled version tag rather than relying only on `latest`. I would retain the previous known-good image for deterministic rollback and give deployment identities only pull permissions.

### Why should production not build images from source?

> Building in production mixes build and runtime responsibilities and makes the result dependent on the production host. A controlled, tested artifact from a registry is more reproducible, auditable, and easier to roll back.
