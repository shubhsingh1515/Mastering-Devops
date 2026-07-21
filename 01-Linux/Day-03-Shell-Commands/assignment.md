# 👨‍💻 Coding / Scripting Exercise

## 13. 👨‍💻 Coding / Scripting Exercise

Create the following structure:

```
deployment/
├── backend/
│   ├── app.log
│   └── server.log
├── frontend/
└── backups/
```

Then:

1. Add two lines of text to `app.log`.
2. Copy `app.log` into `backups/`.
3. Rename `server.log` to `api.log`.
4. Use `find` to locate all `.log` files.
5. Display the last line of `api.log`.

**Expected commands:**

```bash
# Create the structure
mkdir -p deployment/backend deployment/frontend deployment/backups
touch deployment/backend/app.log deployment/backend/server.log

# Add text to app.log
echo "Server started on port 5000" >> deployment/backend/app.log
echo "Database connected successfully" >> deployment/backend/app.log

# Copy app.log to backups
cp deployment/backend/app.log deployment/backups/

# Rename server.log to api.log
mv deployment/backend/server.log deployment/backend/api.log

# Find all .log files
find deployment -name "*.log"

# Display the last line of api.log
tail -1 deployment/backend/api.log
```

---

# 🔬 Hands-on Lab

## 14. 🧪 Hands-on Lab
### Lab: Simulate Basic Production Log Analysis

**Goal:** Build the habit of using shell tools to inspect and troubleshoot systems.

**Instructions:**

1. Create a file named `application.log`.
2. Add at least 20 log entries using:
   ```bash
   echo "Log entry 1" >> application.log
   echo "Log entry 2" >> application.log
   # ... repeat up to 20
   ```
3. Use:
   - `head` — view the first few lines
   - `tail` — view the last few lines
   - `less` — navigate through the log page by page
4. Search for all `.log` files in your practice directory.
5. Check the directory's size using `du -sh`.
6. Check overall disk usage with `df -h`.

**Expected commands:**

```bash
# Create the log file
touch application.log

# Add 20 log entries
echo "Log entry 1" >> application.log
echo "Log entry 2" >> application.log
echo "Log entry 3" >> application.log
echo "Log entry 4" >> application.log
echo "Log entry 5" >> application.log
echo "Log entry 6" >> application.log
echo "Log entry 7" >> application.log
echo "Log entry 8" >> application.log
echo "Log entry 9" >> application.log
echo "Log entry 10" >> application.log
echo "Log entry 11" >> application.log
echo "Log entry 12" >> application.log
echo "Log entry 13" >> application.log
echo "Log entry 14" >> application.log
echo "Log entry 15" >> application.log
echo "Log entry 16" >> application.log
echo "Log entry 17" >> application.log
echo "Log entry 18" >> application.log
echo "Log entry 19" >> application.log
echo "Log entry 20" >> application.log

# View first 5 lines
head -5 application.log

# View last 3 lines
tail -3 application.log

# Navigate through the file (press q to quit)
less application.log

# Find all .log files
find . -name "*.log"

# Check directory size
du -sh .

# Check disk usage
df -h
```

---

# 📋 Mini Assignment

## 15. 📋 Mini Assignment

Imagine your MERN backend has stopped responding.

Write a troubleshooting checklist using only the commands learned so far. Include:

1. **How you'll navigate to the application directory.**
2. **How you'll inspect recent logs.**
3. **How you'll verify disk space.**
4. **How you'll locate configuration or log files.**
5. **Which commands you'll use and why.**

---

### ✅ Sample Solution: MERN Backend Troubleshooting Checklist

**Step 1 — Navigate to the application directory**

```bash
cd /home/ubuntu/mern-backend
```
> `cd` changes the current working directory to the application location.

**Step 2 — Inspect recent logs**

```bash
tail -100 /var/log/mern/backend.log
```
> `tail -100` shows the last 100 lines of the log file, giving context on what happened before the crash.

**Step 3 — Watch logs in real time (optional, if issue is ongoing)**

```bash
tail -f /var/log/mern/backend.log
```
> `tail -f` follows the log file in real time so you can see errors as they occur while reproducing the issue. Press `Ctrl + C` to stop.

**Step 4 — Verify disk space**

```bash
df -h
```
> `df -h` shows free disk space on all mounted filesystems in human-readable format. A full disk can cause application crashes.

```bash
du -sh /var/log/mern
```
> `du -sh` checks the size of the MERN log directory specifically, to see if logs are consuming too much space.

**Step 5 — Locate configuration and log files**

```bash
find /home/ubuntu -name "*.env"
```
> `find` searches for all `.env` configuration files in the project directory.

```bash
find /home/ubuntu -name "*.log"
```
> `find` locates all log files that may contain error information.

**Step 6 — Check for running processes**

```bash
ps aux | grep node
```
> `ps aux` lists all running processes, and the pipe `|` sends the output to `grep node` to filter for Node.js processes. This verifies if the application is still running.

---

### Why These Commands?

| Command | Why It's Used |
|---------|---------------|
| `cd` | Navigate to the correct directory |
| `tail -100` | View recent log entries for crash context |
| `tail -f` | Monitor logs live while reproducing the issue |
| `df -h` | Check if disk space is exhausted |
| `du -sh` | Check specific directory sizes (e.g., logs) |
| `find` | Locate configuration and log files across the filesystem |
| `ps aux \| grep` | Check if the Node.js process is still running |
