# Day 54 Quiz: Load Balancing and High Availability

## Questions

### Q1. What is the primary purpose of a load balancer?

A. Distribute traffic across application instances  
B. Store MongoDB data  
C. Build Docker images  
D. Run Git

### Q2. Why is horizontal scaling useful?

A. It adds more application instances  
B. It always eliminates database bottlenecks  
C. It removes monitoring  
D. It replaces testing

### Q3. What does a health-aware load balancer do?

A. Avoids routing traffic to unhealthy instances  
B. Deletes unhealthy servers  
C. Rebuilds Docker images  
D. Changes application code

### Q4. Why is a stateless API easier to scale?

A. Any healthy instance can handle requests  
B. It requires one server  
C. It stores all state in memory  
D. It avoids databases

### Q5. What is a sticky session?

A. Routing a client repeatedly to the same backend instance  
B. A MongoDB transaction  
C. A Docker volume  
D. A Git branch

### Q6. What is a major drawback of in-memory sessions?

A. They are lost when the instance disappears  
B. They improve availability  
C. They remove the need for load balancing  
D. They are stored in MongoDB

### Q7. Why can scaling API instances make database performance worse?

A. More instances can create more database load and connections  
B. APIs cannot scale  
C. MongoDB automatically deletes data  
D. Load balancers stop working

### Q8. What should happen if one backend becomes unhealthy?

A. Continue sending it traffic  
B. Remove it from service until healthy  
C. Increase traffic to it  
D. Delete all other instances

### Q9. Which is useful during a canary release?

A. Weighted traffic distribution  
B. Disabling health checks  
C. One API instance only  
D. No metrics

### Q10. What does high availability primarily aim to achieve?

A. Continued service despite certain component failures  
B. Zero bugs  
C. Zero cost  
D. No deployments

### Q11. What is the safest first response to one bad replica?

A. Isolate it from new traffic and investigate it  
B. Restart every replica immediately  
C. Send it more traffic  
D. Disable monitoring

### Q12. What should happen before a new API version receives canary traffic?

A. It should pass health, readiness and smoke checks  
B. It should receive all production traffic  
C. The old version must be deleted first  
D. MongoDB monitoring should be disabled

## Answer Key

```text
1 -> A
2 -> A
3 -> A
4 -> A
5 -> A
6 -> A
7 -> A
8 -> B
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
| 6-8 | Review health-aware routing and statelessness |
| Below 6 | Revisit today's lesson |
