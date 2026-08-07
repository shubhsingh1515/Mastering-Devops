# Day 13 — Assignment: Linux Log Management & Troubleshooting

## Objective
Apply what you've learned about log management to locate, analyze, and troubleshoot logs for a production MERN application.

---

## Part 1: Create Your Log Workspace

Create a directory for your practice:
```bash
mkdir -p ~/devops-course/day13
cd ~/devops-course/day13
```

Create a sample application log:
```bash
cat > app.log <<EOF
INFO Server started
INFO Database connected
WARN Slow query detected on /api/users
INFO Request completed
ERROR User authentication failed for user 42
INFO Request completed
WARN High memory usage detected
ERROR MongoDB connection timeout
INFO Request completed
INFO Deployment successful
EOF
```

---

## Part 2: Basic Log Analysis

### Exercise 1 — Look at the log
```bash
cat app.log
```
View the first and last lines:
```bash
head app.log
tail app.log
```

### Exercise 2 — Search the log
Find all errors:
```bash
grep ERROR app.log
```
Case-insensitive search:
```bash
grep -i warn app.log
```
Count errors:
```bash
grep -c "ERROR" app.log
```

### Exercise 3 — Follow the log live
```bash
tail -f app.log
```
In **another terminal**, append a new entry:
```bash
echo "INFO New request completed" >> app.log
```
Observe the new entry appear in real time. Press `Ctrl+C` to stop following.

---

## Part 3: Coding Exercise — log_summary.sh

Create `log_summary.sh`:
```bash
#!/bin/bash

LOGFILE="app.log"

echo "===== Total Lines ====="
wc -l "$LOGFILE"

echo
echo "===== Errors ====="
grep -c "ERROR" "$LOGFILE"

echo
echo "===== Warnings ====="
grep -c "WARN" "$LOGFILE"

echo
echo "===== Last Five Entries ====="
tail -5 "$LOGFILE"
```

Make it executable and run it:
```bash
chmod +x log_summary.sh
./log_summary.sh
```

---

## Part 4: Hands-on Lab — Investigate a Failed MERN Deployment

Create a deployment log:
```bash
cat > deploy.log <<EOF
INFO Starting deployment
INFO Stopping old containers
ERROR Failed to pull image: connection refused
WARN Retrying image pull
INFO Pulled image successfully
INFO Starting new containers
ERROR Container health check failed on worker-1
WARN Restarting worker-1
INFO Worker-1 healthy after restart
INFO Deployment complete
EOF
```

### Tasks

1. **Count ERROR messages:**
   ```bash
   grep -c "ERROR" deploy.log
   ```

2. **Display only WARN messages:**
   ```bash
   grep "WARN" deploy.log
   ```

3. **Show the last 10 lines:**
   ```bash
   tail -10 deploy.log
   ```

4. **Follow the log live:**
   ```bash
   tail -f deploy.log
   ```
   Append a new line in another terminal and watch it appear.

5. **Write a short incident summary** identifying the probable root cause.

---

## Part 5: Mini Assignment — Logging Plan for a Production MERN Application

Design a logging plan that includes:

- Nginx access logs
- Nginx error logs
- Node.js application logs
- MongoDB logs
- Cron job logs
- Deployment logs

For **each** log source, explain:

1. **What information** it should contain.
2. **Who** would use it.
3. **How long** it should be retained.
4. **Why log rotation** is important for it.

Use the table format below:

| Log Source | Information | Who Uses It | Retention |
|------------|-------------|-------------|-----------|
| Nginx access logs | Client IPs, URLs, status codes, response times | DevOps, developers | 30 days |
| Nginx error logs | Failed requests, upstream errors, SSL issues | DevOps, developers | 30 days |
| Node.js app logs | Request logs, errors, warnings, business events | Developers, SREs | 90 days |
| MongoDB logs | Slow queries, replication, connection errors | DBA, DevOps | 30 days |
| Cron job logs | Scheduled job success/failure output | DevOps, sysadmins | 14 days |
| Deployment logs | Build/deploy steps, versions, outcomes | DevOps, release engineers | 90 days |

---

## Part 6: Knowledge Check

Answer the following questions:

1. **What is the difference between `tail` and `tail -f`?**

   <details>
   <summary>Answer</summary>

   `tail` shows the last lines once and exits. `tail -f` keeps running and **follows** the file, displaying new entries in real time.
   </details>

2. **Why should you avoid `cat` on a huge log file?**

   <details>
   <summary>Answer</summary>

   It dumps the entire file to the terminal, which is slow, floods the screen, and can consume significant memory. Use `tail`, `head`, `less`, or `grep` instead.
   </details>

3. **What does `grep -c "ERROR" app.log` output?**

   <details>
   <summary>Answer</summary>

   The **count** of lines in `app.log` that contain the text `ERROR`.
   </details>

4. **How would you view the last 50 lines of a systemd service's logs?**

   <details>
   <summary>Answer</summary>

   `journalctl -u myapp -n 50`
   </details>

5. **Why is log rotation important in production?**

   <details>
   <summary>Answer</summary>

   It prevents logs from growing indefinitely and filling up the disk, which would cause write failures and outages. It compresses old logs and removes very old ones automatically.
   </details>

---

## Submission Checklist

- [ ] Created `~/devops-course/day13/` workspace
- [ ] Created the sample `app.log`
- [ ] Used `head`, `tail`, `grep`, and `tail -f` to analyze it
- [ ] Created and ran `log_summary.sh`
- [ ] Investigated `deploy.log` (counted errors, filtered warnings, tailed, followed live)
- [ ] Wrote an incident summary
- [ ] Created the MERN production logging plan
- [ ] Answered the knowledge check questions
