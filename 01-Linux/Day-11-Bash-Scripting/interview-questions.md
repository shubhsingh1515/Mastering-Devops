# Day 11 — Interview Questions: Bash Scripting Fundamentals

## Beginner

### Q1: What is Bash?
**Bash (Bourne Again Shell)** is both a command-line shell and a scripting language. It's the default shell on most Linux distributions. Bash lets me run commands interactively in the terminal and write scripts to automate repetitive tasks. It supports variables, conditionals, loops, functions, and access to all Linux command-line tools.

### Q2: What is a shebang?
A **shebang** (`#!/bin/bash`) is the first line of a script that tells the operating system which interpreter to use to execute the script. The `#!` (hash + bang) is followed by the absolute path to the interpreter, in this case `/bin/bash`. This makes the script self-contained and executable.

### Q3: How do you execute a Bash script?
Two ways:
1. Make it executable and run directly:
   ```bash
   chmod +x script.sh
   ./script.sh
   ```
2. Run it explicitly with bash:
   ```bash
   bash script.sh
   ```

### Q4: How do you define a variable?
```bash
NAME="value"
```
Rules:
- No spaces around the `=` sign
- Case-sensitive (`NAME` ≠ `name`)
- Access with `$NAME` or `"${NAME}"`
- Variables are text by default; use `$(( ))` for arithmetic

### Q5: What does `$1` represent?
`$1` is the **first command-line argument** passed to the script. Example: running `./deploy.sh backend production` → `$1` = `backend`, `$2` = `production`, `$0` = `./deploy.sh`, `$#` = `2`.

---

## Intermediate

### Q6: Explain exit codes.
Every Linux command returns an **exit code** when it finishes. **`0` means success**, and any **non-zero value** (1, 2, 127, etc.) indicates an error. I can check the last command's exit code with `$?`. Scripts use exit codes to detect failures and make decisions, e.g., `git pull || exit 1` stops the script if the pull fails.

### Q7: When would you use a for loop instead of a while loop?
- **`for` loop** — When I know the number of iterations in advance or iterating over a fixed list (e.g., `for service in nginx docker ssh`).
- **`while` loop** — When the number of iterations depends on a condition (e.g., retry until a service is ready, or count until a limit is reached).

### Q8: Why should scripts check command success?
Checking command success prevents **partial or broken deployments**. If a critical command fails (like `git pull` or `npm install`) but the script continues, it might restart a service with incomplete code. Using `cmd || exit 1` or `set -e` ensures the script stops at the first failure, keeping the system in a consistent state.

### Q9: What are Bash functions used for?
Functions group related commands into a **reusable, named block**. Benefits:
- **Readability** — Scripts look cleaner
- **Reduces duplication** — Write once, use many times
- **Maintainability** — Fix in one place
- **Reusability** — Call the same logic from multiple places

Example:
```bash
check_disk() {
    df -h
}
check_disk
```

### Q10: How would you make a deployment script reusable?
1. **Use command-line arguments** for configurable values (`./deploy.sh backend production`)
2. **Use variables** for paths and names instead of hardcoding
3. **Break logic into functions**
4. **Use environment variables** for secrets (never hardcode passwords)
5. **Add error handling** with exit codes
6. **Make it idempotent** — Safe to run multiple times
7. **Add logging** — Record what the script did and when
8. **Add documentation** — Comments and usage instructions

---

## Advanced

### Q11: Design a deployment script for a production MERN backend.
```bash
#!/bin/bash
set -e   # Exit on any error

APP_DIR="/opt/mern/backend"
SERVICE="myapp"
LOG_FILE="/var/log/deployments.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}

log "=== Deploying MERN Backend ==="

# Check prerequisites
if [ ! -d "$APP_DIR" ]; then
    log "ERROR: $APP_DIR does not exist"
    exit 1
fi

# Pull latest code
cd "$APP_DIR"
git pull
log "Code pulled"

# Install dependencies
npm install
log "Dependencies installed"

# Restart service
sudo systemctl restart "$SERVICE"
log "Service restarted"

# Health check
if curl -f http://localhost:3000/health; then
    log "=== Deployment successful ==="
else
    log "=== Health check FAILED ==="
    exit 1
fi
```

### Q12: How would you implement logging in a Bash deployment script?
Use a `log()` function that timestamps output and writes to both terminal and file:
```bash
LOG_FILE="/var/log/deployments.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}

log "Starting deployment"
git pull
log "Code pulled successfully"
```
Benefits:
- **Timestamps** — Track when each step ran
- **Persistent records** — Review past deployments
- **audit trail** — Who/what/when of changes
- Alternative: redirect all output with `./deploy.sh | tee -a deployment.log`

### Q13: How would you safely roll back if deployment fails?
A safe rollback strategy:
1. **Backup before deploying** — Keep a copy of the current version
2. **Tag releases in Git** — So I can checkout any previous version
3. **Keep the previous build** — Don't delete the last working build
4. **On failure, restore**:
   ```bash
   # Backup current state
   cp -r "$APP_DIR" "${APP_DIR}.backup"

   # Attempt deployment...

   # If deployment fails, roll back
   rm -rf "$APP_DIR"
   cp -r "${APP_DIR}.backup" "$APP_DIR"
   sudo systemctl restart "$SERVICE"
   ```
5. **Test the rollback** regularly
6. **Use systemd `Restart=always`** so the app comes back even if it crashes

### Q14: What makes a Bash script idempotent?
An **idempotent** script produces the same result no matter how many times it's run. Techniques:
1. **Check before creating**: `if [ ! -d "$DIR" ]; then mkdir "$DIR"; fi`
2. **Use mkdir -p** — No error if it exists
3. **Check if command exists**: `command -v git || apt install git`
4. **Check if service is running** before restarting
5. **Use git pull** — Naturally idempotent (safe to re-run)
6. **Only copy if changed**: `rsync` instead of `cp`

Idempotency is essential for automation — tools like Ansible enforce it by checking state before acting.

### Q15: When would you replace Bash automation with a configuration management tool like Ansible?
Migrate to Ansible when:
1. **Managing many servers** — Bash doesn't scale well for fleet management
2. **Need declarative configuration** — Describe desired state, not step-by-step commands
3. **Need built-in idempotency** — Ansible checks state before making changes
4. **Need centralized management** — Run commands across hundreds of servers
5. **Need auditability** — Track changes and versions of infrastructure config
6. **Need integration** — With cloud providers, inventories, and reporting

Bash is still great for:
- Quick single-server tasks
- Docker entrypoints and container init scripts
- Simple cron jobs
- Short utilities

Ansible excels at repeatable, fleet-wide, declarative infrastructure management.
