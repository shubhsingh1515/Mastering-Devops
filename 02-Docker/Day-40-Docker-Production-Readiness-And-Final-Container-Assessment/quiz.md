# Day 40 - Quiz: Docker Production Readiness & Final Container Assessment

## Questions

### Q1
Which is the strongest production practice?

A. Build directly on production  
B. Build once and promote the tested artifact  
C. Always use `latest`  
D. Store secrets in the image

### Q2
Which service should normally be the public boundary?

A. MongoDB  
B. Nginx or a reverse proxy  
C. The internal Node port  
D. The Docker volume

### Q3
Why use a private Docker network?

A. To allow trusted internal service communication without unnecessary public exposure  
B. To disable containers  
C. To store passwords  
D. To build images

### Q4
What does an image digest provide?

A. A human-readable application description  
B. A content-addressable identifier for exact image content  
C. A database password  
D. A network port

### Q5
What should you do when an image contains a Critical vulnerability?

A. Automatically ignore it  
B. Investigate, remediate if possible, rebuild, rescan, and test  
C. Delete production  
D. Disable Docker

### Q6
Why use a non-root user?

A. Least privilege  
B. Faster MongoDB queries  
C. Better CSS  
D. Automatic backups

### Q7
What is a rollback?

A. Restarting the same broken version  
B. Returning to a known-good previous release  
C. Deleting the database  
D. Rebuilding the latest image

### Q8
What should happen before accepting traffic to a newly deployed API?

A. Nothing  
B. Appropriate health and readiness validation  
C. Delete the old version immediately  
D. Expose MongoDB

### Q9
Which statement is correct?

A. A Docker volume is automatically a backup  
B. `latest` is immutable  
C. Application rollback can be complicated by incompatible database migrations  
D. Container `Up` guarantees readiness

### Q10
What is the strongest troubleshooting approach?

A. Restart everything  
B. Collect evidence, isolate the failing layer, identify root cause, then remediate  
C. Delete all containers  
D. Rebuild without investigating

### Q11
Inside an API container, what does `localhost` normally refer to?

A. The MongoDB service  
B. The API container itself  
C. The Docker host's public IP  
D. Nginx

### Q12
What is the main purpose of a readiness check?

A. Decide whether the instance can safely receive traffic  
B. Create a Docker volume  
C. Pin an image digest  
D. Replace backups

## Answer Key

```text
1 -> B
2 -> B
3 -> A
4 -> B
5 -> B
6 -> A
7 -> B
8 -> B
9 -> C
10 -> B
11 -> B
12 -> A
```

## Score

- 11-12: Docker production ready
- 9-10: Good; review the missed production topics
- 6-8: Revisit networking, health, persistence, and rollback
- 0-5: Repeat the Docker fundamentals and production-readiness lessons

## Practical Reflection

Write short answers for these questions:

1. Why does `docker compose up -d` not prove production readiness?
2. Why should only Nginx normally publish host ports?
3. What is the difference between liveness and readiness?
4. Why is a persistent volume not the same as a backup?
5. What evidence would you preserve during a 500-error incident?
6. How can a database migration make application rollback unsafe?
7. Why should staging and production use the same image digest?
8. What is the correct MongoDB hostname from the API container?
