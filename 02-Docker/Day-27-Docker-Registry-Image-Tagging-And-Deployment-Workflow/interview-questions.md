# Day 27 - Interview Questions: Docker Registry and Deployment Workflow

## Beginner Level

### Q1. What is a Docker registry?

A service that stores, distributes, and manages container images.

### Q2. What is a Docker tag?

A human-readable reference associated with image content, commonly used to identify a version or variant.

### Q3. Why should production not blindly use `latest`?

Because `latest` can point to different image content over time, making deployments less deterministic and harder to audit or roll back.

### Q4. What does `docker push` do?

It uploads a locally available image reference to a registry repository.

### Q5. What does `docker pull` do?

It downloads an image from a registry so it can be run on the current machine.

---

## Intermediate Level

### Q6. Explain build once, deploy many.

Build the application artifact once, test that artifact, store it in a registry, and promote the same artifact through staging and production instead of rebuilding for each environment.

### Q7. How would you roll back a Docker deployment?

Deploy the previous known-good image version or digest from the registry. Do not normally rebuild old source code for a rollback.

### Q8. Why are image digests useful?

A digest is a content-addressed identifier for exact image content. It is a stronger immutable reference than a mutable tag.

### Q9. What happens when you run `docker tag`?

Docker adds another name or reference to existing image content. It does not rebuild the image.

### Q10. How do you connect a Docker image to source code?

Use a commit-based image tag, release metadata, labels, a deployment record, and preferably record the exact image digest.

---

## Advanced Level

### Q11. Staging passed testing, but production is running a different image. How could that happen?

Possible causes include rebuilding for production, different dependency resolution, a different build context, a changed base image, or using a mutable tag such as `latest`.

### Q12. Why is artifact promotion safer than rebuilding for each environment?

The same image that was tested in staging is used in production. This removes differences introduced by a second build and improves reproducibility.

### Q13. What should a CI identity be allowed to do in a registry?

CI should have only the permissions required for its job, such as pushing to specific repositories. It should not automatically have unrestricted production administrator access.

### Q14. What should you consider when rolling back an application image?

Check database schema compatibility, migrations, configuration compatibility, traffic behavior, health checks, and the availability of the previous known-good image.

### Q15. How would you deploy an image with maximum content immutability?

Reference the registry-qualified image by digest:

```text
registry.example.com/team/mern-api@sha256:<digest>
```

### Q16. What is the supply chain in a Docker deployment?

It is the path from source repository and dependencies through CI, Docker build, security scanning, registry storage, and deployment. Each stage must be controlled and traceable.

## Short Strong Answers

**Why avoid `latest`?**  
It is mutable, so the same deployment configuration can run different content at different times.

**Why use a registry?**  
To store and distribute the exact image artifact needed by deployment environments.

**Why build once?**  
To ensure the artifact tested is the artifact deployed.

**Why use a digest?**  
It identifies exact image content and cannot silently move like a tag.

**How do you roll back?**  
Redeploy the previous known-good image version or digest from the registry.
