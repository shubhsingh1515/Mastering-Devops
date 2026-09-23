# DevOps Mentorship Program - Day 46

## Phase 3: CI/CD

### Production Deployment Strategies: Rolling, Blue-Green & Canary

**Level:** Intermediate -> Professional  
**Focus:** Zero-downtime deployments, release strategies, rollback, and MERN production scenarios

Yesterday you moved the pipeline through:

```text
Git
  |
  v
CI
  |
  v
Docker Build
  |
  v
Security Scan
  |
  v
Registry
  |
  v
Staging
  |
  v
Smoke Tests
  |
  v
Promotion
```

Today we solve the next production problem:

> How do we deploy a new version without unnecessarily taking the application down—and how do we recover if the release is bad?

We will cover three major strategies:

- Rolling
- Blue-Green
- Canary

---

## 1. Learning Objectives

By the end of today, you should be able to:

- Explain why deployment strategy matters.
- Describe rolling deployments.
- Describe blue-green deployments.
- Describe canary deployments.
- Compare their advantages and trade-offs.
- Explain zero-downtime deployment.
- Design a deployment strategy for a Dockerized MERN application.
- Explain rollback under each strategy.
- Handle database-migration concerns during deployment.
- Answer deployment-strategy interview questions confidently.

---

## 2. Why Deployment Strategy Matters

Suppose production currently runs:

```text
mern-api:v1
```

You need to deploy:

```text
mern-api:v2
```

The simplest approach is:

```text
Stop v1
  |
  v
Start v2
```

During the transition:

```text
Users
  |
  v
❌ No application
```

That is downtime.

A production deployment strategy tries to make the transition:

```text
v1
 |
 v2
```

safe and controlled.

The goal is not merely to run new containers. The goal is to do it in a way that keeps users served, catches failures early, and allows safe rollback.

---

## 3. Rolling Deployment

A rolling deployment replaces instances gradually.

Imagine you have four API containers:

```text
v1  v1  v1  v1
```

Instead of replacing everything at once:

```text
v2  v2  v2  v2
```

you gradually replace them:

```text
v2  v1  v1  v1
```

then:

```text
v2  v2  v1  v1
```

then:

```text
v2  v2  v2  v1
```

finally:

```text
v2  v2  v2  v2
```

Traffic continues flowing while the rollout happens.

### Why It Is Common

Rolling deployment is one of the most widely used strategies because it is:

- simple to understand
- efficient in resource use
- friendly to existing infrastructure
- natural for multi-instance services behind a load balancer

It works well in a Docker + Nginx + multiple API containers setup.

---

## 4. MERN Rolling Example

Suppose Nginx routes traffic to API instances:

```text
api-1
api-2
api-3
api-4
```

Initially:

```text
api-1 -> v1
api-2 -> v1
api-3 -> v1
api-4 -> v1
```

Deploy:

```text
api-1 -> v2
```

Health check:

```text
v2 -> healthy
```

Continue:

```text
api-2 -> v2
```

and so on.

If v2 fails:

```text
v2 ❌
```

you stop the rollout while the remaining instances still run v1.

This is a critical operational advantage: the service does not immediately become unavailable because a subset of instances is unhealthy.

---

## 5. Advantages of Rolling Deployments

Rolling deployments can provide:

- Reduced downtime
- Gradual replacement
- Lower infrastructure overhead
- Simple operational model

### Trade-offs

During rollout:

```text
v1 + v2
```

may both receive traffic.

Therefore:

> Your application versions need to coexist safely.

This is one of the biggest concerns in production deployment strategies.

---

## 6. Version Compatibility

Suppose v1 expects:

```text
users.name
```

while v2 expects:

```text
users.display_name
```

During a rolling deployment:

```text
v1 + v2
```

could simultaneously access the database.

If the schema only supports v2:

```text
v1 -> ❌
```

Your deployment fails.

This is why application releases and database migrations must be designed together.

A deployment strategy is not only about containers. It includes:

- database compatibility
- configuration changes
- API compatibility
- feature-flags safety
- migration timing

---

## 7. Expand-and-Contract Migrations

A safer pattern is:

### Step 1 — Expand

Add the new field:

```text
name
display_name
```

Both exist.

### Step 2 — Deploy compatible application

```text
v1 + v2
```

can operate.

### Step 3 — Migrate data

Populate:

```text
display_name
```

### Step 4 — Move fully to v2

### Step 5 — Contract

Only after v1 is gone do you remove:

```text
name
```

Conceptually:

```text
Old schema
    |
    v
Expand
    |
    v
Compatible schema
    |
    v
Deploy new app
    |
    v
Migrate
    |
    v
Remove old dependency
```

This is extremely important for zero-downtime deployments.

The real lesson is:

> Backward-compatible schema changes reduce release risk.

---

## 8. Blue-Green Deployment

Now consider a different approach.

You have:

```text
BLUE
v1
```

and create:

```text
GREEN
v2
```

Both environments exist.

```text
             Load Balancer / Nginx
                      |
                ┌─────┴─────┐
                ▼           ▼
             BLUE         GREEN
              v1            v2
```

Initially:

```text
100% traffic -> BLUE
```

You deploy v2 to GREEN.
Then test GREEN.
If healthy:

```text
100% traffic -> GREEN
```

BLUE remains available as the rollback environment.

This approach creates a clean separation between the active version and the new candidate version.

---

## 9. Blue-Green Rollback

Suppose:

```text
BLUE = v1
GREEN = v2
```

Traffic:

```text
Users
  |
  v
GREEN
```

Then v2 has a serious problem.
You can switch:

```text
Users
  |
  v
BLUE
```

Now:

```text
v1 -> serving traffic
v2 -> isolated
```

This can make rollback extremely fast.

Blue-green is particularly attractive when a fast rollback is a top priority.

---

## 10. Blue-Green Trade-Off

The major disadvantage:

> You may need two environments at once.

For example:

```text
BLUE:
4 API instances
4 GB RAM

GREEN:
4 API instances
4 GB RAM
```

During deployment:

```text
8 instances worth of capacity
```

So blue-green can be more expensive.

But it provides:

- fast switching
- strong isolation
- simple rollback

This makes it valuable when reliability and quick recovery matter more than infrastructure efficiency.

---

## 11. Canary Deployment

Canary deployment is more gradual.

Instead of sending:

```text
100% -> v2
```

you might send:

```text
95% -> v1
 5% -> v2
```

Then observe:

- Errors
- Latency
- CPU
- Memory
- Business metrics

If healthy:

```text
90% -> v1
10% -> v2
```

Then:

```text
50% -> v1
50% -> v2
```

Eventually:

```text
0% -> v1
100% -> v2
```

---

## 12. Why "Canary"?

The idea comes from using a small exposure group to detect danger before exposing everyone.

In software:

```text
Small traffic
     |
     v
Observe
     |
     v
Increase traffic
     |
     v
Observe
     |
     v
Full rollout
```

It is especially useful for high-risk changes.

You are exposing only a tiny group of users to the new version while the system is still under observation.

---

## 13. Canary Metrics

Suppose v2 receives 5% traffic.
You observe:

```text
v1 error rate: 0.4%

v2 error rate: 4.8%
```

That is a warning.

Even though v2 is technically "running," it is not healthy enough for broader exposure.

You can stop:

```text
v2 -> 5%
```

before exposing more users.

This is a key principle:

> Deployment decisions should be based on evidence, not merely container state.

A container can be running while the app is failing under real traffic.

---

## 14. Rolling vs Blue-Green vs Canary

| Strategy | Main Idea | Rollback | Infrastructure |
|---------|-----------|----------|----------------|
| Rolling | Replace instances gradually | Moderate | Lower |
| Blue-Green | Two environments, switch traffic | Fast | Higher |
| Canary | Gradually increase traffic | Fast if detected early | Moderate |
| Recreate | Stop old, start new | Slower | Lower |

### Mental shortcut

- Rolling → replace gradually
- Blue-Green → switch environments
- Canary → increase traffic gradually

---

## 15. Interview Question

### "Which deployment strategy would you choose?"

Don't answer:

> "Canary is always best."

Instead:

> "It depends on risk, infrastructure cost, traffic-management capabilities, rollback requirements, and application compatibility. For a small application, rolling deployment may be sufficient. For fast rollback, blue-green can be attractive. For high-risk releases where gradual exposure is valuable, canary is often preferable."

That demonstrates engineering judgment.

---

## 16. MERN Production Recommendation

For your current MERN project, a sensible progression is:

### Early production

```text
Rolling
```

because it is relatively straightforward.

### Higher availability requirement

```text
Blue-Green
```

when rapid rollback is valuable and infrastructure capacity allows it.

### Mature/high-risk production

```text
Canary
```

when you have strong:

- Observability
- Traffic control
- Automated rollback
- Metrics

Do not implement canary deployment before you can reliably observe the system.

---

## 17. Health Checks During Deployment

Deployment strategy depends heavily on health checks.

Suppose you're deploying:

```text
v2
```

The container starts.
Docker says:

```text
Running ✓
```

But:

```text
GET /health
```

returns:

```text
500
```

That instance should not receive production traffic.

Therefore:

```text
Start container
     |
     v
Health check
     |
     v
Healthy?
   /     \
  No     Yes
  |       |
  v       v
Stop   Receive
          traffic
```

This connects directly to earlier lessons around health probes, readiness, and deployment safety.

---

## 18. Smoke Tests During Promotion

After switching traffic:

```bash
curl https://api.example.com/health
```

Then test a critical endpoint:

```bash
curl https://api.example.com/api/products
```

Potentially verify:

- Login
- Product listing
- Order creation

Do not make your first production smoke test perform a destructive operation.

A smoke test should confirm the service is alive and minimally functional.

---

## 19. Rollback Strategy

Every deployment should answer:

> "How do we get back?"

For example:

```text
Current:
v2

Previous:
v1
```

If v2 fails:

```text
v2
 |
 v
Failure
 |
 v
Rollback
 |
 v
v1
 |
 v
Health check
 |
 v
Monitor
```

But rollback is not always as simple as changing an image tag.

Check:

- Application
- Database schema
- Configuration
- External APIs
- Feature flags
- Data migrations

---

## 20. Dangerous Rollback Scenario

Suppose v2 performs:

```text
Database migration:
DROP old_column
```

Then v2 fails.
You try:

```text
Rollback -> v1
```

But v1 still expects:

```text
old_column
```

Now:

```text
v1 -> ❌
```

This is why destructive migrations are dangerous.

A production-safe migration should ideally be:

> Backward compatible

during the transition.

---

## 21. Interview Scenario

### "Your deployment is 50% complete and v2 error rate suddenly increases. What do you do?"

Strong answer:

1. Stop the rollout.
2. Prevent additional traffic from moving to v2.
3. Confirm the issue using logs and metrics.
4. Compare v1 and v2 behavior.
5. Roll back traffic to v1 if impact is significant.
6. Preserve logs and deployment information.
7. Investigate root cause.
8. Fix and retest before attempting another rollout.

Do not say:

> "Wait and see."

Production users are already providing evidence.

---

## 22. Review Questions From Earlier Topics

### Q1 — Registry

Why should production deploy the exact image that passed staging?

Answer: To ensure production uses the same validated artifact rather than an independently rebuilt version.

### Q2 — CI/CD

What should happen if staging smoke tests fail?

Answer: Promotion should stop until the issue is investigated and resolved.

### Q3 — Docker Networking

Why does this usually fail inside the API container?

```text
mongodb://localhost:27017
```

Answer: localhost points to the API container, not the MongoDB service.

### Q4 — Security

Why should registry credentials have minimal permissions?

Answer: Least privilege limits the potential impact if credentials are compromised.

### Q5 — Rollback

Why can database migrations make application rollback difficult?

Answer: The newer schema may no longer be compatible with the older application version.

---

## 23. Today's Quiz

### Q1

What is a rolling deployment?

A. Replace all instances simultaneously  
B. Gradually replace old instances with new ones  
C. Deploy only to development  
D. Never deploy

### Q2

What is blue-green deployment?

A. Two environments where traffic switches between versions  
B. Two Docker networks  
C. Two databases only  
D. A Git branching strategy

### Q3

What is a canary deployment?

A. Deploying only to developers  
B. Gradually exposing a new version to a portion of traffic  
C. Rebuilding the database  
D. Deleting the old image

### Q4

Which strategy generally provides the simplest fast traffic switchback?

A. Blue-green  
B. Recreate  
C. Manual SSH  
D. None

### Q5

What is a major advantage of canary deployment?

A. Gradual exposure limits blast radius  
B. It requires no monitoring  
C. It eliminates testing  
D. It always costs less

### Q6

Why are health checks important during deployment?

A. To determine whether the new instance is actually usable  
B. To store secrets  
C. To build Docker images  
D. To create Git commits

### Q7

What is a major blue-green disadvantage?

A. Potentially higher infrastructure cost  
B. No rollback capability  
C. Requires no infrastructure  
D. It cannot run Docker

### Q8

Why should production rollback consider database migrations?

A. Application and schema versions may be incompatible  
B. Docker cannot use databases  
C. Nginx requires MongoDB  
D. Git cannot store code

### Q9

If canary v2 has a significantly higher error rate than v1, what should happen?

A. Increase traffic immediately  
B. Stop or reverse the rollout and investigate  
C. Delete v1  
D. Disable monitoring

### Q10

Which strategy is generally best for every application?

A. Canary  
B. Blue-green  
C. Rolling  
D. There is no universal best strategy

### Answer Key

```text
1 -> B
2 -> A
3 -> B
4 -> A
5 -> A
6 -> A
7 -> A
8 -> A
9 -> B
10 -> D
```

### Score

```text
9–10  Excellent
7–8   Good
5–6   Review deployment strategies
<5    Revisit today's lesson
```

---

## 24. Practical Exercise — Design Your Deployment

For your MERN application, design a rolling deployment.

Current:

```text
API:

api-1 -> v1
api-2 -> v1
api-3 -> v1
```

Desired:

```text
api-1 -> v2
api-2 -> v2
api-3 -> v2
```

### Your procedure

1. Start v2 instance.
2. Wait for health check.
3. Add v2 to traffic.
4. Remove one v1 instance.
5. Repeat.
6. Monitor errors.
7. Stop rollout if health degrades.

Document:

```text
DEPLOYMENT.md
```

with:

```text
Deployment strategy:
Rolling

Initial version:
v1

Target version:
v2

Health check:
/health

Rollback:
Return traffic to v1
```

---

## 25. Practical Failure Drill

Simulate:

```text
v1:
error rate = 0.3%

v2:
error rate = 8.2%
```

At:

```text
20% traffic
```

What should you do?

### Correct response

```text
Stop rollout
    |
    v
Keep v1 serving majority traffic
    |
    v
Investigate v2
    |
    v
If required, remove v2
    |
    v
Restore 100% v1
    |
    v
Analyze logs
```

Do not continue:

```text
20%
 |
 v
50%
 |
 v
100%
```

just because deployment automation says so.

---

## 26. Monthly Cumulative Project — Deployment Strategy Milestone

Your MERN DevOps project should now document a production deployment strategy.

Add:

```text
DEPLOYMENT.md
```

with:

- [ ] Deployment strategy selected
- [ ] Reason for selection
- [ ] Health-check procedure
- [ ] Smoke tests
- [ ] Rollback procedure
- [ ] Database compatibility strategy
- [ ] Monitoring during deployment
- [ ] Failure criteria

For the current project, use:

```text
Rolling deployment
```

as the baseline strategy.

Document future alternatives:

```text
Rolling
   |
   v
Blue-Green
   |
   v
Canary
```

and explain when you would choose each.

---

## 27. Interview Challenge

Answer aloud:

> "How would you achieve zero-downtime deployment for a Dockerized MERN application?"

A strong answer:

> "I would run multiple API instances behind a reverse proxy or load balancer and deploy the new version gradually rather than stopping the entire service. New instances would pass health checks before receiving traffic. Depending on the application's scale and requirements, I would use rolling, blue-green, or canary deployment. I would monitor error rate, latency, and health during rollout, and keep the previous image available for rollback. Database migrations would need to remain backward compatible during the transition."

That is a strong mid-level DevOps answer.

---

## 28. Day 46 Summary

Today's core model:

```text
             PRODUCTION RELEASE
                     |
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Rolling    Blue-Green   Canary
          │          │          │
      gradual     switch      gradual
      replace     traffic     traffic
```

### Rolling

```text
v1 v1 v1
 |
 v2 v1 v1
 |
 v2 v2 v1
 |
 v2 v2 v2
```

### Blue-Green

```text
BLUE  = v1
GREEN = v2

Traffic -> BLUE

        v switch

Traffic -> GREEN
```

### Canary

```text
95% -> v1
 5% -> v2

      v

90% -> v1
10% -> v2

      v

50% -> v1
50% -> v2

      v

100% -> v2
```

### Interview takeaway

The best deployment strategy is not the fanciest one.

Choose based on:

- Risk
- Traffic control
- Observability
- Infrastructure cost
- Rollback requirements
- Database compatibility

And remember:

> A deployment is not successful because the new containers started. It is successful when the new version is healthy, serving traffic correctly, and can be safely recovered if something goes wrong.

---

## 29. Next Lesson Preview

### Day 47 — Release Safety, Rollback & Feature Flags

Tomorrow we build on deployment strategies with rollback, release safety, and feature flags, including:

```text
Deployment
   |
   v
Health
   |
   v
Metrics
   |
   v
Feature flag
   |
   v
Gradual release
   |
   v
Rollback / disable feature
```

The focus will remain on MERN production incidents and DevOps interview scenarios.
