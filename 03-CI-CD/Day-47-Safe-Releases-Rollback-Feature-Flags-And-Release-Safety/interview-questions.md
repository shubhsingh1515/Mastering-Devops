# Day 47 - Interview Questions: Safe Releases and Rollback

## 1. What is a feature flag?

A feature flag is a runtime control that enables or disables a piece of application behavior without requiring a new application artifact.

### Strong answer

> A feature flag separates deployment from release. I can deploy code with the new path disabled, validate the application, enable the path for a controlled audience and disable it quickly if the business behavior is unsafe.

---

## 2. Why do feature flags reduce deployment risk?

They reduce the initial blast radius and make feature-level recovery faster.

### Strong answer

> A flag lets me expose a risky feature gradually instead of giving it to every user immediately. If the defect is isolated to that feature, I can turn it off without rebuilding the image or rolling back unrelated functionality.

---

## 3. What is the difference between deployment and release?

Deployment puts code into an environment. Release makes the functionality available to users.

### Strong answer

> I can deploy v2 to production with `newCheckout` disabled. The infrastructure and application can be validated before the feature is released to internal users, a percentage of traffic or everyone.

---

## 4. What is the difference between feature rollback and application rollback?

A feature rollback changes a runtime flag. An application rollback restores a previous application artifact or version.

### Strong answer

> If only checkout is failing and it is behind a flag, I disable the flag first. If the whole v2 application is unstable or the defect is not contained, I stop the rollout and restore the previous known-good image.

---

## 5. What would you monitor during a canary release?

Monitor technical signals and business outcomes.

### Strong answer

> I would compare v1 and v2 error rates, p95 and p99 latency, availability, request volume, restarts, CPU, memory, database errors and dependency failures. For checkout I would also watch payment failures, checkout success, order creation and cart abandonment. A healthy container alone is not enough.

---

## 6. A checkout feature is deployed, but payment failures increase. What do you do?

Reduce user impact first, then investigate.

### Strong answer

> I would determine scope and severity using logs, traces and business KPIs. If the feature is behind a flag, I would disable it immediately and confirm that the previous path works. If the issue affects the broader release, I would stop the canary or roll back to the previous image. Then I would investigate the API, payment provider, configuration, database writes and recent changes before validating a fix in staging.

---

## 7. Would you always automate rollback?

No. Automate clear and severe signals, but retain human judgment for ambiguous failures.

### Strong answer

> For a well-understood signal such as a sustained 5xx increase above a tested threshold, automatic rollback can reduce recovery time. I would not blindly automate failures involving data integrity, destructive migrations or external dependencies because restoring the old image may not solve those problems.

---

## 8. How do you avoid rollback flapping?

Use thresholds, observation windows, minimum sample sizes and cooldowns.

### Strong answer

> I would require the threshold to persist for a defined period and apply it only after enough traffic has been observed. I would add a cooldown and an escalation path so a temporary spike does not cause repeated rollback and redeploy cycles.

---

## 9. What is the database rollback trap?

The previous application version may no longer work with the current schema.

### Strong answer

> If v2 removes `users.name` and v1 still requires it, restoring v1 can create a second outage. Application rollback and database rollback are separate concerns, so I use expand-and-contract migrations and keep old and new versions compatible during the transition.

---

## 10. Explain expand-and-contract migrations.

Add compatible schema first, migrate application behavior, then remove the old schema later.

### Strong answer

> I first expand the schema by adding the new field while retaining the old one. I deploy code that can use both, backfill data, switch reads and writes to the new field, verify that old application versions are no longer required, and only then contract the schema by removing the old field.

---

## 11. What would you do if v2 has a higher error rate during canary?

Stop increasing exposure and preserve the known-good version.

### Strong answer

> I would hold or reduce v2 traffic, keep the majority on v1, inspect version-specific logs and metrics, and roll back or isolate v2 if the evidence shows user impact. I would not increase exposure simply because the deployment command completed successfully.

---

## 12. Why is a passing health check insufficient?

A health check may not exercise the affected business path.

### Strong answer

> `/health` may prove that the process is running and dependencies are reachable, but it may not test payment authorization, order creation or data correctness. I combine health checks with smoke tests, release metrics and business KPIs.

---

## 13. What is the smallest safe recovery mechanism?

The smallest action that reliably reduces user impact.

### Strong answer

> If one flagged feature is broken, I disable that feature. If the release is broadly unstable, I restore the previous image. If infrastructure or configuration is the cause, I recover that layer. Choosing the smallest effective action limits collateral impact and speeds recovery.

---

## 14. What makes a feature flag production-ready?

It needs ownership, secure control, observability and a removal plan.

### Strong answer

> I want a safe default, authorization around changes, an audit trail, targeting rules, metrics by flag state, a defined failure mode and an expiry or cleanup date. A flag without ownership becomes hidden permanent complexity.

---

## 15. How would you answer: "How do you safely release a new MERN checkout feature?"

### Sample answer

> I would build and scan an immutable image, deploy it with the checkout flag disabled and run health and smoke tests. I would enable it for internal users first, then use canary or percentage-based rollout while comparing technical metrics and checkout KPIs with the baseline. If checkout failures increase, I would disable the flag immediately. If the broader application is unhealthy, I would stop the rollout or restore the previous image. I would use backward-compatible database migrations so old and new versions can coexist and I would preserve the previous artifact for rollback.

---

## 16. When should a human approve a release?

When the risk, impact or evidence cannot be represented safely by automation alone.

### Strong answer

> Automated gates are useful for repeatable signals, but a human should review high-risk schema changes, major payment changes, data migrations, unusual business metric movement and any release with unresolved alerts. Approval should be based on evidence, not just a deployment status.

---

## 17. What should happen before increasing canary exposure?

The current stage must meet predefined release criteria.

### Strong answer

> I would require a sufficient observation window, healthy error and latency comparisons, no critical alerts, stable resource and database behavior, and acceptable business KPIs. If any critical criterion fails, I hold or reverse the rollout.

---

## 18. What does release safety mean?

It means limiting user impact and shortening recovery time when a release behaves unexpectedly.

### Concise answer

> Release safety combines immutable artifacts, progressive exposure, feature flags, observability, backward-compatible migrations and a tested rollback path. It is a decision system, not just a deployment command.
