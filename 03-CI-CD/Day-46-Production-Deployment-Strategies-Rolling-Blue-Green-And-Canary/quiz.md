# Day 46 - Quiz: Production Deployment Strategies

## Questions

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

---

## Answer Key

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

---

## Score

```text
9-10  Excellent
7-8   Good
5-6   Review deployment strategies
0-4   Revisit today's lesson
```

---

## Reflection Questions

1. Why should a release stop if health checks fail?
2. What is the difference between a rolling rollout and a blue-green switch?
3. Why can canary be safer than a full deployment?
4. What is the operational risk of mixed application versions during a rolling release?
5. Why should a migration be backward compatible during zero-downtime releases?
6. How does traffic monitoring help determine whether a deployment should continue?
7. What is the main cost trade-off of blue-green deployment?
8. Why is quick rollback valuable in production?
9. What does a smoke test prove in a deployment pipeline?
10. Why is there no single universally best deployment strategy?
