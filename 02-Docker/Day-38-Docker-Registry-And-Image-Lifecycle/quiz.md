# Day 38 - Quiz: Docker Registry & Image Lifecycle

## Questions

### Q1
What does a Docker registry primarily store?

A. Git branches  
B. Container images  
C. Database records  
D. Environment variables

### Q2
Which is more production-friendly?

A. `mern-api:latest` only  
B. `mern-api:v1.5.0` or an immutable digest  
C. Random image names  
D. Untagged images

### Q3
What is the purpose of `docker push`?

A. Start a container  
B. Upload an image to a registry  
C. Delete an image  
D. Build an image

### Q4
What does `docker pull` do?

A. Downloads an image from a registry  
B. Builds source code  
C. Creates a Git commit  
D. Starts MongoDB

### Q5
Why is build-once-deploy-many useful?

A. It makes deployments less reproducible  
B. It ensures environments receive the same tested artifact  
C. It eliminates testing  
D. It requires rebuilding production

### Q6
Which reference identifies exact image content most strongly?

A. `latest` tag  
B. Image digest  
C. `dev` tag  
D. Container name

### Q7
A production server reports `pull access denied`. What should you investigate first?

A. Registry authentication and authorization  
B. React CSS  
C. MongoDB indexes  
D. Nginx HTML

### Q8
Why retain old production images?

A. For rollback and traceability  
B. To make deployments slower  
C. Because Docker requires every image forever  
D. To disable health checks

### Q9
Which production workflow is preferable?

A. Build -> production directly  
B. Build -> test -> scan -> registry -> staging -> production  
C. Production -> build -> test  
D. Git -> MongoDB

### Q10
Why should production generally avoid rebuilding an image from source?

A. Production servers cannot run Docker  
B. It reduces reproducibility and mixes build/runtime responsibilities  
C. Docker does not support builds  
D. Registries cannot store images

## Answer Key

```text
1 -> B
2 -> B
3 -> B
4 -> A
5 -> B
6 -> B
7 -> A
8 -> A
9 -> B
10 -> B
```

## Score

- 9-10: Excellent
- 7-8: Good
- 5-6: Review registries, digests, and artifact promotion
- 0-4: Revisit the Day 38 README and repeat the command exercise

## Practical Reflection

Write short answers for these questions:

1. Why can a tag be less reliable than a digest?
2. What does build-once-deploy-many look like in your project?
3. Why is a running container not enough evidence that an image is production-ready?
4. Which permissions should a production deployment identity receive?
5. How many previous image versions should your rollback policy retain, and why?
6. Why can a database migration complicate application rollback?
