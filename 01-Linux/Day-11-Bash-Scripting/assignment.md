# Day 11 — Assignment: Bash Scripting Fundamentals

## Objective
Apply what you've learned about Bash scripting to build automation scripts for MERN deployments, health checks, and pre-deployment validation.

---

## Part 1: Create Your Scripting Workspace

```bash
mkdir -p ~/devops-course/day11
cd ~/devops-course/day11
```

## Part 2: Basic Scripts

### Exercise 1 — hello.sh
Create a script that prints a welcome message.

```bash
#!/bin/bash

echo "Welcome to DevOps!"
```

Make it executable and run it:
```bash
chmod +x hello.sh
./hello.sh
```

### Exercise 2 — variable.sh
Create a script using variables and command-line arguments.

```bash
#!/bin/bash

APP="backend"
ENV="${1:-production}"   # Default to production if no argument given

echo "Deploying $APP to $ENV"
```

Run it:
```bash
./variable.sh              # Deploying backend to production
./variable.sh staging      # Deploying backend to staging
```

### Exercise 3 — conditional.sh
Create a script that checks if a directory exists.

```bash
#!/bin/bash

if [ -d "backend" ]; then
    echo "Backend directory exists"
else
    echo "Backend directory missing"
    mkdir backend
    echo "Created backend directory"
fi
```

### Exercise 4 — loop.sh
Create a script using a for loop.

```bash
#!/bin/bash

for service in nginx mongodb nodeapp
do
    echo "Checking $service..."
    systemctl is-active --quiet "$service"
    if [ $? -eq 0 ]; then
        echo "  $service is running"
    else
        echo "  $service is NOT running"
    fi
done
```

---

## Part 3: Coding Exercise — server_health.sh

Create `server_health.sh`:

```bash
#!/bin/bash

echo "===== Hostname ====="
hostname

echo
echo "===== Disk Usage ====="
df -h

echo
echo "===== Memory ====="
free -h

echo
echo "===== Running Services ====="
systemctl --type=service --state=running --no-pager

echo
echo "===== Node Process ====="
ps -ef | grep node | grep -v grep
```

Make it executable:
```bash
chmod +x server_health.sh
./server_health.sh
```

---

## Part 4: Hands-on Lab — MERN Deployment Script

Create `deploy.sh` that:

1. **Verifies a backend directory exists**
   ```bash
   if [ -d "backend" ]; then
       echo "Backend directory found"
   else
       echo "Error: backend directory missing"
       exit 1
   fi
   ```

2. **Changes into the directory**
   ```bash
   cd backend
   ```

3. **Installs dependencies**
   ```bash
   npm install || exit 1
   ```

4. **Restarts the service**
   ```bash
   sudo systemctl restart myapp || exit 1
   ```

5. **Performs a health check**
   ```bash
   curl -f http://localhost:3000/health || exit 1
   ```

6. **Prints "Deployment Successful" only if every step succeeds**
   ```bash
   echo "Deployment Successful"
   ```

### Full script:
```bash
#!/bin/bash

echo "=== Starting MERN Deployment ==="

if [ -d "backend" ]; then
    echo "Backend directory found"
else
    echo "Error: backend directory missing"
    exit 1
fi

cd backend || exit 1

echo "Installing dependencies..."
npm install || exit 1

echo "Restarting service..."
sudo systemctl restart myapp || exit 1

echo "Checking application..."
curl -f http://localhost:3000/health || exit 1

echo
echo "=== Deployment Successful ==="
```

### Stretch Goal
Redirect all output to `deployment.log` while also displaying on terminal:
```bash
./deploy.sh 2>&1 | tee deployment.log
```

---

## Part 5: Mini Assignment — pre_deployment_check.sh

Write a Bash script `pre_deployment_check.sh` that:

1. **Verifies at least 2 GB of free disk space**
   ```bash
   FREE_SPACE=$(df --output=avail / | tail -1 | tr -d ' ')
   if [ "$FREE_SPACE" -lt 2000000 ]; then
       echo "FAIL: Less than 2GB free disk space"
   else
       echo "PASS: Sufficient disk space ($((FREE_SPACE/1000)) MB free)"
   fi
   ```

2. **Confirms Git is installed**
   ```bash
   if command -v git &>/dev/null; then
       echo "PASS: Git is installed"
   else
       echo "FAIL: Git is not installed"
   fi
   ```

3. **Confirms Node.js is installed**
   ```bash
   if command -v node &>/dev/null; then
       echo "PASS: Node.js is installed"
   else
       echo "FAIL: Node.js is not installed"
   fi
   ```

4. **Checks whether the application service is active**
   ```bash
   if systemctl is-active --quiet myapp; then
       echo "PASS: Service is active"
   else
       echo "FAIL: Service is not active"
   fi
   ```

5. **Tests whether port 3000 is listening**
   ```bash
   if ss -tuln | grep -q ":3000 "; then
       echo "PASS: Port 3000 is listening"
   else
       echo "FAIL: Port 3000 is not listening"
   fi
   ```

6. **Prints a pass/fail summary**

### Full script:
```bash
#!/bin/bash

echo "=== Pre-Deployment Checks ==="
echo

PASS=0
FAIL=0

# 1. Disk space check
FREE_SPACE=$(df --output=avail / | tail -1 | tr -d ' ')
if [ "$FREE_SPACE" -lt 2000000 ]; then
    echo "FAIL: Less than 2GB free disk space"
    FAIL=$((FAIL+1))
else
    echo "PASS: Sufficient disk space ($((FREE_SPACE/1000)) MB free)"
    PASS=$((PASS+1))
fi

# 2. Git check
if command -v git &>/dev/null; then
    echo "PASS: Git is installed"
    PASS=$((PASS+1))
else
    echo "FAIL: Git is not installed"
    FAIL=$((FAIL+1))
fi

# 3. Node.js check
if command -v node &>/dev/null; then
    echo "PASS: Node.js is installed"
    PASS=$((PASS+1))
else
    echo "FAIL: Node.js is not installed"
    FAIL=$((FAIL+1))
fi

# 4. Service check
if systemctl is-active --quiet myapp; then
    echo "PASS: Service is active"
    PASS=$((PASS+1))
else
    echo "FAIL: Service is not active"
    FAIL=$((FAIL+1))
fi

# 5. Port 3000 check
if ss -tuln | grep -q ":3000 "; then
    echo "PASS: Port 3000 is listening"
    PASS=$((PASS+1))
else
    echo "FAIL: Port 3000 is not listening"
    FAIL=$((FAIL+1))
fi

echo
echo "================================="
echo "Summary: $PASS passed, $FAIL failed"
echo "================================="

if [ "$FAIL" -gt 0 ]; then
    exit 1
else
    echo "All checks passed. Ready for deployment."
    exit 0
fi
```

---

## Part 6: Knowledge Check

Answer the following questions:

1. **Why is `set -e` important in a deployment script?**

   <details>
   <summary>Answer</summary>

   It makes the script exit immediately if any command returns a non-zero exit code, preventing the script from continuing after a failure and leaving the system in an inconsistent state.
   </details>

2. **What is the difference between `$@` and `$*`?**

   <details>
   <summary>Answer</summary>

   `$@` treats each argument as a separate word (safer with spaces), while `$*` treats all arguments as a single string.
   </details>

3. **Why should variables be quoted in scripts?**

   <details>
   <summary>Answer</summary>

   Quoting variables (`"$VAR"`) prevents word-splitting and glob expansion, so values containing spaces or special characters are handled correctly.
   </details>

4. **What does `command -v git` do and why is it useful?**

   <details>
   <summary>Answer</summary>

   It checks whether `git` exists in the PATH. It's useful for verifying prerequisites before running a script that depends on them.
   </details>

5. **What is the purpose of `grep -v grep` in `ps -ef | grep node | grep -v grep`?**

   <details>
   <summary>Answer</summary>

   It filters out the `grep` command itself from the results, so only actual node processes are shown.
   </details>

---

## Submission Checklist

- [ ] Created `~/devops-course/day11/` workspace
- [ ] Created and ran `hello.sh`
- [ ] Created and ran `variable.sh`
- [ ] Created and ran `conditional.sh`
- [ ] Created and ran `loop.sh`
- [ ] Created and ran `server_health.sh`
- [ ] Created and ran `deploy.sh`
- [ ] Created and ran `pre_deployment_check.sh`
- [ ] Answered the knowledge check questions
