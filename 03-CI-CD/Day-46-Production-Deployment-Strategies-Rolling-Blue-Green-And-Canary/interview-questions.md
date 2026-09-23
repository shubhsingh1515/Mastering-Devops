# Day 46 - Interview Questions: Deployment Strategies

## 1. What is a rolling deployment?

A rolling deployment replaces old application instances gradually with new ones while the service remains available.

### Strong answer

> A rolling deployment updates one instance at a time or a small set at a time. This reduces downtime because the remaining healthy instances keep serving traffic while the new version is being introduced.

---

## 2. What is the difference between rolling and blue-green deployment?

Rolling replaces instances gradually, while blue-green runs two full environments and shifts traffic from one to the other.

### Strong answer

> Rolling is incremental and resource-efficient, while blue-green provides a cleaner switch by having a full parallel environment ready. Blue-green is often faster to roll back, but it costs more in infrastructure.

---

## 3. Why is blue-green useful for rollback?

It keeps the previous version running in a separate environment and allows traffic to switch back immediately.

### Strong answer

> Because the older environment stays intact, rollback is often just a traffic switch. This reduces the risk of keeping users on a broken version while the issue is investigated.

---

## 4. What is a canary deployment?

A canary deployment exposes the new version to only a small percentage of traffic and gradually increases exposure if metrics remain healthy.

### Strong answer

> Canary is useful when a release is risky. You send a small share of real traffic to v2, observe metrics, and expand only if there is no strong evidence of failure.

---

## 5. Why are health checks essential before routing traffic to a new container?

Because a running container can still be unhealthy or have a broken dependency path.

### Strong answer

> The container might be up, but the app may fail on a database connection, config problem, or internal dependency. Health checks ensure the new instance is genuinely ready to serve requests.

---

## 6. What do you monitor during a deployment?

You monitor error rate, latency, CPU and memory, health endpoint responses, and business-critical paths.

### Strong answer

> I would watch application health, HTTP status codes, latency, resource saturation, and logs. A deployment is not successful based only on container start status; it is successful only when the application remains healthy under live traffic.

---

## 7. Why can a rolling deployment fail even when instances are healthy individually?

Because the application versions may be incompatible with the database schema or other shared services.

### Strong answer

> During a rolling update, v1 and v2 may coexist for a period. If the database schema or API contract changes in a backward-incompatible way, one version can break while the other still survives. This is why migration strategy matters.

---

## 8. What is the safest way to handle database schema changes during a deployment?

Use backward-compatible, expand-and-contract migrations.

### Strong answer

> I prefer to add new fields or tables first, deploy compatible versions, migrate the data, then remove the old dependency only after all services are using the new version. This reduces the chance of a production outage during rollout.

---

## 9. What would you do if v2 suddenly has a much higher error rate than v1 during canary release?

Stop the rollout, hold traffic at the current level, investigate, and rollback if needed.

### Strong answer

> I would stop traffic increase immediately, inspect logs and monitoring, compare v1 and v2 behavior, and if the issue is significant, revert to v1. A canary release should be reversible if the evidence says the new version is unsafe.

---

## 10. When would you choose rolling deployment in production?

When low cost and simplicity are most important and the system can tolerate gradual replacement.

### Strong answer

> For a typical MERN app with multiple API replicas, rolling is a good default because it reduces downtime without requiring full duplicate environments. It is practical and easier to operate for a moderate workload.

---

## 11. When would you choose blue-green deployment?

When rollback speed and service isolation are highly important.

### Strong answer

> I would choose blue-green when the business cannot tolerate a slow rollback and the infrastructure budget supports running two production-like environments. It provides a clean switchback path with strong separation between versions.

---

## 12. When would you choose canary deployment?

When the release presents measurable risk, such as a big feature change or a new dependency version.

### Strong answer

> Canary is appropriate when we want to expose a small set of users to v2, watch the impact, and only expand the rollout if the metrics remain healthy. It reduces blast radius significantly.

---

## 13. Why is a deployment not considered successful just because the new container started?

Because the app may still be unhealthy or unable to process requests correctly.

### Strong answer

> Starting a container only proves the process launched. The real success conditions are: health checks pass, smoke tests pass, error rates remain acceptable, and traffic can be processed safely.

---

## 14. What is the relationship between deployment strategy and rollback strategy?

They are tightly coupled. A strong deployment strategy includes a clearly defined rollback path.

### Strong answer

> A deployment without a rollback plan is incomplete. The rollback plan should be considered before the rollout begins, including which version is kept live, how traffic is switched, and how database compatibility is managed.

---

## 15. How would you answer this DevOps interview question?

> "How would you achieve zero-downtime deployment for a Dockerized MERN app?"

### Sample answer

> I would run multiple API containers behind a reverse proxy or load balancer and deploy the new version gradually instead of taking the whole service down. Before routing traffic to a new instance, I would require health checks and smoke tests to pass. Depending on the risk profile, I would use rolling, blue-green, or canary deployment. I would monitor error rate, latency, and health metrics during rollout, keep the previous image available for rollback, and ensure database migrations are backward compatible during the transition.

---

## 16. What is the major cost of blue-green deployment?

It requires duplicate production infrastructure, which increases cost and complexity.

### Strong answer

> Blue-green gives fast rollback and isolation, but you pay for running two full environments at the same time. That is a real operational trade-off and must be justified by availability or business risk.

---

## 17. Why is canary safer than a full cutover?

It limits blast radius by exposing the new version to a small segment of real users first.

### Strong answer

> Instead of flipping all users to v2 instantly, we expose only a small subset. If the app underperforms, we can stop the rollout before the impact spreads broadly.

---

## 18. What should happen if health checks fail during a deployment?

The rollout should stop, and the system should stay on the known-good version.

### Strong answer

> A failed health check is a stop signal. It means the environment is not ready for traffic, and the release should be paused or rolled back. This prevents a broken instance from reaching users.

---

## 19. What is the most important principle behind deployment strategies?

Reduce the risk of user impact while ensuring the old version remains recoverable.

### Strong answer

> The best deployment strategy is the one that protects users, keeps observability active, and provides a clear rollback path. The strategy should support business continuity, not just technical novelty.

---

## 20. How do you explain this in a short interview answer?

### Concise version

> I use deployment strategies to reduce downtime and risk. Rolling is the simplest and most common, blue-green is great for fast rollback, and canary is best when the change is high-risk. The most important part is health checks, monitoring, and a clear rollback plan, especially for database compatibility.
