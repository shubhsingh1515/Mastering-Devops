# Day 56 Quiz: MERN Performance

## Questions

### Q1
What should you do before optimizing performance?

A. Measure the system  
B. Add servers  
C. Add caching everywhere  
D. Increase database size

### Q2
What does a CDN primarily help with?

A. Delivering cached content closer to users  
B. Replacing MongoDB  
C. Running Node.js  
D. Managing Git

### Q3
What is a cache hit?

A. Data is found in the cache  
B. Database connection fails  
C. API crashes  
D. Query is deleted

### Q4
What is a major cost of database indexes?

A. Write, maintenance and storage overhead  
B. They always make writes faster  
C. They eliminate databases  
D. They remove network latency

### Q5
What does `explain()` help investigate?

A. Database query execution behavior  
B. Git history  
C. Docker networking only  
D. React rendering only

### Q6
Why is returning 500,000 records from an API generally bad?

A. It increases database, network, memory and frontend work  
B. It improves caching  
C. It guarantees high availability  
D. It reduces latency

### Q7
What is an N+1 query problem?

A. One initial query followed by many unnecessary related queries  
B. One Docker container  
C. One API server  
D. One cache entry

### Q8
Why is caching not always the answer?

A. It introduces consistency and invalidation complexity  
B. It never improves performance  
C. It replaces monitoring  
D. It eliminates databases

### Q9
If API CPU is 30 percent but MongoDB query latency is very high, what should you investigate?

A. Database queries, indexes and capacity  
B. Only API CPU  
C. Git branches  
D. Frontend colors

### Q10
What does p99 latency describe?

A. A high percentile of request latency showing tail behavior  
B. CPU utilization  
C. Cache size  
D. Database storage

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
```

## Score Guide

```text
9-10 -> Excellent
7-8  -> Good
5-6  -> Review performance concepts
0-4  -> Revisit today's lesson
```

## Self-Review Prompts

After checking your score, explain these without looking at the lesson:

1. Why can normal API CPU coexist with high request latency?
2. What evidence would justify adding a MongoDB index?
3. What freshness and invalidation rule would you choose for a product catalog?
4. When is cursor pagination preferable to offset pagination?
5. How would you prove that an optimization improved production behavior?
