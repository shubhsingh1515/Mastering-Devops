# Day 58 Quiz

## Questions

1. What does an SLI represent?
   - A. A measurement of actual service behavior
   - B. A contractual promise
   - C. A deployment artifact
   - D. A database index

2. What does an SLO represent?
   - A. A measurable reliability target
   - B. A log aggregation tool
   - C. A load balancer
   - D. A container image

3. What is an SLA?
   - A. A formal service commitment
   - B. A cache invalidation strategy
   - C. A readiness probe
   - D. A source-control branch

4. A service has a 99.9% SLO. What is the allowed failure percentage?
   - A. 0.1%
   - B. 1%
   - C. 9.9%
   - D. 99.9%

5. Why do teams use error budgets?
   - A. To balance reliability and delivery risk
   - B. To eliminate every incident
   - C. To replace testing
   - D. To avoid monitoring

6. A canary has a much higher 5xx rate than stable. What is the safest first action?
   - A. Increase canary traffic
   - B. Stop the rollout and investigate or roll back
   - C. Disable alerts
   - D. Ignore the difference

7. Which is a business SLI for an e-commerce MERN application?
   - A. Checkout success rate
   - B. Number of Git commits
   - C. Dockerfile length
   - D. Number of developers

8. What does fast error-budget burn indicate?
   - A. Reliability is deteriorating rapidly
   - B. CPU is necessarily low
   - C. The cache is necessarily healthy
   - D. The deployment definitely succeeded

## Answer key

```text
1. A
2. A
3. A
4. A
5. A
6. B
7. A
8. A
```

## Scoring

- 8/8: Excellent
- 6–7/8: Good; review alerting and burn rate
- 4–5/8: Review SLI, SLO and SLA definitions
- 0–3/8: Re-read the lesson and complete the assignment
