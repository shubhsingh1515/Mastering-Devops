# Day 26 - Interview Questions: Docker Security & Production Hardening

## Beginner Level

### Q1. Why run a container as a non-root user?

To reduce the privileges available to the application and limit the impact of a potential compromise.

### Q2. Should secrets be baked into a Docker image?

No. Secrets should be supplied at runtime through a secure configuration or secret-management system.

### Q3. What does `--read-only` do?

It makes the container's root filesystem read-only, reducing unauthorized modification of files inside the container.

---

## Intermediate Level

### Q4. What are Linux capabilities?

Linux capabilities are fine-grained privileges that split the traditional root powers into smaller permissions that can be selectively granted or removed.

### Q5. Why would you use `--cap-drop=ALL`?

To remove unnecessary Linux privileges and reduce the container's attack surface while leaving in only the capabilities it actually needs.

### Q6. Why are resource limits useful?

They prevent a single container from consuming too much CPU or memory and reduce the blast radius if the container is compromised or misbehaves.

### Q7. Why shouldn't MongoDB be exposed to the host unnecessarily?

Because the API can reach the database via the Docker network, and exposing the database increases the risk surface without adding necessary value.

---

## Advanced Level

### Q8. Your Node.js container is running as root. What would you change?

I would update the Dockerfile to create or use a non-root user and set `USER` to that user before the app starts. I would also review the image for unnecessary packages, capabilities, and writable paths.

### Q9. What is the difference between a read-only root filesystem and explicitly writable paths?

A read-only root filesystem prevents arbitrary writes to the main filesystem, while explicit writable paths are selectively mounted or created for runtime needs such as `/tmp`, logs, or cache directories.

### Q10. How would you explain Docker security in one sentence?

Docker security is about reducing privileges, attack surface, exposed ports, writable paths, and secrets while keeping the application functional and observable.

### Q11. Why is image minimization not the same as security by itself?

Smaller images can reduce attack surface, but compatibility, vulnerability patching, secrets handling, user configuration, and runtime policies are equally important.

### Q12. What should you check before deploying an image to production?

- the base image version and provenance
- known vulnerabilities
- whether secrets are present
- whether the process runs as a non-root user
- whether unnecessary packages or capabilities remain
- whether the app has necessary health checks and resource limits

## Short Strong Answers

**Why run as non-root?**  
To reduce the attacker’s privilege and limit the impact of a compromise.

**Why no `.env` in the image?**  
Because the image should be reusable and secrets should be injected separately at runtime.

**What does a read-only filesystem do?**  
It reduces write access to the container filesystem and makes tampering harder.

**Why drop capabilities?**  
To remove functionalities the application does not need.

**Why are resource limits important?**  
To prevent one container from starving the host or neighboring containers.
