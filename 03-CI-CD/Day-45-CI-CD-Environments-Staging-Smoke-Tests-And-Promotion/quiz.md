# Day 45 - Quiz: Staging, Smoke Tests, and Promotion

## Questions

### Q1

Why does staging exist?

A. To store Git repositories  
B. To validate deployments in a production-like environment  
C. To replace CI  
D. To store Docker credentials

### Q2

Should production normally use a rebuilt image?

A. Yes  
B. No, promote the validated artifact

### Q3

What is a smoke test?

A. Full performance testing  
B. A lightweight test verifying basic deployment functionality  
C. A Docker build  
D. A database backup

### Q4

If staging smoke tests fail, what should happen?

A. Deploy to production anyway  
B. Stop promotion and investigate  
C. Delete the registry  
D. Disable health checks

### Q5

What should differ between staging and production?

A. Necessarily the application image  
B. Environment-specific configuration and infrastructure  
C. Source code  
D. Dockerfile contents for every release

### Q6

Why isolate staging databases from production databases?

A. To prevent test workloads from affecting real production data  
B. To make Docker slower  
C. To eliminate backups  
D. To disable MongoDB

### Q7

Deployment means:

A. Putting an artifact into an environment  
B. Writing source code  
C. Deleting an image  
D. Running Git

### Q8

Promotion means:

A. Advancing a validated artifact toward another environment  
B. Rebuilding source code  
C. Changing MongoDB schemas  
D. Restarting Nginx

### Q9

A staging API returns 502. What should you investigate first?

A. Nginx/upstream connectivity and API health  
B. Delete production  
C. Change React CSS  
D. Rebuild MongoDB

### Q10

Why is the same Docker image useful across staging and production?

A. It improves reproducibility and confidence that production uses the validated artifact  
B. It removes configuration  
C. It eliminates testing  
D. It forces identical databases

---

## Answer Key

```text
1 -> B
2 -> B
3 -> B
4 -> B
5 -> B
6 -> A
7 -> A
8 -> A
9 -> A
10 -> A
```

---

## Score Interpretation

```text
9-10  Excellent
7-8   Good
5-6   Review staging/promotion concepts
<5    Revisit today's lesson and practical exercise
```

---

## Reflection Questions

1. Why does staging exist if CI already has tests?
2. What is the difference between deployment and promotion?
3. Why should a smoke test be lightweight?
4. Why should production not use the same database credentials as staging?
5. Why is `localhost` often wrong inside a Docker container?
6. Why does promoting the same image simplify rollback?
7. What configuration values are normally environment-specific?
8. What should happen if the staging health endpoint fails?
9. Why is a failed release in staging safer than a failed release in production?
10. How would you explain the promotion process in one interview answer?

---

## Short Answer Challenge

Write 3-5 lines on each of these:

- Why staging is necessary
- Why the artifact should remain the same
- Why smoke tests matter
- Why secrets must be isolated

A good answer should sound like release engineering thinking, not just memorized definitions.
