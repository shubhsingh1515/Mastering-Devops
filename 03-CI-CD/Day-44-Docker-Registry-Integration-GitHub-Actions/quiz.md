# Day 44 - Quiz: Docker Registry Integration

## Questions

### Q1

What is the main purpose of a Docker registry?

A. Run MongoDB  
B. Store and distribute container images  
C. Run GitHub Actions  
D. Compile JavaScript

### Q2

Which tag provides strong source traceability?

A. `latest`  
B. Git SHA  
C. Random  
D. `test`

### Q3

What should normally happen before pushing a production candidate image?

A. Required tests and security gates  
B. Delete the old image  
C. Disable CI  
D. Restart MongoDB

### Q4

Where should registry credentials be stored?

A. Dockerfile  
B. Git repository  
C. Protected CI/CD secrets or identity mechanism  
D. README

### Q5

What does `docker push` do?

A. Uploads an image to a registry  
B. Runs a container  
C. Builds an image  
D. Deletes an image

### Q6

What does `manifest unknown` commonly indicate during deployment?

A. The requested image or tag cannot be found  
B. MongoDB is corrupted  
C. Nginx is overloaded  
D. Git is unavailable

### Q7

Why should CD normally deploy the image produced by CI rather than rebuild it?

A. For reproducibility  
B. To make builds slower  
C. To avoid registries  
D. To remove testing

### Q8

What should you investigate if `docker push` returns access denied?

A. Authentication and authorization  
B. React CSS  
C. MongoDB schema  
D. Nginx HTML

### Q9

Which is a good production image reference?

A. `mern-api:8f31c2a`  
B. `mern-api:password123`  
C. `mern-api:unknown`  
D. `mern-api:anything`

### Q10

What role does the registry play between CI and CD?

A. Artifact handoff and storage point  
B. Database server  
C. Frontend server  
D. DNS resolver

## Answer Key

```text
1 -> B
2 -> B
3 -> A
4 -> C
5 -> A
6 -> A
7 -> A
8 -> A
9 -> A
10 -> A
```

## Score

```text
9-10  Excellent
7-8   Good
5-6   Review registry integration
0-4   Revisit today's lesson
```

## Reflection Questions

1. Why should a failed test prevent Docker publication?
2. What is the difference between a Git SHA tag and `latest`?
3. Which registry values are variables and which are secrets?
4. Why should fork pull requests have restricted permissions?
5. Why should CI scan an image before publishing it?
6. How would you investigate `denied: requested access to the resource is denied`?
7. How would you investigate `manifest unknown`?
8. Why should CD deploy the exact CI artifact?
9. How does a registry support rollback?
10. What is the difference between an image tag and an image digest?
