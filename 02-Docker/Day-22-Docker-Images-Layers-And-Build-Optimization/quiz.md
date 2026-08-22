# Day 22 — Quiz: Docker Images, Layers & Build Optimization

## Questions

1. Why are Docker layers useful?
   - A. They automatically encrypt containers
   - B. They enable reuse and caching of unchanged build steps
   - C. They replace Linux permissions
   - D. They create DNS records

2. What file controls which files are excluded from the Docker build context?
   - A. `.dockerignore`
   - B. `.dockerexclude`
   - C. `.dockerconfig`
   - D. `.ignore`

3. Why should `package*.json` usually be copied before source?
   - A. To expose port 3000
   - B. To improve dependency-layer caching
   - C. To start Node
   - D. To enable SSH

4. What command shows image history?
   - A. `docker layers`
   - B. `docker history`
   - C. `docker cache`
   - D. `docker inspect-history`

5. What is the main purpose of a multi-stage build?
   - A. Run two databases
   - B. Keep build-time dependencies out of the final runtime image
   - C. Automatically scale containers
   - D. Replace Kubernetes

6. Should production secrets be baked into an image?
   - A. Yes
   - B. No

7. Which is generally preferable for a production application?
   - A. Run as root whenever possible
   - B. Run as a non-root user where practical
   - C. Disable all permissions
   - D. Use `chmod 777`

8. If `server.js` changes but `package.json` does not, what should ideally happen?
   - A. Dependencies needlessly reinstall
   - B. Docker can reuse the dependency layer
   - C. The base image must rebuild
   - D. MongoDB must restart

9. Which command builds an image?
   - A. `docker compile`
   - B. `docker build`
   - C. `docker package`
   - D. `docker make`

10. What is a major benefit of a multi-stage React build?
    - A. It lets the final image omit unnecessary build tooling
    - B. It makes MongoDB faster
    - C. It removes HTTP
    - D. It replaces Nginx automatically

---

## ✅ Answer Key with Explanations

1. **B** — Layers allow Docker to reuse unchanged build steps instead of rebuilding everything.
2. **A** — `.dockerignore` excludes files from the build context.
3. **B** — Copying manifests first allows dependency installation to remain cached when source changes.
4. **B** — `docker history image:tag` displays image history and layers.
5. **B** — Multi-stage builds keep build tools and development dependencies out of the runtime image.
6. **B** — Secrets should be injected at runtime and managed separately from the image.
7. **B** — Non-root execution follows least privilege and reduces potential impact.
8. **B** — A correctly ordered Dockerfile can reuse the dependency layer.
9. **B** — `docker build` creates an image from a Dockerfile and build context.
10. **A** — The final Nginx image needs static assets, not the complete Node build environment.

---

## Bonus Questions

11. What does `EXPOSE 3000` do?

**Answer:** It documents the container port. It does not publish the port to the host.

12. What command maps host port 3001 to container port 3000?

**Answer:**

```bash
docker run -p 3001:3000 image
```

13. Why should `node_modules` normally be excluded from `.dockerignore`?

**Answer:** Local dependencies may be platform-specific and can unnecessarily enlarge the build context. Dependencies should generally be installed inside the image.

14. What is one risk of putting a secret in a Dockerfile `RUN` command?

**Answer:** The value may remain visible in image history or build metadata even if the file is later deleted.

15. What does `USER node` accomplish?

**Answer:** It configures the container to run the following process as the non-root `node` user.

16. Why might a smaller Alpine image fail to run a Node dependency?

**Answer:** Alpine uses `musl` libc, and some native dependencies expect `glibc` or require additional build/runtime libraries.

17. What is the purpose of `npm ci --omit=dev` in a runtime dependency stage?

**Answer:** It performs a lockfile-based install while omitting development dependencies that are not required to run the application.

18. What should be measured after Docker optimization?

**Answer:** Build time, image size, cache reuse, pull/start time, vulnerability results, runtime behavior, and reproducibility.
