# Day 51 Quiz: Centralized Logging and Alerting

## Questions

### Q1. What is centralized logging?

A. Storing logs from multiple components in a central system  
B. Deleting logs after every deployment  
C. Logging only from MongoDB  
D. Logging only during development

### Q2. What is the primary purpose of an alert?

A. Provide an actionable notification about an important condition  
B. Store every log  
C. Replace dashboards  
D. Replace testing

### Q3. Which is generally the better alert?

A. CPU changed  
B. Production 5xx rate is greater than 5% for 5 minutes  
C. One debug message occurred  
D. One successful request occurred

### Q4. What is alert fatigue?

A. Too many low-value alerts causing people to ignore notifications  
B. A CPU problem  
C. A Docker failure  
D. A database index

### Q5. Why does log volume matter?

A. It affects storage, search cost, performance and noise  
B. It improves security automatically  
C. It removes the need for monitoring  
D. It eliminates incidents

### Q6. Which event should generally be retained more aggressively?

A. Routine successful request  
B. Critical security event  
C. Debug message  
D. Repeated health check

### Q7. What should you do after receiving a production alert?

A. Immediately restart every server  
B. Establish scope and investigate using metrics and logs  
C. Delete the logs  
D. Disable monitoring

### Q8. Why can a business metric be a valuable alert?

A. Technical infrastructure can be healthy while users cannot complete important workflows  
B. Business metrics replace all infrastructure metrics  
C. Business metrics build Docker images  
D. Business metrics configure DNS

### Q9. Why should logs be protected?

A. They can contain sensitive operational or user information  
B. Logs are always public  
C. Logs are executable  
D. Logs replace secrets

### Q10. Should every individual application error generate an alert?

A. Yes  
B. No; alert on meaningful patterns or conditions requiring action

## Answer Key

```text
1 -> A
2 -> A
3 -> B
4 -> A
5 -> A
6 -> B
7 -> B
8 -> A
9 -> A
10 -> B
```

## Scoring

| Score | Result |
|---|---|
| 9-10 | Excellent |
| 7-8 | Good |
| 5-6 | Review logging and alerting |
| Below 5 | Revisit today's lesson |