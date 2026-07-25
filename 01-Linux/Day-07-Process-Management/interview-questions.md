# 💼 Interview Questions (Process Management) — with Detailed Answers & Examples

## Beginner

### 1. What is a process?
**Answer:** A process is a **running instance of a program**. When you execute a command or application, Linux loads it into memory and assigns it a unique Process ID (PID). The process has its own memory space, environment, and execution state.

**Example:**
```bash
# When you run node server.js, a new process is created
node server.js
# Linux assigns a PID (e.g., 2511) and tracks CPU/memory usage
```

**Analogy:** A program is like a recipe book (stored on disk). A process is like actually cooking the recipe (executing in memory).

---

### 2. What is a PID?
**Answer:** PID stands for **Process ID** — a unique numeric identifier that Linux assigns to every running process. PIDs are used to manage and control processes (sending signals, monitoring, etc.).

**Example:**
```bash
# View PID of current shell
echo $$
# Output: 1234

# Find PID of a specific process
ps -ef | grep ssh
# Output: root 789 1 0 10:15 ? 00:00:00 sshd
#                    ^ PID is 789
```

---

### 3. What command lists running processes?
**Answer:** The `ps` command lists running processes. Common variations:

| Command | Description |
|---------|-------------|
| `ps` | Processes in the current terminal |
| `ps -ef` | All processes system-wide |
| `ps aux` | Detailed list with CPU/memory |

**Example:**
```bash
ps -ef
# Output:
# UID   PID  PPID  C STIME TTY     TIME CMD
# root    1     0  0 10:15 ?    00:00:01 /sbin/init
# root  789     1  0 10:15 ?    00:00:00 /usr/sbin/sshd
```

---

### 4. What does `top` display?
**Answer:** `top` displays a **real-time, dynamic view** of running processes, including:

- System uptime and load average
- Total tasks (processes) and their states
- CPU usage (user, system, idle, etc.)
- Memory usage (total, free, used, cached)
- Per-process: PID, user, CPU%, MEM%, command

**Example output (abbreviated):**
```
top - 10:30:45 up 2:15, 2 users, load average: 0.15, 0.20, 0.10
Tasks: 123 total, 1 running, 122 sleeping
%Cpu(s): 5.2 us, 2.1 sy, 92.7 id
MiB Mem: 7985 total, 2048 free

  PID USER      %CPU %MEM    COMMAND
 2511 ubuntu    45.0  8.2    node server.js
  789 root       0.5  0.3    sshd
```

---

### 5. What is the difference between a program and a process?
**Answer:**

| Aspect | Program | Process |
|--------|---------|---------|
| Definition | A file stored on disk | A running instance in memory |
| State | Passive (not executing) | Active (executing) |
| Resources | None allocated | CPU, memory, I/O allocated |
| Identifier | Filename (e.g., `node`) | PID (e.g., 2511) |
| Persistence | Exists until deleted | Exists until terminated |
| Example | `/usr/bin/node` | `node server.js` (PID 2511) |

**Analogy:**
- **Program:** A car parked in a garage.
- **Process:** The same car driving on the road.

---

## Intermediate

### 6. Explain foreground vs. background processes.
**Answer:**

**Foreground process:**
- Runs in the terminal and **occupies the shell**.
- You cannot run other commands until it finishes or is stopped.
- Started by simply typing the command.

**Background process:**
- Runs **without blocking the terminal**.
- You can continue running other commands while it executes.
- Started by appending `&` to the command.

**Example:**
```bash
# Foreground — terminal is blocked
sleep 30
# (Terminal is unusable for 30 seconds)

# Background — terminal is free
sleep 30 &
# [1] 3456
# You can immediately run other commands

# List background jobs
jobs
# Output: [1] + Running sleep 30 &

# Bring it to foreground
fg
# sleep 30 resumes in foreground
```

**Use cases:**
- **Background:** Long-running tasks (downloads, scripts, servers).
- **Foreground:** Interactive commands (editors, quick tasks).

---

### 7. What is the difference between `kill` and `kill -9`?
**Answer:**

| Signal | Command | Behavior |
|--------|---------|----------|
| SIGTERM (15) | `kill PID` | **Graceful termination** — process can clean up resources, close files, save state |
| SIGKILL (9) | `kill -9 PID` | **Force termination** — kernel immediately kills the process; no cleanup possible |

**Example:**
```bash
# Graceful — application gets a chance to clean up
kill 2511
# Node.js can close database connections, write logs, etc.

# Force — immediate death, no cleanup
kill -9 2511
# Node.js dies instantly; connections may drop abruptly
```

**When to use each:**
```bash
# 1. Try graceful termination first
kill 2511

# 2. Wait a few seconds, check if it's still running
ps -ef | grep 2511

# 3. Only use force if the process is unresponsive
kill -9 2511
```

> ⚠️ Never use `kill -9` as your first option. It can cause data corruption or orphaned resources.

---

### 8. How would you find a running Node.js process?
**Answer:**

**Method 1 — Using `ps` and `grep`:**
```bash
ps -ef | grep node
```

**Example output:**
```
ubuntu    2511     1  0 10:30 ?  00:00:45 node /var/www/app/server.js
ubuntu    2890  2511  0 10:35 ?  00:00:02 node /var/www/app/worker.js
```

**Method 2 — Using `pgrep` (if available):**
```bash
pgrep -l node
# Output: 2511 node
```

**Method 3 — Using `top` or `htop` interactively:**
```bash
top
# Then press 'u' and type 'ubuntu' to filter by user
```

**Method 4 — Using `pidof`:**
```bash
pidof node
# Output: 2511 2890 (multiple PIDs if multiple instances)
```

---

### 9. What is load average?
**Answer:** Load average represents the **average number of processes waiting for CPU time** over specific time intervals. It's displayed by `uptime` and `top`.

**Example:**
```bash
uptime
# Output: load average: 0.45, 0.60, 0.72
```

**Interpretation:**
| Value | Meaning |
|-------|---------|
| 0.45 | Average load over the last **1 minute** |
| 0.60 | Average load over the last **5 minutes** |
| 0.72 | Average load over the last **15 minutes** |

**How to interpret:**
- **Load < # of CPU cores:** System is not fully utilized.
- **Load ≈ # of CPU cores:** System is running at capacity.
- **Load > # of CPU cores:** System may be overloaded.

**Example with a 4-core server:**
```
# Healthy — system is underutilized
load average: 1.2, 1.5, 1.8

# At capacity — all cores busy
load average: 3.8, 4.0, 4.1

# Overloaded — processes are waiting
load average: 8.5, 7.2, 5.0
```

---

### 10. What does `jobs` do?
**Answer:** The `jobs` command **lists all background jobs** started in the current terminal session, showing their job number, status, and command.

**Example:**
```bash
sleep 300 &
# [1] 3456
sleep 120 &
# [2] 3457
sleep 60 &
# [3] 3458

jobs
# Output:
# [1]  Running    sleep 300 &
# [2]-  Running    sleep 120 &
# [3]+  Running    sleep 60 &

# Bring job 2 to foreground
fg %2

# Stop foreground job
Ctrl + C
```

**Job status indicators:**
- `+` — Current default job (used by `fg` without argument)
- `-` — Previous default job
- `Running` — Currently executing
- `Stopped` — Suspended (Ctrl+Z)

---

## Advanced

### 11. A Node.js process uses 100% CPU. How would you investigate?
**Answer:**

**Step 1 — Identify the problematic process:**
```bash
# Find the Node.js PID consuming CPU
top
# Press 'P' to sort by CPU usage
```

**Step 2 — Get more details:**
```bash
# Check exact command and arguments
ps -ef | grep node

# See child threads
top -H -p <PID>

# Check file descriptors (could be leaking)
ls -la /proc/<PID>/fd | wc -l
```

**Step 3 — Capture a profile:**
```bash
# Send SIGUSR1 to trigger a Node.js debugger/dump
kill -USR1 <PID>

# Or use Node's built-in inspector
node --inspect server.js
```

**Step 4 — Check logs:**
```bash
# Application logs
tail -100 /var/log/app/app.log

# System logs for out-of-memory (OOM) killer
dmesg | grep -i oom
```

**Step 5 — Check for common causes:**

| Cause | What to check |
|-------|---------------|
| Infinite loop | Recently deployed code changes |
| Memory leak | `top` memory growth over time |
| Too many requests | `ss -tuln \| grep 3000` for connection count |
| Blocked I/O | Check database and API response times |
| Garbage collection | Node.js GC logs |

**Step 6 — Take action:**
```bash
# If necessary, restart gracefully
kill <PID>

# Before restarting, consider:
# - Did a recent deploy cause this?
# - Is traffic unusually high?
# - Is the database slow?
```

---

### 12. What are zombie processes?
**Answer:** A **zombie process** is a process that has finished executing but still has an entry in the process table because its parent hasn't read its exit status. Zombies appear as `Z` in process state.

**Example:**
```bash
# Find zombie processes (state Z)
ps aux | grep Z
# Output: ubuntu 3456 0.0 0.0 0 0 ? Z 10:30 0:00 [defunct]
```

**Why zombies happen:**
```
Parent creates child process
        ↓
Child finishes execution
        ↓
Child becomes zombie (waiting for parent to read exit code)
        ↓
Parent reads exit code via wait() or waitpid()
        ↓
Zombie is removed from process table
```

**Problems with zombies:**
- They consume a slot in the process table (limited resource).
- Too many zombies can exhaust the process table.
- They cannot be killed with `kill -9` (already dead).

**How to fix zombie processes:**
```bash
# 1. Kill the parent process (zombies will be reaped by init)
kill <parent-PID>

# 2. If parent won't die, restart the parent's service
sudo systemctl restart parent-service

# 3. In code, ensure parent calls wait() for child processes
```

---

### 13. Why should SIGTERM usually be preferred over SIGKILL?
**Answer:** SIGTERM (signal 15) allows the process to **clean up gracefully**, while SIGKILL (signal 9) **terminates immediately** without cleanup.

**What SIGTERM allows:**
```javascript
// Node.js example — SIGTERM handler
process.on('SIGTERM', () => {
  console.log('Shutting down gracefully...');
  
  // 1. Close database connections
  db.close();

  // 2. Finish processing current requests
  server.close(() => {
    // 3. Write final logs
    logger.info('Server shutdown complete');
    
    // 4. Exit cleanly
    process.exit(0);
  });
});
```

**Risks of using SIGKILL first:**
- **Data corruption:** Open files may be corrupted.
- **Orphaned connections:** Database connections left open.
- **Incomplete writes:** Log entries or responses may be truncated.
- **Resource leaks:** Shared memory, semaphores, temp files not cleaned.

**Best practice workflow:**
```bash
# Step 1: Graceful termination (SIGTERM)
kill <PID>

# Step 2: Wait 5-10 seconds
sleep 5

# Step 3: Check if process still exists
ps -ef | grep <PID>

# Step 4: Only force kill if still running
kill -9 <PID>
```

---

### 14. Describe a production process troubleshooting workflow.
**Answer:**

**The PROD workflow (5 steps):**

**P — Problem Identification:**
```bash
# What's the symptom?
# - Application slow?
# - Errors in logs?
# - Service down?

# Check basic system health
uptime             # System load
free -h            # Memory usage
df -h              # Disk usage
ss -tuln           # Listening ports
```

**R — Resource Analysis:**
```bash
# Identify resource bottlenecks
top                # CPU and memory per process
ps aux --sort=-%cpu | head -10   # Top CPU consumers
ps aux --sort=-%mem | head -10   # Top memory consumers

# Check for specific application processes
ps -ef | grep node
```

**O — Observe Logs:**
```bash
# Application logs
tail -100 /var/log/app/app.log

# System logs
journalctl -u app-service --since "10 minutes ago"

# Nginx/Apache logs (if applicable)
tail -100 /var/log/nginx/access.log
tail -100 /var/log/nginx/error.log
```

**D — Diagnose Root Cause:**
```
Is the process running?
├── No → Start it, check why it stopped (OOM? Crash?)
└── Yes → Is it using high resources?
    ├── High CPU → Code issue, traffic spike, infinite loop?
    ├── High memory → Memory leak, cache growth?
    ├── High I/O → Disk bottleneck, database issue?
    └── Normal → Network issue? DNS? External dependency?
```

**Decision Framework:**
```bash
# Check if process is responding
curl -I http://localhost:3000/health

# Check if port is open
ss -tuln | grep 3000

# If not responding, check reason
# Connection refused → Process crashed
# Timeout → Process hung, network issue
# 500 error → Application error
```

**Action:**
```bash
# If process crashed:
# 1. Read logs
# 2. Fix the issue
# 3. Restart safely

# If process is hung/kill it gracefully first
kill <PID>

# Wait, force kill if needed
sleep 5
kill -9 <PID>

# Restart
sudo systemctl restart app
```

---

### 15. How would you diagnose repeated application crashes?
**Answer:**

**Step 1 — Check system logs for OOM (Out of Memory):**
```bash
# Check if the kernel killed the process
dmesg | grep -i "killed process"
# Output: [12345.678] Killed process 2511 (node) total-vm:1234567kB

# Check system memory at crash time
free -h
```

**Step 2 — Check application crash logs:**
```bash
# Application error logs
tail -200 /var/log/app/error.log

# Journal logs (if managed by systemd)
journalctl -u app-service --since "24 hours ago" | grep -i error

# Core dumps (if enabled)
ls -la /var/crash/
```

**Step 3 — Check resource limits:**
```bash
# Check system limits for the user
ulimit -a

# Check if open file limit is too low
cat /proc/<PID>/limits | grep "open files"
```

**Step 4 — Monitor over time:**
```bash
# Use a simple loop to track memory usage
while true; do
  ps -o pid,%cpu,%mem,rss,command -p $(pgrep -d, -x node) >> /tmp/node_monitor.log
  sleep 10
done
```

**Step 5 — Common crash patterns:**

| Pattern | Likely Cause | Fix |
|---------|--------------|-----|
| Crashes at same time daily | Cron job conflict | Check cron, adjust schedule |
| Crashes under high traffic | Insufficient resources | Scale up, add more instances |
| Crashes after memory grows | Memory leak | Profile heap, fix leak |
| Crashes with segfault | Native module issue | Rebuild or update modules |
| Restarts immediately | Health check misconfigured | Fix health check endpoint |

**Step 6 — Implement monitoring:**
```bash
# Add a simple health check script
#!/bin/bash
# health_check.sh
URL="http://localhost:3000/health"
if ! curl -f "$URL" > /dev/null 2>&1; then
  echo "$(date): Application down, restarting..." >> /var/log/app/restarts.log
  sudo systemctl restart app-service
fi
