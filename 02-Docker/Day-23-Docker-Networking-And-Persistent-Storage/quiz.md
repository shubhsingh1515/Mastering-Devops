# Day 23 — Quiz: Docker Networking & Persistent Storage

## Questions

1. Inside a Node container, what does `localhost` refer to?
   - A. The MongoDB container
   - B. The Docker host
   - C. The Node container itself
   - D. The Internet

2. How do two containers communicate using a user-defined Docker network?
   - A. Through container/service names and Docker networking
   - B. Only through public IPs
   - C. Through SSH
   - D. Through Nginx

3. Which command creates a Docker network?
   - A. `docker network create`
   - B. `docker create network`
   - C. `docker net init`
   - D. `docker connect`

4. Which command creates a named volume?
   - A. `docker volume create`
   - B. `docker storage init`
   - C. `docker disk create`
   - D. `docker mount create`

5. What is the purpose of a MongoDB volume?
   - A. Expose MongoDB publicly
   - B. Persist database data independently of a container's lifecycle
   - C. Encrypt MongoDB automatically
   - D. Replace backups

6. Does a volume automatically constitute a backup strategy?
   - A. Yes
   - B. No

7. Why might you avoid publishing port `27017`?
   - A. MongoDB cannot use TCP
   - B. The application can communicate over the private Docker network
   - C. Docker doesn't support 27017
   - D. Nginx requires it

8. Which command inspects a Docker network?
   - A. `docker network inspect`
   - B. `docker network show-config`
   - C. `docker inspect-network-config`
   - D. `docker net status`

9. Which URI is correct when MongoDB's container hostname is `mongodb`?
   - A. `mongodb://localhost:27017/mydb`
   - B. `mongodb://mongodb:27017/mydb`
   - C. `mongodb://127.0.0.1:80/mydb`
   - D. `mongodb://docker:3000/mydb`

10. What is a major benefit of separating Nginx, Node, and MongoDB into containers?
   - A. All services become public automatically
   - B. Components can be isolated, deployed, and managed independently
   - C. Backups are no longer required
   - D. Security configuration becomes unnecessary

---

## Answer Key

1. C
2. A
3. A
4. A
5. B
6. B
7. B
8. A
9. B
10. B

---

## Bonus Questions

### 11. Why is `localhost` wrong inside a container that must reach another container?

Because it refers to the current container, not the peer service.

### 12. What does `docker volume` protect against?

Loss of data when a container is removed or recreated.

### 13. Why is `-p 3000:3000` often used for the API but not for MongoDB?

The API is meant to be reachable from outside the Docker network, while MongoDB usually stays private.

### 14. What is the purpose of `mongo-data:/data/db`?

It mounts a Docker volume to the MongoDB data directory so database files remain on disk outside the container filesystem.

### 15. What is the missing concept after persistence?

Backup and restore strategy.
