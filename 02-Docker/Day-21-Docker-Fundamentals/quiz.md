# Day 21: Docker Fundamentals Quiz

## Questions

### 1. What creates a Docker image?
A. `docker start`  
B. `docker build`  
C. `docker exec`  
D. `docker logs`

---

### 2. What creates a running container from an image?
A. `docker run`  
B. `docker image`  
C. `docker build`  
D. `docker pull`

---

### 3. What does this mean?
```bash
-p 8080:3000
```
A. Container 8080 → host 3000  
B. Host 8080 → container 3000  
C. Two containers communicate  
D. Docker exposes both ports automatically

---

### 4. What does `FROM` specify in a Dockerfile?
A. Runtime command  
B. Base image  
C. Container port  
D. Volume

---

### 5. What does `WORKDIR /app` do?
A. Creates a host directory  
B. Sets the working directory inside the image/container  
C. Publishes port 80  
D. Starts Node.js

---

### 6. What does `RUN npm ci` execute?
A. At image build time  
B. Only when Docker stops  
C. On the host after deployment  
D. During DNS resolution

---

### 7. What does `CMD` specify?
A. Default runtime command  
B. Base OS  
C. Build cache  
D. Firewall rule

---

### 8. Does `EXPOSE 3000` automatically publish port 3000?
A. Yes  
B. No

---

### 9. Which command shows container logs?
A. `docker journal`  
B. `docker logs`  
C. `docker output`  
D. `docker trace`

---

### 10. Which is lighter in typical deployments?
A. A full VM with its own guest OS  
B. A container sharing the host kernel

---

## Answer Key

```text
1. B
2. A
3. B
4. B
5. B
6. A
7. A
8. B
9. B
10. B
```

---

## Quick Scoring

- 9–10: Excellent Docker understanding
- 7–8: Good progress, review Docker networking and Dockerfile instructions
- 5–6: You know the basics, but need more repetition
- 0–4: Revisit the lesson and rewrite the commands from memory

---

## Bonus interview practice

Be ready to explain:
- image vs container
- build vs runtime
- `CMD` vs `RUN`
- `EXPOSE` vs `-p`
- why `0.0.0.0` matters in containerized apps
- why Docker reduces environment drift in MERN projects
