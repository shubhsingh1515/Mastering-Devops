# Day 37 - Quiz: Docker Production Deployment Strategies & Rollback

## Questions

### Q1
Which is generally the strongest production image reference?

A. `latest`  
B. Container name  
C. Specific version/commit tag or immutable digest  
D. Random generated name

### Q2
What does “build once, deploy many” mean?

A. Build the same source separately on every server  
B. Build one tested artifact and promote it through environments  
C. Never test the image  
D. Rebuild production manually

### Q3
Why are previous image versions retained?

A. To increase disk usage  
B. To enable traceability and rollback  
C. To disable networking  
D. To replace health checks

### Q4
A new API container is Up but `/health` fails. Should production traffic be considered safe?

A. Yes  
B. No  
C. Only if Nginx is running  
D. Only if MongoDB is running

### Q5
Why can database migrations complicate rollback?

A. Databases cannot be accessed by Docker  
B. New schema changes may be incompatible with the previous application version  
C. Docker automatically deletes databases  
D. Nginx controls MongoDB schema

### Q6
What is a blue-green deployment?

A. Two database volumes  
B. Two application environments/versions where traffic can be switched between them  
C. Two Docker networks with random names  
D. A Docker image optimization method

### Q7
What is a canary deployment?

A. Deploying a new version to a small portion of traffic before broader rollout  
B. Deploying only to development  
C. Deleting the previous version  
D. Running MongoDB twice

### Q8
Which is the best rollback approach?

A. Rebuild from memory  
B. Deploy a previously tested known-good image  
C. Restart the failed container repeatedly  
D. Delete production data

### Q9
Which deployment sequence is safest?

A. Deploy → hope → investigate  
B. Build → test → deploy → health check → monitor → rollback if needed  
C. Delete old → build on production → deploy  
D. Restart → restart → restart

### Q10
What is a major advantage of immutable image references?

A. They make containers permanent  
B. They provide reproducibility and precise release identification  
C. They remove the need for testing  
D. They automatically back up databases

## Answer Key

```text
1 -> C
2 -> B
3 -> B
4 -> B
5 -> B
6 -> B
7 -> A
8 -> B
9 -> B
10 -> B
```

## Score

- 9–10: Excellent
- 7–8: Good
- 5–6: Review deployment concepts
- 0–4: Revisit Docker production fundamentals

## Practical Reflection

Write short responses to the following:

1. Why is `latest` a risky deployment target?
2. What does “build once, deploy many” mean in practice?
3. Why is `Container Up` not enough evidence of success?
4. How does health validation improve deployment safety?
5. Why must database migration strategy be considered during rollback planning?
