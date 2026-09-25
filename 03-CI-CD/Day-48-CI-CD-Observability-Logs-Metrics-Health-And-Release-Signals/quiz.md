# Day 48 - Quiz: CI/CD Observability

## Questions

### Q1

What do logs primarily tell you?

A. What happened  
B. How much traffic exists  
C. Infrastructure cost  
D. Git history

### Q2

What do metrics primarily provide?

A. Quantitative measurements over time  
B. Source code  
C. Docker images  
D. Secrets

### Q3

What does p95 latency represent?

A. Approximately 95% of requests are at or below that latency  
B. The fastest 5%  
C. CPU usage  
D. Error count

### Q4

Why are business metrics useful during deployments?

A. Infrastructure can look healthy while user functionality is broken  
B. They replace logs  
C. They replace testing  
D. They eliminate monitoring

### Q5

A container is running but `/health` returns HTTP 500. Is it ready?

A. Yes  
B. No  
C. Only when CPU is low  
D. Always

### Q6

What is a correlation or request ID useful for?

A. Connecting logs belonging to one request across services  
B. Building Docker images  
C. Creating Git branches  
D. Encrypting MongoDB

### Q7

After deployment, 5xx increases from 0.2% to 6%. What should you do?

A. Ignore it  
B. Investigate and potentially stop or roll back the release  
C. Delete the database  
D. Increase traffic

### Q8

Why compare metrics to a baseline?

A. To understand whether the release changed system behavior  
B. To avoid testing  
C. To remove logs  
D. To increase Docker image size

### Q9

Which is the best release-monitoring approach?

A. CPU only  
B. HTTP status only  
C. Multiple technical and business signals  
D. Container count only

### Q10

What should guide automated rollback?

A. Carefully defined, meaningful failure signals  
B. Random timing  
C. Developer preference only  
D. Docker image size

### Q11

Which endpoint should usually include required dependency checks?

A. Readiness  
B. Git status  
C. Docker registry  
D. Build endpoint

### Q12

What can a trace help identify?

A. Where a request spent time across services  
B. The author of a commit  
C. The size of a Docker image  
D. The number of Git branches

## Answer Key

```text
1 -> A
2 -> A
3 -> A
4 -> A
5 -> B
6 -> A
7 -> B
8 -> A
9 -> C
10 -> A
11 -> A
12 -> A
```

## Score

```text
11-12  Excellent
9-10   Good
7-8    Review observability
0-6    Revisit the lesson and repeat the incident drill
```

## Reflection Questions

1. Why can CPU and memory look normal while checkout is failing?
2. Why is p95 often more useful than average latency?
3. What information should never be written to application logs?
4. What makes readiness different from liveness?
5. Why should a release dashboard include deployment markers?
6. What is the smallest safe action when only a flagged feature is broken?
7. Why are minimum sample sizes useful for automated gates?
8. Which evidence would you preserve during a release incident?
