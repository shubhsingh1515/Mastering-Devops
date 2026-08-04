# Day 11: Bash Scripting Fundamentals

## 📅 Phase 1: Linux
- **Topic:** Bash Scripting Fundamentals
- **Goal:** Learn to automate repetitive Linux tasks using Bash scripts—the first major automation skill every DevOps engineer needs.

---

## 🎯 Learning Objectives

By the end of this lesson, I'll be able to:

- **Create executable Bash scripts.**
- **Use variables** to store data.
- **Accept user input** interactively.
- **Write conditional logic** with `if`/`else`.
- **Use loops** (`for`, `while`) for repetition.
- **Create reusable functions.**
- **Check exit codes** to handle failures.
- **Pass command-line arguments** to scripts.
- **Automate common MERN deployment tasks.**

---

## 🤔 Why This Topic Is Important

Imagine deploying my MERN application.

**Without scripting:**
```bash
git pull
npm install
npm run build
sudo systemctl restart myapp
curl http://localhost:3000/health
```
I repeat these commands every deployment.

**With Bash:**
```bash
./deploy.sh
```

**Automation reduces mistakes and makes deployments consistent.** This is the essence of DevOps.

---

## 📖 Theory Explained

### Every Script Starts With a Shebang

```bash
#!/bin/bash
```

This tells Linux **which interpreter** should execute the script. The `#!` is called a "shebang" (hash + bang). Without it, the system won't know to use Bash.

### Variables

```bash
APP_NAME="mern-api"

echo $APP_NAME
```
Output: `mern-api`

**Rules for variables:**
- No spaces around `=` (e.g., `NAME="value"`, not `NAME = "value"`)
- Case-sensitive
- Use `$VAR` to access the value

### User Input

```bash
read -p "Enter your name: " NAME

echo "Hello $NAME"
```
`read` captures user input into a variable. The `-p` flag shows a prompt.

### Command-Line Arguments

```bash
#!/bin/bash

echo "Application: $1"
echo "Environment: $2"
```

Run:
```bash
./deploy.sh backend production
```
Output:
```
Application: backend
Environment: production
```

**Arguments:**
- `$0` — Script name
- `$1` — First argument
- `$2` — Second argument
- `$#` — Number of arguments
- `$@` — All arguments

### Conditional Statements

```bash
if [ -f ".env" ]; then
    echo ".env found"
else
    echo ".env missing"
fi
```

**Common file tests:**

| Test | Meaning |
|------|---------|
| `-f` | File exists |
| `-d` | Directory exists |
| `-z` | Empty string |
| `-n` | Non-empty string |
| `-eq` | Equal (numbers) |
| `-ne` | Not equal (numbers) |
| `-gt` | Greater than (numbers) |
| `-lt` | Less than (numbers) |

### Loops

**For Loop** — iterate over a list:
```bash
for service in nginx docker ssh
do
    echo "$service"
done
```

**While Loop** — repeat while a condition is true:
```bash
COUNT=1

while [ $COUNT -le 3 ]
do
    echo $COUNT
    COUNT=$((COUNT+1))
done
```

### Functions

```bash
check_disk() {
    df -h
}

check_disk
```

**Functions improve readability and reduce duplication.** They let me organize a script into logical, reusable blocks.

### Exit Codes

Linux commands return an **exit code** after running.

```bash
echo $?
```

**Typical values:**
- `0` → Success
- `Non-zero` → Error (e.g., `1`, `2`, `127`)

**Example:**
```bash
mkdir test
echo $?   # 0 if the directory was created
```

### Using Exit Codes

```bash
if git pull
then
    echo "Pull successful"
else
    echo "Pull failed"
fi
```

This lets me make decisions based on whether a command succeeded.

---

## 🌍 Real-World MERN Example

### Deployment Script

```bash
#!/bin/bash

echo "Updating code..."
git pull || exit 1

echo "Installing dependencies..."
npm install || exit 1

echo "Restarting service..."
sudo systemctl restart myapp || exit 1

echo "Checking application..."
curl -f http://localhost:3000/health

echo "Deployment completed."
```

**Key concept:** `|| exit 1` means "if the command fails (non-zero exit code), exit the script immediately with code 1."

This script **stops immediately if a critical step fails**, preventing partial deployments.

---

## 🏗️ Script Flow Diagram

```
Start
  │
git pull
  │
Success?
 ├── No → Exit
 └── Yes
       │
npm install
       │
Restart Service
       │
Health Check
       │
Deployment Complete
```

---

## 🛠️ Practical Exercise

### Step 1 — Create a workspace
```bash
mkdir -p ~/devops-course/day11
cd ~/devops-course/day11
```

### Step 2 — Create a script
```bash
nano hello.sh
```
Contents:
```bash
#!/bin/bash

echo "Welcome to DevOps!"
```

### Step 3 — Make it executable
```bash
chmod +x hello.sh
```

### Step 4 — Run it
```bash
./hello.sh
```

### Create a Variable Script
```bash
#!/bin/bash

APP="backend"

echo "Deploying $APP"
```

### Accept Input
```bash
#!/bin/bash

read -p "Environment: " ENV

echo "Deploying to $ENV"
```

### Add Conditional Logic
```bash
#!/bin/bash

if [ -d "backend" ]; then
    echo "Backend exists"
else
    echo "Backend missing"
fi
```

---

## 💻 Commands Used

| Command | Purpose |
|---------|---------|
| `chmod +x` | Make a script executable |
| `./script.sh` | Execute a script |
| `read` | Read user input |
| `echo` | Display output |
| `if` | Conditional execution |
| `for` | Iterate over a list |
| `while` | Repeat while a condition is true |
| `function_name()` | Define a function |
| `$?` | Check last command's exit code |
| `$1, $2` | Read command-line arguments |

---

## ✅ Best Practices

1. **Always start with a shebang** (`#!/bin/bash`).
2. **Quote variables** — `"$VARIABLE"` — to handle spaces safely.
3. **Check exit codes** after important commands.
4. **Break large scripts into functions.**
5. **Add comments** explaining non-obvious logic.
6. **Keep scripts idempotent** where possible (safe to run multiple times).

---

## ❌ Common Mistakes

1. **Forgetting `chmod +x`** — The script won't be executable.
2. **Using unquoted variables** containing spaces — Causes errors.
3. **Ignoring command failures** — Script continues even after a critical error.
4. **Hardcoding server paths** throughout the script — Makes it non-reusable.
5. **Running destructive commands without validation** — Can cause data loss.

---

## 💼 Interview Questions

### Beginner

**Q1: What is Bash?**
> **Bash (Bourne Again Shell)** is both a command-line shell and a scripting language. It's the default shell on most Linux distributions. Bash lets me run commands interactively and write scripts to automate repetitive tasks.

**Q2: What is a shebang?**
> A **shebang** (`#!/bin/bash`) is the first line of a script that tells the operating system which interpreter to use to execute the script. The `#!` is followed by the path to the interpreter (e.g., `/bin/bash`).

**Q3: How do you execute a Bash script?**
> First make it executable with `chmod +x script.sh`, then run it with `./script.sh`. Alternatively, I can run `bash script.sh` without making it executable.

**Q4: How do you define a variable?**
> ```bash
> NAME="value"
> ```
> No spaces around the `=`. Access the value with `$NAME` or `"${NAME}"`.

**Q5: What does `$1` represent?**
> `$1` is the **first command-line argument** passed to the script. `$2` is the second, `$0` is the script name itself, `$#` is the count, and `$@` is all arguments.

### Intermediate

**Q6: Explain exit codes.**
> Every Linux command returns an **exit code** when it finishes. `0` means success, and any non-zero value (1, 2, 127, etc.) indicates an error. I can check the last command's exit code with `$?`. Exit codes let scripts detect failures and handle them appropriately.

**Q7: When would you use a for loop instead of a while loop?**
> Use a **`for` loop** when I know the number of iterations in advance or when iterating over a fixed list (e.g., `for service in nginx docker ssh`). Use a **`while` loop** when the number of iterations depends on a condition being true (e.g., retry until a service is ready).

**Q8: Why should scripts check command success?**
> Because a failed command can leave the system in an inconsistent state. By checking exit codes (e.g., `git pull || exit 1`), I can stop the script immediately and prevent partial or broken deployments.

**Q9: What are Bash functions used for?**
> Functions group related commands into a reusable, named block. They improve **readability**, **reduce duplication**, and make scripts easier to maintain. Example:
> ```bash
> check_disk() {
>     df -h
> }
> ```

**Q10: How would you make a deployment script reusable?**
> - Use **command-line arguments** for configurable values (e.g., `./deploy.sh backend production`)
> - Use **variables** for paths and names instead of hardcoding
> - Break logic into **functions**
> - Use **environment variables** for secrets
> - Add **error handling** with exit codes
> - Make it **idempotent** (safe to run multiple times)

### Advanced

**Q11: Design a deployment script for a production MERN backend.**
> ```bash
> #!/bin/bash
> set -e   # Exit on any error
> 
> APP_DIR="/opt/mern/backend"
> SERVICE="myapp"
> 
> echo "=== Deploying MERN Backend ==="
> 
> # 1. Pull latest code
> cd "$APP_DIR"
> git pull
> 
> # 2. Install dependencies
> npm install
> 
> # 3. Build (if needed)
> npm run build
> 
> # 4. Restart service
> sudo systemctl restart "$SERVICE"
> 
> # 5. Health check
> if curl -f http://localhost:3000/health; then
>     echo "=== Deployment successful ==="
> else
>     echo "=== Health check failed ==="
>     exit 1
> fi
> ```

**Q12: How would you implement logging in a Bash deployment script?**
> I can log to a file while also displaying to the terminal:
> ```bash
> LOG_FILE="/var/log/deployments.log"
> 
> log() {
>     echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
> }
> 
> log "Starting deployment"
> git pull
> log "Code pulled successfully"
> ```
> Using `tee -a` writes to both the terminal and the log file. Timestamps help track when each step ran.

**Q13: How would you safely roll back if deployment fails?**
> A safe rollback strategy:
> 1. **Backup before deploying** — Copy the current version to a backup location
> 2. **Tag releases in Git** — So I can checkout a previous version
> 3. **Keep the previous build** — Don't delete the last working build
> 4. **On failure, restore the backup** and restart the service
> 5. **Test the rollback** regularly so I know it works
> ```bash
> # Backup current state
> cp -r "$APP_DIR" "${APP_DIR}.backup"
> 
> # If deployment fails, restore
> cp -r "${APP_DIR}.backup" "$APP_DIR"
> sudo systemctl restart "$SERVICE"
> ```

**Q14: What makes a Bash script idempotent?**
> An **idempotent** script can be run multiple times with the same result, without causing errors or duplication. Examples:
> - Check if something exists before creating it (`if [ -d "$DIR" ]`)
> - Only install a package if it's not already installed
> - Use `mkdir -p` (doesn't error if directory exists)
> - Make commands safe to re-run (e.g., `git pull` is naturally idempotent)
> Idempotency is crucial for automation tools and CI/CD.

**Q15: When would you replace Bash automation with a configuration management tool like Ansible?**
> Consider migrating to Ansible when:
> - Managing **many servers** (Bash is hard to maintain at scale)
> - Need **declarative configuration** (describe the desired state, not the steps)
> - Need **idempotency** built-in (Ansible checks state before acting)
> - Need **centralized management** and reporting
> - Need **change tracking** and versioning of infrastructure configs
> - Bash is still great for quick, single-server tasks and Docker entrypoints, but Ansible excels at fleet-wide automation.

---

## 📝 Quiz (10 Questions)

Let me test my understanding:

**1. What does `#!/bin/bash` specify?**
<details>
<summary>Answer</summary>

It specifies the **Bash interpreter** that should execute the script. This is called the shebang line.

</details>

**2. Which command makes a script executable?**
<details>
<summary>Answer</summary>

`chmod +x <script.sh>` — Adds execute permission.

</details>

**3. Which variable contains the first command-line argument?**
<details>
<summary>Answer</summary>

`$1` — The first argument passed to the script.

</details>

**4. What exit code indicates success?**
<details>
<summary>Answer</summary>

**`0`** — Exit code 0 means the command succeeded.

</details>

**5. Which keyword starts a conditional?**
<details>
<summary>Answer</summary>

`if` — For example, `if [ -f ".env" ]; then ... fi`.

</details>

**6. Which keyword repeats over a list?**
<details>
<summary>Answer</summary>

`for` — For example, `for service in nginx docker; do ... done`.

</details>

**7. How do you define a function?**
<details>
<summary>Answer</summary>

`name() { ... }` — For example, `check_disk() { df -h; }`.

</details>

**8. Which command reads user input?**
<details>
<summary>Answer</summary>

`read` — For example, `read -p "Enter name: " NAME`.

</details>

**9. What does `$?` contain?**
<details>
<summary>Answer</summary>

The **exit code of the previous command** — `0` for success, non-zero for error.

</details>

**10. Why should important commands be checked for failure?**
<details>
<summary>Answer</summary>

To **stop or handle failures before continuing**. If a critical step (like `git pull`) fails, I don't want the script to continue into a broken deployment.

</details>

---

## 👨‍💻 Coding Exercise

### server_health.sh

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

**To use it:**
```bash
chmod +x server_health.sh
./server_health.sh
```

**What I learned:**
- `grep -v grep` filters out the grep command itself from the output
- `--state=running` filters services to only running ones
- This script gives a quick snapshot of server health

---

## 🧪 Hands-on Lab

### Lab: Build Your First MERN Deployment Script

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

**Stretch Goal:** Redirect all output to `deployment.log` while also displaying it on the terminal using `tee`:
```bash
./deploy.sh 2>&1 | tee deployment.log
```

---

## 📋 Mini Assignment

### pre_deployment_check.sh

Write a Bash script that:

1. **Verifies at least 2 GB of free disk space**
   ```bash
   FREE_SPACE=$(df --output=avail / | tail -1 | tr -d ' ')
   if [ "$FREE_SPACE" -lt 2000000 ]; then
       echo "FAIL: Less than 2GB free disk space"
   else
       echo "PASS: Sufficient disk space"
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

> This resembles the type of validation often performed in production deployment pipelines.

---

## 📚 Recommended Documentation

I should read the manual pages for deeper understanding:

```bash
man bash          # Bash reference
help test         # File test operators (-f, -d, -z, etc.)
man chmod         # File permissions
```

**Additional resources:**
- GNU Bash Reference Manual
- Bash Guide for Beginners
- Advanced Bash-Scripting Guide

---

## 📖 Summary

Today I learned:

✅ **Bash script structure** — Shebang, variables, and execution.

✅ **Variables and user input** — Storing data and reading from the user.

✅ **Command-line arguments** — `$1`, `$2`, `$@`, `$#`.

✅ **Conditional statements** — `if`/`else` with file tests.

✅ **Loops and functions** — `for`/`while` loops and reusable functions.

✅ **Exit codes** — Checking command success with `$?`.

✅ **Building practical automation scripts** — Deployment, health checks, and pre-deployment validation for MERN deployments.

### Key Takeaways

```
Script → chmod +x → ./script.sh → Automation
```

These scripting fundamentals are the **bridge between manual administration and full DevOps automation**. They will be used repeatedly in upcoming lessons on cron jobs, Ansible, CI/CD, Docker entrypoints, and deployment pipelines.

---

## 🔗 Connecting to the Bigger Picture

Bash scripting connects with everything I've learned:

- **Permissions** → Scripts need `chmod +x` and proper ownership
- **Services** → Scripts manage services with `systemctl`
- **Package Managers** → Scripts install packages with `apt`
- **SSH** → Scripts run remotely over SSH connections
- **Processes** → Scripts manage processes and check their status
- **Networking** → Scripts test ports and endpoints with `curl` and `ss`

Bash is the glue that ties all Linux administration together into automated workflows.

---

*"Bash scripting turns manual Linux administration into repeatable, automated infrastructure management. It's where DevOps automation truly begins."*
