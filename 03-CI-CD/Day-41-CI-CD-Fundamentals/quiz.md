# Day 41 - Quiz: CI/CD Fundamentals

## Questions

### Q1
What does Continuous Integration primarily provide?

A. Automated validation of integrated code changes  
B. Database backups  
C. DNS management  
D. Cloud billing

### Q2
What is Continuous Deployment?

A. Manually preparing releases  
B. Automatically deploying passing changes to production  
C. Writing Dockerfiles  
D. Running MongoDB

### Q3
Which should normally happen before production deployment?

A. Tests and required quality/security gates  
B. Delete the previous image  
C. Remove the Git repository  
D. Disable health checks

### Q4
What is a CI artifact?

A. A produced build output that can be consumed later  
B. A Docker network  
C. A Git password  
D. A database collection

### Q5
Why tag a Docker image with a Git commit SHA?

A. For traceability and reproducibility  
B. To make it mutable  
C. To disable Docker  
D. To expose MongoDB

### Q6
Where should CI/CD secrets generally be stored?

A. Git source code  
B. Dockerfile  
C. CI/CD secret store or appropriate external secret manager  
D. README

### Q7
What should happen if unit tests fail?

A. Deploy anyway  
B. Skip the failed tests  
C. Stop or fail the appropriate downstream pipeline stages  
D. Delete production

### Q8
What is the main difference between Continuous Delivery and Continuous Deployment?

A. Delivery keeps software deployable; Deployment automatically releases it  
B. They are completely unrelated  
C. Deployment is only about Docker  
D. Delivery means database backups

### Q9
Why is `npm ci` commonly preferred in CI when a lockfile exists?

A. It performs a clean, lockfile-based installation  
B. It removes all tests  
C. It publishes to production  
D. It exposes environment variables

### Q10
What is a quality gate?

A. A condition that must pass before the pipeline continues  
B. A Docker volume  
C. A database index  
D. A network port

### Q11
Which image reference provides the strongest release traceability?

A. `mern-api:latest` only  
B. `mern-api:abc123` linked to the commit  
C. `mern-api:random`  
D. An untagged local image

### Q12
What should a pipeline do with production credentials on an untrusted fork pull request?

A. Expose all credentials to the test job  
B. Commit credentials to the branch  
C. Limit permissions and withhold production credentials  
D. Print them for debugging

### Q13
What does a health check validate?

A. That the application responds according to an expected health contract  
B. That the Git repository is public  
C. That the image tag is mutable  
D. That MongoDB is exposed to the Internet

### Q14
Which test is usually the fastest and most isolated?

A. Unit test  
B. End-to-end test  
C. Production smoke test  
D. Browser performance test

### Q15
What is the safest artifact promotion model?

A. Rebuild independently in every environment  
B. Build once, test and scan it, then promote the same artifact  
C. Always deploy untagged images  
D. Build directly on the production server

## Answer Key

```text
1  -> A
2  -> B
3  -> A
4  -> A
5  -> A
6  -> C
7  -> C
8  -> A
9  -> A
10 -> A
11 -> B
12 -> C
13 -> A
14 -> A
15 -> B
```

## Score

- 14-15: Excellent CI/CD understanding
- 11-13: Good; review gates, artifacts, and secrets
- 8-10: Revisit CI/CD stages and test levels
- 0-7: Repeat the lesson and complete the practical assignment

## Reflection Questions

1. Why should tests run before Docker image publication?
2. What is the difference between an artifact and source code?
3. Why is `latest` alone a weak production tag?
4. How should a pipeline handle a failed security scan?
5. Why should pull requests from forks have restricted permissions?
6. What is the difference between a liveness check and a readiness check?
7. What source revision produced the image running in production?
8. Which stages would you keep manual, if any, before production?
