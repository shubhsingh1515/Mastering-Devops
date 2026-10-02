# Day 55 Quiz: MERN Scaling and Capacity Planning

## Questions

### Q1. What is capacity planning?

A. Estimating workload and required system capacity  
B. Writing Dockerfiles  
C. Creating Git branches  
D. Deleting logs

### Q2. What should you identify before scaling?

A. The bottleneck  
B. The newest framework  
C. The largest server  
D. The number of Git branches

### Q3. What is horizontal scaling?

A. Adding more instances  
B. Increasing one server's CPU  
C. Deleting servers  
D. Reducing traffic

### Q4. What is a cache hit?

A. Requested data is found in the cache  
B. MongoDB crashes  
C. API restarts  
D. A queue fails

### Q5. What is a major challenge introduced by caching?

A. Cache invalidation and stale data  
B. Git conflicts  
C. Docker image size only  
D. CI configuration

### Q6. Why use background workers?

A. To move appropriate slow or asynchronous work out of the request path  
B. To replace all APIs  
C. To eliminate databases  
D. To disable monitoring

### Q7. What does a queue help absorb?

A. Traffic and workload bursts  
B. Git commits  
C. DNS records  
D. Docker layers

### Q8. Why can excessive retries be dangerous?

A. They can amplify load on a failing dependency  
B. They always improve reliability  
C. They remove latency  
D. They prevent incidents

### Q9. What does a circuit breaker help with?

A. Preventing repeated calls to an unhealthy dependency  
B. Building containers  
C. Managing Git branches  
D. Storing static assets

### Q10. What is the best way to validate scaling assumptions?

A. Load testing and production-like measurements  
B. Guessing  
C. Adding the largest server available  
D. Waiting for an outage

### Q11. What should happen when queue depth grows continuously?

A. Add worker capacity, optimize processing or apply backpressure  
B. Ignore it because queues are infinite  
C. Disable monitoring  
D. Add API replicas without measuring workers

### Q12. Why can adding API replicas worsen MongoDB performance?

A. More replicas can create more database connections and concurrent queries  
B. APIs cannot scale horizontally  
C. MongoDB deletes old data automatically  
D. Load balancers stop working

## Answer Key

```text
1 -> A
2 -> A
3 -> A
4 -> A
5 -> A
6 -> A
7 -> A
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
| 6-8 | Review bottlenecks, caching and queues |
| Below 6 | Revisit today's lesson |
