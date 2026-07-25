# 👨‍💻 Coding / Scripting Exercise

## 13. 👨‍💻 Coding / Scripting Exercise

Create a script named `process_report.sh`:

```bash
#!/bin/bash

echo "=== Hostname ==="
hostname

echo
echo "=== Uptime ==="
uptime

echo
echo "=== Top 10 Processes ==="
ps -ef | head

echo
echo "=== Listening Ports ==="
ss -tuln
```

Make it executable:

```bash
chmod +x process_report.sh
```

Run:

```bash
./process_report.sh
```

**Expected output (example):**

```
=== Hostname ===
ubuntu-server

=== Uptime ===
10:30:45 up 2:15, 2 users, load average: 0.15, 0.20, 0.10

=== Top 10 Processes ===
UID   PID  PPID  C STIME TTY     TIME CMD
root    1     0  0 08:15 ?    00:00:01 /sbin/init
root  123     1  0 08:15 ?    00:00:00 /lib/systemd/systemd-journald
root  456     1  0 08:15 ?    00:00:00 /usr/sbin/sshd
...
ubuntu 2511 2456  0 10:00 ?   00:00:45 node /var/www/app/server.js

=== Listening Ports ===
LISTEN 0 128 0.0.0.0:22 0.0.0.0:*
LISTEN 0 128 0.0.0.0:80 0.0.0.0:*
LISTEN 0 128 0.0.0.0:443 0.0.0.0:*
```

---

# 🔬 Hands-on Lab

## 14. 🧪 Hands-on Lab
### Lab: Investigate a Simulated High-Load Server

**Goal:** Become comfortable identifying and managing processes.

### Instructions

**Step 1** — Start a background process:
```bash
sleep 300 &
```

**Step 2** — Find its PID:
```bash
# Method 1 — list all jobs
jobs

# Method 2 — find it with ps
ps -ef | grep sleep
```

**Step 3** — View it with:
```bash
ps -ef | grep sleep
```
Expected: `ubuntu 3456 1234 0 10:30 pts/0 00:00:00 sleep 300`

**Step 4** — Open `top` and observe running processes:
```bash
top
```
Note: Press `q` to exit. Observe CPU%, MEM%, and load averages.

**Step 5** — Gracefully terminate the sleep process:
```bash
kill <PID>
```
Replace `<PID>` with the actual process ID from step 2.

**Step 6** — Confirm it no longer appears in the process list:
```bash
ps -ef | grep sleep
```
Expected: No output (or just the `grep` command itself).

**Step 7** — Practice with multiple background jobs:
```bash
sleep 60 &
sleep 120 &
sleep 180 &

# List all jobs
jobs

# Bring specific job to foreground
fg %2

# Stop it
Ctrl + C
```

### Lab Documentation Template

```markdown
# Process Investigation Lab

## System Information
- Hostname: [your hostname]
- Uptime: [uptime output]
- Load average: [load from uptime]

## Process Discovery
- Command used to find PID: ps -ef | grep sleep
- PID found: [PID]
- Process name: sleep
- Process state: S (sleeping)

## Process Termination
- Command used: kill [PID]
- SIGTERM sent? Yes/No
- SIGKILL needed? Yes/No
- Process removed from list? Yes/No

## Observations
[Any notes about what you observed]
```

---

# 📋 Mini Assignment

## 15. 📋 Mini Assignment
### MERN Backend Process Troubleshooting Plan

Your MERN backend has become unresponsive.

Write a troubleshooting plan using the commands learned so far. Include:

1. **How you'll verify the Node.js process is running.**
2. **How you'll check CPU and memory usage.**
3. **How you'll inspect network ports.**
4. **How you'll examine recent logs.**
5. **Under what circumstances you would terminate and restart the process.**

---

### ✅ Sample Solution: MERN Backend Troubleshooting Plan

#### Step 1 — Verify Node.js Process Is Running

```bash
# Check if Node.js process exists
ps -ef | grep node

# Expected output if running:
# ubuntu 2511 1 0 10:30 ? 00:00:45 node /var/www/app/server.js

# If nothing shows up, the process has crashed or never started
# If multiple entries show, there may be duplicate instances
```

**Interpretation:**
- **Single instance found:** Process is running — check resource usage next.
- **Multiple instances found:** Previous instances may not have been cleaned up.
- **Nothing found:** Process crashed — check logs.

---

#### Step 2 — Check CPU and Memory Usage

```bash
# Live monitoring
top

# Sort by CPU usage (press P in top)
# Sort by memory usage (press M in top)

# Quick snapshot — top CPU consumers
ps aux --sort=-%cpu | head -6

# Quick snapshot — top memory consumers
ps aux --sort=-%mem | head -6

# System memory overview
free -h

# Swap usage
swapon --show
```

**What to look for:**
| Observation | Possible Issue |
|-------------|----------------|
| Node.js uses 95%+ CPU | Infinite loop, traffic spike, or inefficient code |
| Memory grows continuously | Memory leak |
| Swap usage > 0 | System is running out of RAM |
| Load average >> CPU cores | System is overloaded |

---

#### Step 3 — Inspect Network Ports

```bash
# Check if the application is listening on its port
ss -tuln | grep 3000

# Expected:
# LISTEN 0 128 0.0.0.0:3000 0.0.0.0:*

# Test the application locally
curl http://localhost:3000/health

# Check connection count
ss -tun | grep :3000 | wc -l
```

**Troubleshooting:**
- **Port 3000 not listening:** Application crashed or not started.
- **Port 3000 listening but curl fails:** Application hung or in bad state.
- **Too many connections (>1000):** May be under DDoS or connection leak.

---

#### Step 4 — Examine Recent Logs

```bash
# Application logs
tail -100 /var/log/app/app.log
tail -100 /var/log/app/error.log

# System logs (for OOM killer or system issues)
dmesg | tail -20
journalctl -u app-service --since "1 hour ago"

# Nginx logs (if behind reverse proxy)
tail -100 /var/log/nginx/access.log
tail -100 /var/log/nginx/error.log

# Check for out of memory kills
dmesg | grep -i "killed process"
```

**Common log patterns:**
| Log Message | Meaning |
|-------------|---------|
| `FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed` | Out of memory |
| `Error: EMFILE: too many open files` | File descriptor leak |
| `Error: connect ECONNREFUSED 127.0.0.1:27017` | MongoDB is down |
| `Killed process 2511 (node)` | OOM killer terminated the process |

---

#### Step 5 — When to Terminate and Restart

**Terminate and restart under these circumstances:**

| Condition | Action | Reason |
|-----------|--------|--------|
| Process consuming 100% CPU for >5 minutes | `kill <PID>` then restart | Likely stuck in infinite loop |
| Memory growing unbounded | `kill <PID>` then restart | Memory leak — needs investigation |
| Process not responding to requests | `kill -9 <PID>` if graceful fails | Hung process, force terminate |
| Multiple duplicate instances found | `pkill node` then restart | Clean up orphaned processes |
| OOM killer terminated the process | Restart, then investigate | Need more RAM or fix memory leak |

**Restart procedure:**

```bash
# Step 1: Graceful termination first
kill <PID>

# Step 2: Wait 5 seconds
sleep 5

# Step 3: Check if process still exists
ps -ef | grep node

# Step 4: Force kill if needed
kill -9 <PID>

# Step 5: Start the application
cd /var/www/app
sudo -u nodeapp node server.js &
```

**Before restarting, always consider:**
1. Did a recent deployment cause this?
2. Is traffic abnormally high?
3. Are dependent services (database, cache) healthy?
4. Will restarting cause downtime for users?

**Post-restart checklist:**

```bash
# 1. Verify process is running
ps -ef | grep node

# 2. Verify port is listening
ss -tuln | grep 3000

# 3. Test the API
curl http://localhost:3000/health

# 4. Monitor for first 5 minutes
watch -n 10 'ps -ef | grep node | grep -v grep'
```

### Quick Reference: Troubleshooting Decision Tree

```
Application unresponsive?
│
├── Check process: ps -ef | grep node
│   ├── Not running → Restart, check logs
│   └── Running → Check resources
│
├── Check CPU/MEM: top
│   ├── High CPU → kill + investigate code
│   ├── High memory → kill + check for leak
│   └── Normal → Check network
│
├── Check port: ss -tuln | grep 3000
│   ├── Not listening → Restart app
│   ├── Listening but slow → Check connections
│   └── Normal → Check logs
│
├── Check logs: tail -100 /var/log/app/error.log
│   ├── OOM → Increase memory or fix leak
│   ├── ECONNREFUSED → Database down
│   └── No errors → Check external dependencies
│
└── Restart if:
    ├── Process hung for >5 minutes
    ├── Memory leak detected
    ├── Multiple duplicate processes
    └── After root cause is identified
