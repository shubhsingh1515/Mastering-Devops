# Day 25 - Quiz: Docker Compose

## Questions

1. What problem does Docker Compose primarily solve?
   - A. Linux kernel compilation
   - B. Declaratively managing multi-container applications
   - C. Replacing Git
   - D. Encrypting MongoDB

2. In this configuration, what is `api`?

   ```yaml
   services:
     api:
       build: ./backend
   ```

   - A. An IP address
   - B. A service name
   - C. A host port
   - D. A volume

3. How should the API reach MongoDB when the service is named `mongodb`?
   - A. `mongodb:27017`
   - B. `localhost:27017`
   - C. `docker-host:27017`
   - D. A randomly selected container IP

4. What does this mapping do?

   ```yaml
   volumes:
     - mongo-data:/data/db
   ```

   - A. Publishes MongoDB to the Internet
   - B. Mounts persistent storage into MongoDB
   - C. Creates a host port
   - D. Creates a second API container

5. Which command starts services in the background?
   - A. `docker compose up -d`
   - B. `docker compose start-all`
   - C. `docker compose run-background`
   - D. `docker compose launch`

6. Does a simple `depends_on` entry guarantee that MongoDB is ready to accept connections?
   - A. Yes
   - B. No

7. Why should container IP addresses generally not be hardcoded?
   - A. They are always public
   - B. They can change; service discovery should use logical names
   - C. Docker does not use IP addresses
   - D. MongoDB does not support IP addresses

8. Which command follows Compose logs?
   - A. `docker compose logs -f`
   - B. `docker compose follow`
   - C. `docker compose tail-live`
   - D. `docker logs --compose`

9. What does `docker compose down` generally do?
   - A. Removes Docker from the machine
   - B. Removes the Compose-created containers and network
   - C. Deletes every Docker volume on the machine
   - D. Removes all Docker images

10. What is a health check useful for?
    - A. Determining whether a service passes a defined health test
    - B. Automatically backing up MongoDB
    - C. Replacing Docker networking
    - D. Encrypting containers

11. Which statement about a named volume is correct?
    - A. It is automatically an off-site backup
    - B. It stores data independently of a container's writable layer
    - C. It publishes a database port
    - D. It makes an application stateless without configuration

12. Which service would commonly be the public entry point in a production-style setup?
    - A. MongoDB
    - B. Nginx
    - C. The internal volume
    - D. The Docker network

## Answer Key

1. B  
2. B  
3. A  
4. B  
5. A  
6. B  
7. B  
8. A  
9. B  
10. A  
11. B  
12. B

## Bonus Questions

### 13. Why is `mongodb://localhost:27017/mern` usually wrong inside the API container?

Because `localhost` identifies the API container itself. The MongoDB service is reached through the Compose service name `mongodb`.

### 14. What is the safest interpretation of `docker compose down --volumes`?

It removes the stack and its declared volumes. For a local MongoDB stack, this may delete the database data, so it must be used deliberately.

### 15. Why should the API still implement retry logic when a health check exists?

Services can become unavailable after startup. Retry and backoff logic handles transient failures throughout the application's lifetime.
