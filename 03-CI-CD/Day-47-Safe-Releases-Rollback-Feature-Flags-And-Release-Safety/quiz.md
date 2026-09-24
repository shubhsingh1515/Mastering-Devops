# Day 47 - Quiz: Safe Releases and Feature Flags

## Questions

### Q1

What does a feature flag provide?

A. Runtime control over application functionality  
B. Automatic Docker image creation  
C. A database backup  
D. DNS management

### Q2

What is a major advantage of feature flags?

A. They allow feature-level rollback without necessarily redeploying  
B. They eliminate testing  
C. They guarantee zero bugs  
D. They remove the need for monitoring

### Q3

What is an application rollback?

A. Returning traffic or deployment to a previous application artifact  
B. Turning off one feature while keeping the image  
C. Restarting MongoDB  
D. Deleting the Git repository

### Q4

Why can a database migration complicate an application rollback?

A. Older application versions may be incompatible with the new schema  
B. Docker cannot access databases  
C. Git loses the previous commit  
D. Nginx cannot route HTTP traffic

### Q5

Which metric is especially valuable for a checkout feature?

A. Checkout success rate  
B. Docker image size only  
C. Git commit count  
D. CPU model

### Q6

What is progressive rollout?

A. Gradually increasing exposure to a new release  
B. Deploying all traffic at once  
C. Deleting the previous image  
D. Rebuilding every server after each request

### Q7

Should every increase in HTTP errors trigger an automatic rollback?

A. Yes, regardless of context  
B. No; assess severity, user impact, duration and context  
C. Yes, but only on weekends  
D. No monitoring is needed

### Q8

What is the smallest recovery action when only a flagged feature is broken?

A. Disable the feature  
B. Destroy the database  
C. Rebuild every image  
D. Delete the container registry

### Q9

What does deployment not equal release mean?

A. Code can be deployed while functionality stays disabled or is released gradually  
B. Deployments are unnecessary  
C. Releases never require code  
D. Docker replaces source control

### Q10

Which is a safe database migration sequence for rollback-friendly releases?

A. Remove the old field first, then deploy code  
B. Expand, deploy compatible code, backfill, switch behavior and contract later  
C. Drop the database and restore it after deployment  
D. Change the schema and application in unrelated order

### Q11

At 10% traffic, v2 has a 6.7% 5xx rate and much higher latency than v1. What should happen?

A. Increase traffic to 50% immediately  
B. Stop the rollout, investigate and roll back or isolate v2 if needed  
C. Disable all monitoring  
D. Delete the known-good v1 image

### Q12

Why can a service be healthy while a business feature is failing?

A. Infrastructure checks do not necessarily validate business outcomes  
B. Health checks always test payments  
C. CPU metrics contain order data  
D. A 200 response guarantees successful checkout

---

## Answer Key

```text
1 -> A
2 -> A
3 -> A
4 -> A
5 -> A
6 -> A
7 -> B
8 -> A
9 -> A
10 -> B
11 -> B
12 -> A
```

---

## Score

```text
11-12  Excellent
9-10   Good
7-8    Review release safety
0-6    Revisit the lesson and repeat the incident drills
```

---

## Reflection Questions

1. Why should the default value of a high-risk feature flag usually be safe and disabled?
2. How does a feature rollback differ operationally from an image rollback?
3. Why should a previous production image remain available in the registry?
4. Which signals would you combine before increasing canary exposure?
5. Why are business metrics important during a checkout release?
6. What is the danger of a destructive database migration during a rollout?
7. How does expand-and-contract migration support rollback?
8. When is automated rollback useful?
9. Why can automated rollback flap during temporary metric spikes?
10. What evidence should be preserved during a production incident?
