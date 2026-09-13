# Day 34 - Quiz: Compose Resilience, Dependencies, and Deployment

Try to answer each question before checking the key.

## Questions

### Q1

What does this primarily express?

```yaml
depends_on:
  - mongodb
```

A. MongoDB is guaranteed healthy  
B. A service dependency or startup relationship  
C. MongoDB is publicly exposed  
D. MongoDB data is persistent

### Q2

What does a Docker health check tell you?

A. Whether the image was built successfully  
B. Whether a defined application or service condition passes  
C. Whether the container has a volume  
D. Whether Git is working

### Q3

An API container repeatedly restarts. What should you do first?

A. Delete the MongoDB volume  
B. Inspect logs and container state  
C. Change the image name  
D. Reinstall Docker

### Q4

What is a common cause of an API failing to connect to MongoDB?

A. Using `mongodb` as a service hostname  
B. Using `localhost` for another container  
C. Using a Docker network  
D. Using a persistent volume

### Q5

What does `restart: on-failure` attempt to provide?

A. Automatic restart after a failure exit  
B. Automatic image optimization  
C. Database backups  
D. DNS configuration

### Q6

What is readiness primarily concerned with?

A. Whether the application can currently serve traffic  
B. Whether the Dockerfile exists  
C. Whether Git is installed  
D. Whether the image has a tag

### Q7

Why is graceful shutdown important?

A. It makes images smaller  
B. It helps stop applications without unnecessarily dropping active work  
C. It creates Docker networks  
D. It replaces MongoDB

### Q8

A health check uses `curl`, but the image does not contain `curl`. What can happen?

A. The health check can fail even though the application is working  
B. MongoDB is deleted  
C. Docker installs `curl` automatically  
D. Nginx changes its configuration

### Q9

Which is the better production troubleshooting sequence?

A. `Restart -> Restart -> Restart`  
B. `Logs -> State -> Health -> Network -> Configuration -> Dependencies`  
C. `Delete volume -> Rebuild everything`  
D. Change ports randomly

### Q10

Which statement is correct?

A. Container `Up` always means the application is healthy  
B. `depends_on` always guarantees readiness  
C. Restarting a failed service proves the root cause is fixed  
D. Container state, application health, and dependency availability are separate concerns

### Q11

What should a readiness endpoint commonly return when the API cannot use MongoDB?

A. HTTP `200` with a fake success response  
B. HTTP `301`  
C. HTTP `503 Service Unavailable`  
D. HTTP `404` for every failure

### Q12

What is the purpose of `start_period` in a health check?

A. To provide startup grace time before ordinary failures count  
B. To permanently disable health checks  
C. To publish a host port  
D. To delete the container after startup

### Q13

Why should a health-check command be tested inside the image?

A. To verify that the required executable and endpoint are available  
B. To change the image tag  
C. To create a Docker volume  
D. To expose MongoDB publicly

### Q14

What is a safe replacement sequence for `api:v1` and `api:v2`?

A. Stop v1, immediately route traffic to an untested v2  
B. Start v2, verify health, route traffic, then gracefully stop v1  
C. Delete all volumes and rebuild both versions  
D. Restart Nginx repeatedly

## Answer Key

```text
1 -> B
2 -> B
3 -> B
4 -> B
5 -> A
6 -> A
7 -> B
8 -> A
9 -> B
10 -> D
11 -> C
12 -> A
13 -> A
14 -> B
```

## Score

| Score | Result |
|---|---|
| 13-14 | Excellent: ready to design resilient Compose stacks |
| 10-12 | Good: review readiness and restart loops |
| 7-9 | Review health checks, dependencies, and diagnostics |
| 0-6 | Revisit Compose fundamentals and complete the lab |

## Practical Reflection

Write short answers to these questions:

1. What does `Up` prove, and what does it not prove?
2. Which command would you run first when the API is restarting?
3. What should happen when the API is live but MongoDB is unavailable?
4. Which health-check commands are actually present in your images?
5. How does graceful shutdown protect a deployment?
