# Day 42 - Quiz: GitHub Actions Production CI

## Questions

### Q1

What is a GitHub Actions runner?

A. A database server  
B. An environment that executes workflow jobs  
C. A Docker registry  
D. A Git branch

### Q2

What is a workflow?

A. A complete YAML automation definition  
B. A MongoDB collection  
C. A Docker volume  
D. A Git commit

### Q3

What is a job?

A. A group of related steps executed on a runner  
B. A secret  
C. A Docker image  
D. A database

### Q4

What is a step?

A. An individual command or reusable action within a job  
B. A Git repository  
C. A Docker network  
D. A production server

### Q5

What is the main purpose of dependency caching?

A. Replacing tests  
B. Faster repeated pipeline execution  
C. Storing production passwords  
D. Publishing Docker images

### Q6

Which command is preferred for deterministic Node dependency installation when a lockfile is present?

A. `npm install` only  
B. `npm ci`  
C. `npm remove`  
D. `npm publish`

### Q7

What should happen when `test` fails and `build` contains `needs: test`?

A. Build runs anyway  
B. Build should not proceed  
C. Production deploys  
D. The registry is deleted

### Q8

Which value is generally appropriate for a Docker image tag?

A. Commit SHA  
B. Database password  
C. Secret token  
D. Unrelated random text

### Q9

What is a workflow artifact?

A. A useful output produced by a workflow  
B. A GitHub password  
C. A Docker daemon  
D. A database user

### Q10

Why separate CI and CD?

A. To distinguish validation and building from controlled deployment  
B. To eliminate testing  
C. To avoid Git  
D. To prevent Docker from running

### Q11

Which trigger runs a workflow when a pull request targets `main`?

A. `pull_request: branches: [main]`  
B. `docker: main`  
C. `database: pull_request`  
D. `runner: merge`

### Q12

What does `${{ github.sha }}` represent?

A. The database password  
B. The commit SHA for the workflow revision  
C. The runner's IP address  
D. The Docker registry URL

### Q13

Where should a registry token be stored?

A. In `README.md`  
B. In the Dockerfile  
C. In GitHub encrypted secrets or an external secret manager  
D. In a public environment variable

### Q14

What should happen when the dependency cache is empty?

A. The pipeline should still install dependencies and continue correctly  
B. The repository should be deleted  
C. Tests should be skipped permanently  
D. Production should deploy

### Q15

What is the safest image promotion model?

A. Rebuild independently in every environment  
B. Build, test, and scan one image, then promote that same image  
C. Always deploy `latest` only  
D. Build directly on the production server

## Answer Key

```text
1  -> B
2  -> A
3  -> A
4  -> A
5  -> B
6  -> B
7  -> B
8  -> A
9  -> A
10 -> A
11 -> A
12 -> B
13 -> C
14 -> A
15 -> B
```

## Score

```text
14-15  Excellent understanding
11-13  Good; review gates, artifacts, and secrets
8-10   Revisit workflow structure and dependency handling
0-7    Repeat the lesson and complete the practical assignment
```

## Reflection Questions

1. What is the difference between a workflow, job, step, and runner?
2. Why is `npm ci` preferred in CI when a lockfile exists?
3. Why should a cache never be required for correctness?
4. What is the difference between a cache and an artifact?
5. Why should a failed test prevent Docker publication?
6. Why is a commit SHA more useful than `latest` alone?
7. Which values belong in variables and which belong in secrets?
8. Why should fork pull requests have restricted permissions?
9. Which jobs can safely run in parallel?
10. What would you measure before optimizing a slow pipeline?
