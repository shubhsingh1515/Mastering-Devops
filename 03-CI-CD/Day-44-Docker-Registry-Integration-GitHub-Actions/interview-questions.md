# Day 44 - Interview Questions: Docker Registry Integration

## 1. Why push Docker images to a registry?

A registry provides centralized storage and distribution for container artifacts. It allows staging and production to pull the exact image built and tested by CI.

## 2. Why tag Docker images with a Git SHA?

A Git SHA creates direct traceability between the deployed container image and the source revision that produced it. It improves debugging, auditing, release identification, and rollback.

## 3. Why is `latest` alone a weak production tag?

`latest` is mutable and does not identify the source revision. A commit SHA, release version, or digest provides stronger identity.

## 4. When should an image be pushed?

After the required build, test, and security gates pass. An organization may push first to a quarantine repository, but only compliant artifacts should enter the trusted release flow.

## 5. How do you authenticate CI to a registry?

Use protected CI/CD secrets or a supported workload identity. Credentials must remain outside source code, be exposed only to trusted jobs, and have the minimum permissions needed to push the intended repository.

## 6. Why use `--password-stdin` with `docker login`?

It avoids placing the password directly in command arguments, reducing accidental exposure through process inspection or logs. The credential should still come from a protected secret or identity mechanism.

## 7. What happens if `docker push` fails with access denied?

First determine whether the image build succeeded. Then verify the registry URL, repository path, login result, credential validity, namespace, and push permissions. I would not rebuild unnecessarily because the failure may only concern authentication or authorization.

## 8. What does `manifest unknown` usually mean?

The requested image or tag cannot be found in the registry. Compare the Git SHA, CI tag, pushed registry tag, repository path, and deployment reference for an exact match.

## 9. Why should CD deploy the image produced by CI instead of rebuilding it?

Build once and promote the same artifact. Rebuilding can produce different results because of changed base images, dependencies, build tools, environment variables, or external downloads.

## 10. What is the role of a registry between CI and CD?

The registry is the artifact handoff and storage point. CI creates, tests, scans, and publishes the image; CD pulls that exact image and promotes it through environments.

## 11. What permissions should a publishing workflow have?

Only the permissions required to authenticate and push to the intended repository. It should not automatically delete repositories, access production databases, modify unrelated resources, or deploy to production.

## 12. Why scan before publishing a trusted image?

Scanning can identify known vulnerabilities before the image becomes available to deployment systems. The organization's policy should define which findings block publication.

## 13. How would you protect registry credentials in pull request workflows?

Treat fork pull requests as untrusted. Run validation with limited permissions and do not expose registry or production credentials to code that has not been trusted or merged.

## 14. How does a registry support rollback?

It retains previous known-good image tags or digests. Production can reference an earlier artifact without rebuilding the application.

## 15. Why is an image digest useful in addition to a tag?

A tag can be moved to another image, while a digest identifies the exact image content. Deploying by digest provides immutable content identity.

## 16. Give a strong answer for the complete flow.

> A Git push triggers GitHub Actions. The runner checks out the commit, installs dependencies using the lockfile, runs linting and tests, and builds the application. CI then builds a Docker image tagged with the commit SHA and scans it for vulnerabilities. If the required gates pass, the workflow authenticates to the private registry using protected credentials and pushes the exact image. Deployment systems can then pull that immutable or tightly versioned artifact for staging and production.

## 17. Why is a Docker registry more than a place to store images?

It is part of the software supply chain. A registry provides distribution, versioning, artifact traceability, access control, retention, and often vulnerability scanning or signing. It becomes the handoff point between CI and deployment.
