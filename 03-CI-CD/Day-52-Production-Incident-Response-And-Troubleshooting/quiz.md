# Day 52 Quiz: Production Incident Response and Troubleshooting

## Questions

### Q1. What is the first priority during a production incident?

A. Write the RCA  
B. Reduce user impact  
C. Rewrite the application  
D. Delete logs

### Q2. What is triage?

A. Determining scope, severity and impact  
B. Rebuilding Docker  
C. Creating Git branches  
D. Deleting containers

### Q3. When is rollback especially appropriate?

A. A new deployment clearly caused a serious issue and rollback is safe  
B. Every time CPU increases  
C. Whenever a log appears  
D. Before investigating anything

### Q4. What is a root cause?

A. The underlying reason a failure occurred  
B. Always the last error message  
C. Always a server restart  
D. A dashboard

### Q5. Why document an incident timeline?

A. To understand sequence, decisions and recovery  
B. To replace monitoring  
C. To reduce Docker image size  
D. To delete logs

### Q6. What should happen after rollback?

A. Assume success  
B. Verify metrics, health, logs and user-facing behavior  
C. Immediately deploy again  
D. Disable monitoring

### Q7. What is fix forward?

A. Deploying a corrected version rather than returning to the old version  
B. Restarting MongoDB  
C. Scaling horizontally  
D. Deleting Git history

### Q8. Why are backward-compatible migrations useful?

A. They make mixed application versions and safer rollback easier  
B. They eliminate databases  
C. They prevent logging  
D. They replace Docker

### Q9. What is a good RCA outcome?

A. Blame an individual  
B. Identify system improvements that prevent recurrence  
C. Delete incident data  
D. Ignore the incident

### Q10. What proves recovery?

A. Docker says `running`  
B. User-facing service and relevant health and business metrics return to acceptable levels  
C. Git says merged  
D. CI says passed

### Q11. What is a safe first mitigation when a broken checkout flow is isolated behind a feature flag?

A. Disable the feature flag and verify the fallback flow  
B. Delete all production logs  
C. Restart every server randomly  
D. Disable monitoring

### Q12. What should be preserved before changing production during an incident?

A. Relevant logs, metrics, deployment metadata and request IDs  
B. Only the last Git commit  
C. Nothing; evidence is unnecessary  
D. User passwords

## Answer Key

```text
1 -> B
2 -> A
3 -> A
4 -> A
5 -> A
6 -> B
7 -> A
8 -> A
9 -> B
10 -> B
11 -> A
12 -> A
```

## Scoring

| Score | Result |
|---|---|
| 11-12 | Excellent |
| 9-10 | Good |
| 6-8 | Review incident response |
| Below 6 | Revisit today's lesson |
