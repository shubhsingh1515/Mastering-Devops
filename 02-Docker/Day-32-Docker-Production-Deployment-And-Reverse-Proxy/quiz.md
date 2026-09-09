# Day 32 - Quiz: Docker Production Deployment and Reverse Proxy

## Questions

1. Nginx and Node are separate containers. Which address is normally correct for Nginx to reach Node?
   - A. `localhost:3000`
   - B. `api:3000`
   - C. `127.0.0.1:3000`
   - D. Host public IP only

2. What is the primary purpose of a reverse proxy?
   - A. Store MongoDB data
   - B. Forward client requests to backend services
   - C. Build Docker images
   - D. Create Git branches

3. Which service should generally not be publicly exposed in a basic MERN Docker architecture?
   - A. Nginx
   - B. MongoDB
   - C. HTTPS endpoint
   - D. Reverse proxy

4. Nginx returns `502 Bad Gateway`. What should you do first?
   - A. Delete all containers
   - B. Diagnose the upstream path and backend connectivity
   - C. Delete the Docker volume
   - D. Reinstall Docker

5. Why should Nginx not use a hard-coded container IP?
   - A. IPs never work in Docker
   - B. Container IPs can change
   - C. Nginx does not support IP addresses
   - D. Docker requires DNS

6. What does this mapping mean?

   ```yaml
   ports:
     - "80:80"
   ```

   - A. Host port 80 maps to container port 80
   - B. Container port 80 maps to MongoDB
   - C. Two containers share port 80
   - D. Nginx uses port 8080

7. What is a common production approach for React?
   - A. Run the development server forever
   - B. Build static assets and serve them through a production web server
   - C. Run MongoDB inside React
   - D. Put React source code into MongoDB

8. Why might the API have no `ports` section in Compose?
   - A. It cannot communicate
   - B. Nginx can reach it over the internal Docker network
   - C. Node does not use ports
   - D. Compose deletes the API

9. Which path represents a correct request flow?
   - A. MongoDB -> Browser -> Nginx -> API
   - B. Browser -> Nginx -> API -> MongoDB
   - C. Browser -> MongoDB -> Nginx
   - D. API -> Browser -> MongoDB

10. A container is `Up`, but Nginx returns `502`. What does this tell you?
    - A. The backend must be healthy
    - B. Container state alone does not prove upstream connectivity
    - C. Docker networking is definitely working
    - D. MongoDB is definitely broken

11. Which command tests whether the API service name resolves from Nginx?
    - A. `docker compose exec nginx getent hosts api`
    - B. `docker image api`
    - C. `docker volume api`
    - D. `docker tag api`

12. What is the best first boundary to test when investigating a `502`?
    - A. Delete the database
    - B. Nginx-to-API connectivity
    - C. Rebuild every image without logs
    - D. Change MongoDB data

## Answer Key

1. B  
2. B  
3. B  
4. B  
5. B  
6. A  
7. B  
8. B  
9. B  
10. B  
11. A  
12. B

## Score Guide

- 11-12: Excellent
- 9-10: Good; review request routing
- 6-8: Review Docker networking and Compose
- Below 6: Revisit reverse proxy fundamentals

## Bonus Questions

### 13. Why is `proxy_pass http://localhost:3000` usually wrong inside Nginx?

Because `localhost` points to the Nginx container. The API is a separate service and should be addressed as `api:3000` on the shared Docker network.

### 14. What should you inspect after seeing a `502`?

Nginx logs, API logs, service state, network membership, DNS resolution, API port and listener, the API health endpoint, and database dependencies.

### 15. Why is the API allowed to have an internal port without a public port mapping?

A service can listen on a container port and be reachable by other containers on the same network without publishing the port on the host.

### 16. Why is a React production build preferred over its development server?

A production build creates static assets that a production web server can serve efficiently without shipping the development toolchain.

### 17. What is the core Day 32 troubleshooting principle?

Trace the request from the client through Nginx, Docker network, service DNS, API port, application health, and database rather than treating containers in isolation.
