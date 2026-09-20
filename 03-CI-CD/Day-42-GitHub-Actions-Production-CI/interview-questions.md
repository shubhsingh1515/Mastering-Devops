# Day 42 - Interview Questions: GitHub Actions Production CI

## 1. What is GitHub Actions?

GitHub Actions is a repository-integrated automation platform that runs workflows in response to events such as pushes, pull requests, schedules, releases, or manual dispatches.

## 2. What is a workflow?

A workflow is a YAML-defined automation process stored in `.github/workflows/`. It contains triggers, jobs, permissions, variables, and steps.

## 3. What is a job?

A job is a logical unit of work executed on a runner. Jobs can run in parallel or depend on other jobs with `needs`.

## 4. What is a step?

A step is an individual command or reusable action inside a job. Steps in one job run in order and share that job's workspace.

## 5. What is a runner?

A runner is the machine or execution environment that executes a job. It can be GitHub-hosted or self-hosted.

## 6. How would you trigger CI for a Node.js application?

I would trigger it for pull requests targeting the protected branch and for pushes to the main branch. I might also add manual or scheduled triggers for maintenance and security workflows.

## 7. Why use `npm ci` instead of `npm install` in CI?

When a lockfile exists, `npm ci` performs a clean installation based on the locked dependency tree. This improves reproducibility and exposes differences between `package.json` and the lockfile.

## 8. Why control the Node.js version?

A controlled version reduces differences between developer machines and runners. It makes failures reproducible and prevents an unexpected runtime upgrade from changing pipeline behavior.

## 9. What is dependency caching?

Dependency caching stores reusable package data so later workflow runs can install more quickly. It improves speed but must not be required for correctness because caches can expire or be unavailable.

## 10. What is the difference between a cache and an artifact?

A cache helps future runs avoid repeated work. An artifact is a useful output from the current run, such as a build directory, test report, coverage report, or Docker metadata.

## 11. What is a quality gate?

A quality gate is a condition that must pass before a later stage continues. Examples include linting, tests, builds, security scans, approvals, and health checks.

## 12. How do you stop Docker construction after tests fail?

Place validation in one job and Docker construction in another job with `needs: validate`. If validation fails, the dependent Docker job is skipped by default.

## 13. Why tag a Docker image with the Git commit SHA?

The commit SHA connects the image to an exact source revision. It improves traceability, debugging, auditing, release identification, and rollback.

## 14. Why is `latest` alone a weak production tag?

`latest` is mutable and does not identify the source revision. It makes it difficult to determine what is running and can make rollback ambiguous.

## 15. How should secrets be handled in GitHub Actions?

Store them in GitHub encrypted secrets or an external secret manager, expose them only to trusted jobs and environments, use least-privilege permissions, and never commit or print them.

## 16. How should fork pull requests be handled?

Treat fork code as untrusted. Run validation with limited permissions and do not expose production credentials or deployment capabilities to the pull request job.

## 17. Why separate CI and CD?

CI validates and builds code. CD promotes a trusted artifact through staging and production. Separating them reduces accidental releases and makes approval and environment controls clearer.

## 18. What should happen if linting fails?

The validation job should fail and later quality-dependent jobs should not proceed. The failure should be visible in the pull request checks.

## 19. What should happen if tests fail?

The workflow should fail the required check, prevent image publication or deployment, and provide logs or reports that help the developer fix the problem.

## 20. What is an artifact in a containerized pipeline?

The final release artifact is commonly a Docker image identified by a versioned tag and preferably an image digest. Test reports and build directories can also be workflow artifacts.

## 21. What does build once and promote mean?

CI creates one tested and scanned artifact, stores it, and promotes that same artifact through staging and production. Rebuilding in each environment can produce different content from what was tested.

## 22. How can jobs run in parallel safely?

Independent jobs such as lint, unit tests, and security checks can run in parallel. A later Docker or publish job should use `needs` on all required upstream jobs.

## 23. How would you reduce a 20-minute pipeline to 8 minutes?

Measure each stage first, then cache dependencies, parallelize independent work, avoid unnecessary jobs, optimize tests, improve Docker layer reuse, and reuse artifacts. I would verify the change with timing data.

## 24. Tests pass locally but fail in CI. What do you investigate?

I would compare Node versions, operating systems, lockfiles, environment variables, database readiness, filesystem and timezone assumptions, network access, test ordering, and the exact runner logs.

## 25. Why should an image be scanned before publishing?

Scanning can identify known vulnerabilities before the image becomes available to deployment systems. The release policy should define which findings block publication.

## 26. What is the difference between liveness and readiness?

Liveness shows that a process is alive. Readiness shows that the application is prepared to receive traffic, which may include dependency and initialization checks.

## 27. Can a successful Docker build prove that the application works?

No. A build proves that the image was constructed, not that the application starts correctly, connects to dependencies, passes health checks, or serves correct responses.

## 28. What permissions should a basic CI workflow have?

It should request only what it needs. A read-only validation workflow can commonly use `permissions: contents: read`. Publishing and deployment should use separate, tightly scoped permissions.

## 29. How would you protect production deployment?

Require successful upstream checks, restrict deployment to protected branches or releases, use GitHub environment approvals, scope secrets to production, use short-lived credentials, and retain the tested image tag and digest.

## 30. Give a strong interview answer for a Node.js CI design.

> I would trigger CI on pull requests and relevant branch pushes. The runner would use a controlled Node version, check out the code, install dependencies with `npm ci`, run linting and unit or integration tests, and build the application. For a containerized service, I would build an image tagged with the commit SHA and scan it before publication. Docker publication and deployment would depend on the required quality gates, while production would use protected environments and controlled approvals.
