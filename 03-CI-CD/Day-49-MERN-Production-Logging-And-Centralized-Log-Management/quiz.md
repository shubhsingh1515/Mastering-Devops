# Day 49 Quiz: MERN Production Logging

## Questions

### Q1. What is structured logging?

A. Logging only errors  
B. Logging machine-readable fields such as JSON  
C. Logging to a text editor  
D. Logging only to Dockerfiles

### Q2. What is a request ID useful for?

A. Connecting events belonging to the same request  
B. Building Docker images  
C. Creating Git repositories  
D. Scaling MongoDB

### Q3. Which should never be logged?

A. Request latency  
B. HTTP status  
C. Passwords and tokens  
D. Application version

### Q4. Why is centralized logging useful?

A. It lets you search logs across services from one place  
B. It eliminates testing  
C. It replaces Git  
D. It creates Docker images

### Q5. Why is stdout commonly used for container logs?

A. Containers are ephemeral and external systems can collect stdout  
B. stdout is encrypted automatically  
C. stdout stores databases  
D. stdout prevents all errors

### Q6. Which log level generally represents a serious failure?

A. DEBUG  
B. INFO  
C. WARN  
D. ERROR

### Q7. What is an access log primarily concerned with?

A. HTTP requests and responses  
B. Docker image layers  
C. Git commits  
D. Database backups

### Q8. Why should you not simply log an entire user object?

A. It may contain sensitive or unnecessary data  
B. It increases Git history  
C. It breaks Docker  
D. It prevents MongoDB queries

### Q9. Which combination gives stronger observability?

A. Logs only  
B. Metrics only  
C. Logs + metrics + traces  
D. CPU only

### Q10. What should a production log ideally include?

A. Useful context and correlation information without sensitive data  
B. Every secret  
C. Entire request bodies  
D. Database passwords

## Answer Key

```text
1 -> B
2 -> A
3 -> C
4 -> A
5 -> A
6 -> D
7 -> A
8 -> A
9 -> C
10 -> A
```

## Scoring

| Score | Result |
|---|---|
| 9-10 | Excellent |
| 7-8 | Good |
| 5-6 | Review production logging |
| Below 5 | Revisit today's lesson |
