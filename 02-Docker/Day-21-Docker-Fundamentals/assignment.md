# Day 21: Docker Fundamentals — Assignment

## 1) Create a practice Docker workspace

Use only terminal commands to create the following structure:

```text
devops-course/
    docker/
        day21/
            app/
            notes/
            scripts/
```

Then verify with:

```bash
ls -R devops-course
```

---

## 2) Build a simple Node.js Docker app

Create a directory called:

```bash
mkdir -p ~/devops-course/docker/day21/app
cd ~/devops-course/docker/day21/app
```

Create a file named `server.js` with this content:

```javascript
const http = require("http");

const server = http.createServer((req, res) => {
  if (req.url === "/health") {
    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(JSON.stringify({ status: "ok" }));
    return;
  }

  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Docker app running\n");
});

server.listen(3000, "0.0.0.0", () => {
  console.log("Server listening on port 3000");
});
```

Create a Dockerfile:

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY server.js .
EXPOSE 3000
CMD ["node", "server.js"]
```

Build the image:

```bash
docker build -t docker-day21:1.0 .
```

Run the container:

```bash
docker run -d --name docker-day21 -p 3000:3000 docker-day21:1.0
```

Test it:

```bash
curl http://localhost:3000/health
```

Expected response:

```json
{ "status": "ok" }
```

---

## 3) Inspect the running container

Run the following:

```bash
docker ps
docker logs docker-day21
docker inspect docker-day21
docker exec -it docker-day21 sh
```

Inside the container, run:

```bash
ps
ls
```

Then exit:

```bash
exit
```

---

## 4) Mini assignment questions

Write short answers to these questions:

1. What is the difference between a Docker image and a Docker container?
2. Why is `0.0.0.0` useful when binding a server inside a container?
3. Why is `EXPOSE 3000` not the same as `-p 3000:3000`?
4. Why is `COPY package*.json ./` often used before `COPY . .` in Dockerfiles?
5. Why is Docker useful for MERN application deployment?

---

## 5) Hands-on lab objective

By the end of this assignment, you should be able to:
- build a Docker image
- run a container from that image
- expose a port from container to host
- inspect logs and metadata
- explain the difference between image and container
- describe why Docker improves consistency across environments

---

## 6) Weekly practice reminder

Repeat this exercise 2–3 times until you can explain each command and each Dockerfile instruction without reading the notes.

This helps build real confidence for interviews and real-world DevOps tasks.
