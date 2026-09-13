# Day 35 - Quiz: Production Compose and Operational Hardening

Try to answer each question before checking the key.

## Questions

### Q1

Which service should normally be the public entry point in a basic production MERN architecture?

A. MongoDB  
B. Node API  
C. Nginx  
D. Docker network

### Q2

Why should MongoDB normally remain on an internal network?

A. MongoDB does not use TCP  
B. To reduce unnecessary public exposure  
C. To make React faster  
D. To reduce image layers

### Q3

What does this provide?

```yaml
restart: on-failure
```

A. Automatic image rebuilding  
B. Automatic restart after qualifying container failures  
C. Automatic database backup  
D. Automatic HTTPS

### Q4

A container shows `Up`, but the application is not responding. What should you consider?

A. Container state is not the same as application health  
B. Docker must be reinstalled  
C. The image is necessarily corrupt  
D. The volume is necessarily deleted

### Q5

Which is preferable for service-to-service communication?

A. Hard-coded container IP  
B. Compose service name  
C. Random public IP  
D. Manually maintained host entry

### Q6

What is the difference between persistence and backup?

A. They are identical  
B. Persistence survives container replacement; backup provides recoverable copies  
C. Backup only applies to images  
D. Persistence means encryption

### Q7

What should you investigate if an API continuously restarts?

A. Only the frontend  
B. Logs, configuration, dependencies, health, and resources  
C. Delete everything immediately  
D. Change the Docker network randomly

### Q8

Why is graceful shutdown useful?

A. It makes Docker images smaller  
B. It helps applications terminate without unnecessarily dropping active work  
C. It replaces backups  
D. It exposes MongoDB

### Q9

Which command validates resolved Compose configuration?

A. `docker stats`  
B. `docker compose config`  
C. `docker history`  
D. `docker volume rm`

### Q10

Which statement is most accurate?

A. Restart policies solve application bugs  
B. A Docker volume is a complete backup strategy  
C. Container `Up` guarantees application readiness  
D. Production reliability combines health, configuration, networking, persistence, and operations

### Q11

Why should production logs normally use stdout and stderr?

A. The container runtime can capture and centralize them  
B. They automatically encrypt passwords  
C. They create database backups  
D. They expose every internal port

### Q12

What does `stop_grace_period` provide?

A. Time for a service to shut down cleanly before stronger termination  
B. Automatic database replication  
C. An additional public port  
D. Automatic image signing

### Q13

Why should a production image use a versioned tag such as `mern-api:7f81a2c`?

A. It makes rollback and incident investigation more predictable  
B. It guarantees the image has no vulnerabilities  
C. It automatically creates a volume  
D. It removes the need for health checks

### Q14

Can basic single-host Compose alone guarantee Kubernetes-style zero-downtime rolling deployment?

A. Always  
B. No; traffic management and an appropriate deployment strategy are still required  
C. Only when MongoDB is public  
D. Only when all services run as root

## Answer Key

```text
1 -> C
2 -> B
3 -> B
4 -> A
5 -> B
6 -> B
7 -> B
8 -> B
9 -> B
10 -> D
11 -> A
12 -> A
13 -> A
14 -> B
```

## Score

| Score | Result |
|---|---|
| 13-14 | Excellent: ready for production Compose operations |
| 10-12 | Good: review persistence, health, and deployment behavior |
| 7-9 | Review networking, logging, and restart policies |
| 0-6 | Revisit Docker production fundamentals and complete the lab |

## Practical Reflection

Write short answers to these questions:

1. Which services in your stack are public, and which are internal?
2. What data is persistent, and where are its backups?
3. What does your health check actually prove?
4. What happens when the API crashes repeatedly?
5. How does your application shut down after `SIGTERM`?
6. How would you diagnose a website that loads while API requests return `502`?
