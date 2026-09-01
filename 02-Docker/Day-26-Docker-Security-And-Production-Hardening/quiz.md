# Day 26 - Quiz: Docker Security & Production Hardening

## Questions

1. What is the main reason to run containers as non-root?
   - A. Faster startup
   - B. Reduced privilege and smaller compromise impact
   - C. Smaller images
   - D. Automatic encryption

2. Should `JWT_SECRET` be baked into a Docker image?
   - A. Yes
   - B. No

3. What does `--cap-drop=ALL` do?
   - A. Deletes all containers
   - B. Removes Linux capabilities from the container
   - C. Disables networking
   - D. Deletes volumes

4. What does `--read-only` protect?
   - A. The root filesystem from writes
   - B. The Docker registry
   - C. DNS
   - D. MongoDB backups

5. Why might a read-only filesystem break an application?
   - A. Some applications need to write temporary or runtime data
   - B. Node cannot use TCP
   - C. Docker requires write access everywhere
   - D. Nginx cannot start

6. What is the relationship between image minimization and security?
   - A. Fewer unnecessary components can reduce attack surface
   - B. Small images cannot be hacked
   - C. Image size automatically encrypts data
   - D. Image size replaces authentication

7. Why shouldn't MongoDB be unnecessarily published to the host?
   - A. It doesn't need to be reachable externally when Node can use the private Docker network
   - B. MongoDB doesn't use TCP
   - C. Docker blocks port 27017
   - D. Nginx requires it

8. What does a container health check tell you?
   - A. Whether the application meets a defined health condition
   - B. Whether the host is physically secure
   - C. Whether the database has backups
   - D. Whether the image has no vulnerabilities

9. Why use resource limits?
   - A. To prevent uncontrolled resource consumption from affecting the host and other services
   - B. To encrypt containers
   - C. To create Docker networks
   - D. To replace monitoring

10. What is the best security philosophy for containers?
   - A. Give every container root and restrict nothing
   - B. Defense in depth and least privilege
   - C. Expose every port
   - D. Put credentials in Dockerfiles

## Answer Key

1. B  
2. B  
3. B  
4. A  
5. A  
6. A  
7. A  
8. A  
9. A  
10. B

## Bonus Questions

### 11. Why is `mongodb://localhost:27017/mern` usually wrong inside the API container?

Because `localhost` refers to the current container. The correct target is usually the Compose service name `mongodb`.

### 12. Why is `--tmpfs /tmp` sometimes used with `--read-only`?

Because some applications need a writable temporary directory even when the rest of the root filesystem is read-only.

### 13. What is the principle behind "least privilege" in Docker?

Only give the container the permissions, filesystem access, ports, and capabilities that are truly necessary for it to function.

### 14. Why should image scanning happen before deployment?

To identify known vulnerabilities before the image is used in a real environment.

### 15. What is a practical defense-in-depth strategy?

Use multiple layers such as non-root user, minimal image, dropped capabilities, read-only filesystem, secret management, and resource limits.
