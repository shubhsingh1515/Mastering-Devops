# Day 45 - Interview Questions: Staging, Smoke Tests, and Promotion

## 1. Why do we need staging if CI runs tests already?

CI validates the code and build process. Staging validates the actual deployable artifact in a production-like infrastructure environment, including configuration, network access, service communication, and runtime setup.

---

## 2. What is the purpose of a staging environment?

A staging environment gives the team a safe place to validate the release before it reaches real users. It reduces risk, catches infrastructure problems, and proves that the deployment works in a realistic environment.

---

## 3. What is the difference between development, staging, and production?

Development is for active coding and debugging. Staging is production-like and used for validation. Production serves real users and has stricter availability, security, and reliability requirements.

---

## 4. Should the application image be rebuilt in every environment?

Normally, no. A better pattern is to build a single validated artifact and deploy that same image across environments while only changing environment-specific configuration.

---

## 5. What is promotion?

Promotion is moving an already validated artifact from one environment to the next, usually from staging to production, without rebuilding it.

---

## 6. What is a smoke test?

A smoke test is a lightweight post-deployment test to confirm that the service is alive and fundamentally working. It checks whether the app is ready to continue through the release process.

---

## 7. What are examples of good smoke tests for a MERN app?

Examples include:

- `GET /health`
- `GET /api/health`
- `GET /api/products`

These checks confirm response code, service availability, and basic functionality.

---

## 8. What should happen if the staging health endpoint returns 502?

The promotion should stop immediately. The release should be treated as failed and the root cause should be investigated before further progression.

---

## 9. Why is the same Docker image useful across staging and production?

It improves reproducibility and confidence. The production environment is deploying the exact artifact that was validated in staging instead of a different image built from the same source.

---

## 10. Why should staging and production secrets be separated?

To reduce risk and prevent test workloads or compromised staging systems from affecting production. Production data should stay isolated from non-production systems.

---

## 11. What is a promotion gate?

A promotion gate is a pass/fail rule that decides whether an artifact may move forward. Examples include health checks, smoke tests, security checks, and manual approvals.

---

## 12. What is the difference between deployment and promotion?

Deployment means placing the artifact in an environment. Promotion means advancing an already validated artifact from one environment to the next.

---

## 13. Why is `localhost` a common bug in Dockerized apps?

Inside a container, `localhost` points to the same container, not to another service. If the API needs MongoDB, it should connect to the service name such as `mongodb`, not the loopback interface.

---

## 14. What should you investigate when staging fails but CI passed?

You should investigate environment variables, service networking, database credentials, container health, Nginx or reverse proxy behavior, and any runtime configuration differences between CI and staging.

---

## 15. What is the best interview answer for staging vs production?

> CI proves the code is valid; staging proves the deployable artifact works in a production-like environment; production is where the release is exposed to real users.

---

## 16. Why should production use the same image that passed staging?

Because it preserves a traceable, previously validated artifact and makes rollback easier. If production fails, the team can revert to the exact known-good image instead of rebuilding from an uncertain state.

---

## 17. Should you always require a manual approval after smoke tests?

Not necessarily. It depends on risk, compliance, release maturity, and operational needs. Some teams use automation-only promotion; others add a human approval gate for high-risk release paths.

---

## 18. What is the relationship between smoke tests and full regression tests?

Smoke tests are quick deployment checks. Full regression or end-to-end tests are broader and more comprehensive. Smoke tests are used early to stop bad deployments; full tests are used for deeper validation.

---

## 19. Why is environment isolation important for data access?

Because staging should not accidentally change or observe real user data. It is safer to isolate data sources between environments to reduce risk and prevent operational mistakes.

---

## 20. What is a strong production promotion answer?

> I would deploy the exact CI-built image to staging, validate its runtime behavior with health checks and smoke tests, and only promote it to production if those gates pass. Production would receive the same image with only its environment-specific configuration and secrets, and a rollback would use the last known-good image if needed.

---

## 21. Why is environment-specific configuration crucial?

Because the same application binary or image can behave differently depending on runtime values such as `NODE_ENV`, `PORT`, or `MONGO_URI`. The same artifact should not assume one configuration across all environments.

---

## 22. How would you explain promotion in a DevOps interview?

I would say that promotion is not a rebuild; it is the controlled movement of a previously validated artifact through environments after passing the required checks.

---

## 23. What is the main difference between a health check and a smoke test?

A health check is one specific validation of service readiness. A smoke test can be a small set of checks that confirm the service is operational enough to continue the deployment pipeline.

---

## 24. What are the main things that can differ between staging and production even when the image is the same?

Examples include:

- environment variables
- secrets
- network configuration
- database endpoint
- DNS and TLS setup
- scaling and resource limits
- IAM permissions

---

## 25. How does this improve rollback?

Because the team knows exactly which image was validated and promoted. If the new release fails, they can quickly redeploy the last good artifact instead of rebuilding and revalidating from scratch.

---

## 26. How would you respond to the question: "The staging deployment is healthy, but production fails immediately after promotion. What could be different?"

I would immediately consider production configuration, production secrets, network paths, DNS, TLS, IAM, database endpoints, resource limits, and external service availability. The artifact may be the same, but the environment is not.

---

## 27. Why do organizations separate CI and CD?

CI is responsible for validation and artifact creation. CD is responsible for controlled release through environments. Separating them makes quality gates and release controls clearer and safer.

---

## 28. What should promotion gate criteria include?

Typical criteria include:

- CI passed
- security scans passed
- image built and published
- staging deployment succeeded
- health checks passed
- smoke tests passed
- approved environment and secrets are in place

---

## 29. What do interviewers usually look for in this topic?

They usually want to hear that you understand:

- the purpose of staging
- the difference between deployment and promotion
- smoke tests as quick operational checks
- environment-specific configuration and secrets
- controlled release flow and rollback safety

---

## 30. Final interview answer to memorize

> I would build and validate the Docker image in CI, publish it to a registry, and deploy that exact image to staging with staging-specific configuration and secrets. I would then run health checks and smoke tests against the actual deployed service to confirm it is functioning in a production-like environment. If those checks pass, I would promote the same image to production using production-specific settings, with a documented rollback path to the last known-good artifact if anything fails.
