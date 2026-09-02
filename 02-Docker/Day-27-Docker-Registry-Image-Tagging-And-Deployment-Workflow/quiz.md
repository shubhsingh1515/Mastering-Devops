# Day 27 - Quiz: Docker Registry, Tagging and Deployment

## Questions

1. What is the purpose of a Docker registry?
   - A. Run Linux processes
   - B. Store and distribute container images
   - C. Replace Dockerfiles
   - D. Manage CPU

2. Which reference is generally more deterministic for production?
   - A. `latest`
   - B. A specific version tag or immutable digest
   - C. A random container ID
   - D. A local source directory

3. What does `docker push` do?
   - A. Uploads an image to a registry
   - B. Starts a container
   - C. Builds an image
   - D. Creates a volume

4. What does `docker pull` do?
   - A. Deletes an image
   - B. Downloads an image from a registry
   - C. Builds a Dockerfile
   - D. Creates a network

5. Why is build-once/promote useful?
   - A. It ensures environments use the same tested artifact
   - B. It removes testing
   - C. It avoids registries
   - D. It requires rebuilding production

6. What is an image digest?
   - A. A content-addressed identifier for image content
   - B. A container password
   - C. A port number
   - D. A Docker network

7. What is a common rollback strategy?
   - A. Rebuild random source code
   - B. Redeploy the previous known-good image
   - C. Delete the registry
   - D. Change MongoDB's port

8. Why should registry credentials follow least privilege?
   - A. To limit damage if credentials are compromised
   - B. To make images smaller
   - C. To speed up Docker
   - D. To improve DNS

9. Which deployment flow is preferable?
   - A. Build separately for staging and production
   - B. Build once, test, then promote the same artifact
   - C. Build directly on production
   - D. Build manually on every server

10. Which gives the strongest exact-content reference?
    - A. Mutable `latest` tag
    - B. Image digest
    - C. Container name
    - D. Hostname

11. What does `docker tag` do?
    - A. Adds a reference to image content
    - B. Starts a new container
    - C. Uploads an image automatically
    - D. Deletes old layers

12. Which item is part of Docker supply-chain security?
    - A. Image scanning and registry access control
    - B. Exposing every port
    - C. Storing tokens in Dockerfiles
    - D. Rebuilding manually on production

## Answer Key

1. B  
2. B  
3. A  
4. B  
5. A  
6. A  
7. B  
8. A  
9. B  
10. B  
11. A  
12. A

## Bonus Questions

### 13. Why can two servers using `latest` run different code?

The tag may have moved after one server pulled it but before the other server pulled it. Each server can therefore have different image content behind the same tag.

### 14. What is artifact promotion?

Moving the same built and tested image from the registry to staging and then to production without rebuilding it.

### 15. Why should a previous image remain in the registry?

It provides a known-good artifact for fast rollback after a failed deployment.

### 16. What is the difference between a tag and a digest?

A tag is a human-friendly name that may move. A digest is a content-addressed identifier for exact image content.

### 17. Why is rebuilding during rollback risky?

The rebuild may resolve different dependencies, use a different base image, or produce a different artifact from the one previously tested.
