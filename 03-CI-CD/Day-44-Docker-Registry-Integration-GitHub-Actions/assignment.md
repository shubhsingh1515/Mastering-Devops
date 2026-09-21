# Day 44 - Assignment: Publish a MERN Docker Image

## Objective

Extend the Day 42 GitHub Actions workflow so that it builds, scans, authenticates to, and pushes a Docker image to a registry. The image must be tagged with the Git commit SHA and must not be published when required quality or security gates fail.

Do not hard-code passwords, tokens, or other credentials in the workflow.

---

## Target Pipeline

```text
Checkout
  |
  v
npm ci -> Lint -> Tests -> Application build
                                  |
                                  v
                           Docker build
                                  |
                                  v
                           Docker scan
                                  |
                              PASS?
                             /     \\
                           NO       YES
                           |          |
                         Stop   Registry login
                                      |
                                      v
                                  Docker push
```

---

## Part 1: Inspect the Application

Record the actual values used by your project:

```text
Repository layout:
Backend directory:
Node.js version:
Package manager:
Package lockfile:
Lint command:
Test command:
Build command:
Dockerfile path:
Registry hostname:
Registry repository:
Application port:
Health endpoint:
```

Adapt every path and command to the repository instead of copying placeholder values unchanged.

## Part 2: Run Checks Locally

```bash
npm ci
npm run lint
npm test
npm run build
```

Record the result of every command. If the project uses different scripts, record the actual commands and explain why.

## Part 3: Build and Scan the Image

```bash
GIT_SHA=$(git rev-parse --short HEAD)
docker build -t "registry.example.com/mern-api:$GIT_SHA" ./backend
```

Run the scanner approved by your project and record its policy result. Do not continue to publication when a required security policy fails.

```text
Scanner:
Image scanned:
Scan result:
Blocking findings:
Remediation required:
```

## Part 4: Configure Registry Authentication

Create protected CI/CD secrets for the registry username and password or token. Example names:

```text
REGISTRY_USERNAME
REGISTRY_PASSWORD
```

The workflow must read credentials from protected secrets or a supported workload identity, use `--password-stdin` where required, avoid printing secrets, and avoid exposing publication credentials to untrusted fork pull requests.

## Part 5: Extend the Workflow

Add login and push after all required validation and scanning steps:

```yaml
- name: Log in to registry
  run: |
    echo "${{ secrets.REGISTRY_PASSWORD }}" \\
      | docker login "$REGISTRY" \\
        --username "${{ secrets.REGISTRY_USERNAME }}" \\
        --password-stdin

- name: Push image
  run: docker push "$REGISTRY/$IMAGE_NAME:${{ github.sha }}"
```

Adapt the workflow so that tests pass before publication, the image uses `${{ github.sha }}`, scanning happens before trusted publication, and push uses the same tag that was built and scanned.

## Part 6: Prove Job Dependencies

Temporarily make a test fail on a disposable branch or draft pull request. Expected result:

```text
Validation       failed
Docker build     skipped or blocked
Image scan       skipped or blocked
Registry login   not attempted
Image push       not attempted
```

Restore the test and verify that the full pipeline runs after validation succeeds.

## Part 7: Failure Simulation

Test these scenarios in a safe development repository or disposable registry repository:

- Wrong credentials: login fails and push is blocked.
- Wrong repository name: login succeeds but push fails.
- Successful release: build, scan, login, and push succeed.

Never use real production credentials for failure simulation.

## Part 8: Verify the Registry Artifact

Verify that the registry contains:

```text
registry.example.com/mern-api:<commit-sha>
```

Record the Git SHA, short SHA, full image reference, image tag, image digest, registry repository, CI run, and scan result. Explain why deployment should use this exact tag or digest rather than rebuilding or relying only on `latest`.

## Required Deliverables

```text
[ ] Updated GitHub Actions workflow
[ ] Successful local quality-check results
[ ] Successful Docker build result
[ ] Image scan result
[ ] Registry login and push evidence
[ ] Failed-gate run evidence
[ ] Registry image tag or digest record
[ ] Secrets and permissions explanation
[ ] Explanation of how CD deploys the exact CI artifact
```
