# Day 53 Quiz: Self-Healing MERN Infrastructure

## Questions

### Q1. What does liveness primarily determine?

A. Whether the instance is alive  
B. Whether users like the UI  
C. Whether MongoDB has backups  
D. Whether CI passed

### Q2. What does readiness determine?

A. Whether the instance should receive traffic  
B. Whether Git is available  
C. Whether Docker is installed  
D. Whether logs exist

### Q3. A readiness check fails because MongoDB is temporarily unavailable. Should the application always restart?

A. Yes  
B. No

### Q4. What is a major benefit of load-balancer health checks?

A. They can remove unhealthy instances from traffic  
B. They eliminate databases  
C. They replace CI  
D. They guarantee no bugs

### Q5. What is a restart loop?

A. A process repeatedly crashes and gets restarted  
B. A successful deployment  
C. A database backup  
D. A rolling deployment

### Q6. Why is graceful shutdown useful?

A. It allows active requests to finish before termination  
B. It deletes logs  
C. It removes health checks  
D. It rebuilds Docker images

### Q7. Which is a good production architecture?

A. One API instance with no health check  
B. Multiple instances with readiness checks and load balancing  
C. One server with manual SSH recovery only  
D. Automatic restarts with no monitoring

### Q8. What should happen to an instance whose readiness check fails?

A. It should normally stop receiving new traffic  
B. It must always be destroyed  
C. It must receive more traffic  
D. CI should rebuild Git

### Q9. Why can automatic restarts fail to solve an external dependency outage?

A. Restarting does not restore the unavailable dependency  
B. Docker cannot restart  
C. Logs disappear automatically  
D. CPU becomes zero

### Q10. What is self-healing?

A. Automatically recovering from certain known failure conditions  
B. Eliminating all incidents  
C. Replacing engineers  
D. Removing monitoring

### Q11. What should a rolling deployment do before sending traffic to a new instance?

A. Wait for liveness and readiness to pass  
B. Send all traffic immediately  
C. Delete the old version first  
D. Ignore dependency checks

### Q12. What is the safest first response when one replica is unready but other replicas are healthy?

A. Remove the unready replica from traffic and investigate it  
B. Restart every replica immediately  
C. Send extra traffic to the unready replica  
D. Disable monitoring

## Answer Key

```text
1 -> A
2 -> A
3 -> B
4 -> A
5 -> A
6 -> A
7 -> B
8 -> A
9 -> A
10 -> A
11 -> A
12 -> A
```

## Scoring

| Score | Result |
|---|---|
| 11-12 | Excellent |
| 9-10 | Good |
| 6-8 | Review health and recovery design |
| Below 6 | Revisit today's lesson |
