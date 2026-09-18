# Day 41 - Interview Questions: CI/CD Fundamentals

## 1. What is Continuous Integration?

Continuous Integration is the practice of frequently integrating code changes into a shared repository and automatically validating those changes with checks such as linting, tests, and builds.

## 2. What is Continuous Delivery?

Continuous Delivery keeps validated software in a deployable state. The artifact is ready for release, but production deployment may require a manual approval or business decision.

## 3. What is Continuous Deployment?

Continuous Deployment automatically releases changes to production after all required automated gates pass.

## 4. What is the difference between Continuous Delivery and Continuous Deployment?

Delivery means the software is always ready to deploy. Deployment means the software is automatically deployed to production after validation.

## 5. Describe a CI/CD pipeline for a MERN application.

A Git push or pull request triggers the pipeline. The runner checks out the revision, installs dependencies from the lockfile, runs linting and unit tests, runs integration tests where practical, builds the application, builds a Docker image tagged with the Git commit SHA, and runs required security checks. After the gates pass, the image is stored in a registry and the release pipeline promotes that exact artifact to staging, validates health, and eventually deploys it to production.

## 6. Why should tests run before Docker deployment?

Tests provide an early quality gate. Running them before publication or deployment prevents a known-broken change from becoming a shared or production artifact and reduces wasted build and deployment time.

## 7. What is a pipeline gate?

A pipeline gate is a condition that must pass before later stages continue. Examples include successful tests, an acceptable vulnerability scan, required code review, or a passing post-deployment health check.

## 8. What happens when a required CI test fails?

The relevant job should fail and downstream actions such as image publication or deployment should be skipped according to the pipeline policy. The failure should be visible through logs and test reports.

## 9. Why use a Git commit SHA in a Docker image tag?

It creates a direct relationship between the artifact and source revision. This improves traceability, incident investigation, reproducibility, and rollback. A release can identify exactly which commit produced the running image.

## 10. Why is `latest` alone insufficient for production?

`latest` is mutable and does not identify a specific source revision. It makes it harder to know what is running and can make rollback ambiguous. Use an immutable or uniquely versioned tag and preferably record the image digest as well.

## 11. What is an artifact?

An artifact is a build output that a later process can consume. Examples include a Docker image, frontend build files, an npm package, a test report, or a security report.

## 12. What does build once and promote mean?

It means CI creates one tested and scanned artifact, stores it, and promotes that same artifact through staging and production. Rebuilding separately for each environment can produce different content and weakens confidence in what was tested.

## 13. How do you handle secrets in CI/CD?

Store secrets in the CI platform's secret store or an external secret manager. Give them only to jobs that need them, restrict them to trusted branches or environments, mask them in logs, rotate them when necessary, and never commit them to source code or bake them into images.

## 14. How should fork pull requests be handled?

Treat fork code as untrusted. Run safe validation with limited permissions, avoid exposing production credentials, and require trusted branch or environment approval before any deployment capability is available.

## 15. What is the difference between unit, integration, and end-to-end tests?

Unit tests validate small isolated functions or modules. Integration tests validate interactions between components such as an API and MongoDB. End-to-end tests validate a complete user flow through the browser, frontend, API, and supporting services.

## 16. Why is a test pyramid useful?

It encourages many fast unit tests, fewer integration tests, and a smaller number of expensive end-to-end tests. This provides useful coverage while keeping feedback time and test maintenance manageable.

## 17. What should you investigate when tests pass locally but fail in CI?

Compare runtime versions, operating systems, lockfiles, dependency installation, environment variables, database availability, filesystem assumptions, timezone and locale, test ordering, network access, service readiness, and the exact CI logs. Reproduce using the same runtime or container when possible.

## 18. Why use `npm ci` in CI?

When a lockfile exists, `npm ci` performs a clean installation based on the locked dependency graph. This reduces dependency drift and makes the runner environment more consistent with the committed lockfile.

## 19. Why build Docker images inside CI?

CI provides a controlled and repeatable build environment. The resulting image can be tested, scanned, tagged, stored in a registry, and promoted as a known artifact.

## 20. What is the difference between liveness and readiness?

Liveness indicates whether the process is alive. Readiness indicates whether the instance is prepared to receive traffic, which may include dependency or initialization checks. A live process can still be unready.

## 21. What should happen after deploying to staging?

Run health checks and smoke tests, inspect logs and deployment status, verify the expected artifact identity, and confirm that critical user paths work before promoting the artifact to production.

## 22. What is a rollback in CI/CD?

A rollback returns service to a previously known-good artifact or release. Versioned image tags and recorded digests make rollback faster and more reliable because the previous image does not need to be rebuilt.

## 23. Can a successful container start prove deployment success?

No. A process can start while the application is unable to connect to its database, accept requests, or serve correct responses. Health, readiness, smoke, and dependency checks provide stronger evidence.

## 24. What is a strong answer to: Why do companies use CI/CD?

CI/CD reduces manual deployment work, catches problems earlier, makes releases repeatable, improves feedback speed, provides traceability, enforces quality gates, and supports safer promotion and rollback of tested artifacts.

## 25. Senior scenario: A pipeline takes 40 minutes. What would you improve?

Measure the slow stages first. Cache safe dependency downloads, run independent jobs in parallel, use appropriate test selection for pull requests, preserve full tests for protected branches, reduce unnecessary Docker build context, use efficient layer ordering, and keep security scans visible without weakening required controls.

## 26. Senior scenario: A deployment passed CI but failed in staging. What do you check?

Confirm the deployed image digest matches the tested artifact, inspect environment configuration, verify service dependencies and secrets, check health and readiness results, review logs and network connectivity, compare staging runtime versions, and determine whether the failure is application, configuration, infrastructure, or data related.

## 27. Senior scenario: How would you protect the production deployment job?

Restrict it to protected branches or release events, require successful upstream jobs, use environment approval where appropriate, give it minimal permissions, use short-lived credentials or workload identity, keep secrets out of pull request jobs, and record the artifact, approver, and deployment result.

## 28. Project answer template

> A Git push or pull request triggers CI. The runner checks out the source, installs dependencies from the lockfile, runs linting and unit tests, executes integration tests where practical, builds the application, and creates a Docker image tagged with the Git commit SHA. Required security checks run before the image is published. The release process promotes the exact tested image to staging, validates health and smoke tests, and then deploys to production under the project's approval policy. The previous known-good image and its digest are retained for rollback.
