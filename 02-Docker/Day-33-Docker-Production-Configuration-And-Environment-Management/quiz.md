# Day 33 - Quiz: Docker Production Configuration and Environment Management

Try to answer each question before checking the key.

## Questions

### Q1

What is the main reason to externalize environment-specific configuration?

A. To make Docker images larger  
B. To allow the same artifact to run in different environments  
C. To disable Docker networking  
D. To avoid using Git entirely

### Q2

Which file should generally not be committed when it contains real secrets?

A. `.env`  
B. `.env.example`  
C. `README.md`  
D. `Dockerfile`

### Q3

Which command shows the resolved Compose configuration?

A. `docker history`  
B. `docker compose config`  
C. `docker stats`  
D. `docker volume ls`

### Q4

An API container needs MongoDB in the same Compose network. Which hostname is normally correct?

A. `localhost`  
B. `mongodb`  
C. `127.0.0.1`  
D. The public server IP

### Q5

Why should production secrets not be placed in a Dockerfile?

A. Docker does not support environment variables  
B. They can become part of image layers or metadata  
C. Node.js cannot read secrets  
D. Nginx deletes them

### Q6

A typical React/Vite production environment variable is often:

A. Evaluated only by MongoDB  
B. Embedded during the frontend build  
C. Stored automatically in a Docker volume  
D. Ignored by the build

### Q7

Which release approach is generally preferable?

A. Build one independent image for every environment  
B. Build once and promote the tested artifact  
C. Rebuild manually before every deployment  
D. Commit `.env` into Git

### Q8

The API container is running but cannot connect to MongoDB. What should you check early?

A. `docker compose config` and `MONGO_URI`  
B. Delete the volume immediately  
C. Reinstall Docker  
D. Change frontend CSS

### Q9

Why is `.env.example` useful?

A. It safely documents expected configuration variables  
B. It stores production passwords  
C. It replaces the Dockerfile  
D. It creates containers

### Q10

What is the best response to a missing required `MONGO_URI`?

A. Silently continue  
B. Use an arbitrary production database  
C. Fail fast with a clear configuration error  
D. Disable MongoDB

### Q11

Which statement about `.env` is correct?

A. It automatically encrypts all values  
B. It is a production secret manager  
C. It is a convenient configuration input and must still be protected  
D. It prevents values from appearing in logs

### Q12

What does `Up` in `docker compose ps` prove?

A. Every dependency is ready  
B. The application is healthy  
C. The container process is running  
D. MongoDB has accepted the API connection

## Answer Key

1. B  
2. A  
3. B  
4. B  
5. B  
6. B  
7. B  
8. A  
9. A  
10. C  
11. C  
12. C

## Score

| Score | Result |
|---|---|
| 11-12 | Excellent: ready to apply the pattern |
| 9-10 | Good: review frontend and secret handling |
| 6-8 | Review Compose interpolation and networking |
| 0-5 | Revisit the lesson and complete the lab |
