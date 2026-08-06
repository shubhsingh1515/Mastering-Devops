# Day 12 — Assignment: Scheduling Tasks with Cron

## Objective
Apply what you've learned about cron scheduling to automate MERN server maintenance — backups, log cleanup, health checks, and production maintenance scheduling.

---

## Part 1: Create Your Cron Workspace

Create a directory for your scripts:
```bash
mkdir -p ~/devops-course/day12
cd ~/devops-course/day12
```

---

## Part 2: Basic Cron Practice

### Exercise 1 — Test a cron job
Add a simple test job to verify cron is working:
```bash
crontab -e
```
Add this line:
```
*/2 * * * * echo "Cron works" >> ~/cron-test.log
```
Wait a few minutes, then verify:
```bash
cat ~/cron-test.log
```
You should see the message repeated every 2 minutes.

### Exercise 2 — Remove the test job
After confirming it works, edit the crontab and remove the test line:
```bash
crontab -e
```
Delete the line and save.

---

## Part 3: Coding Exercise — health_check.sh

Create `health_check.sh`:
```bash
#!/bin/bash

URL="http://localhost:3000/health"

if curl -fs "$URL" >/dev/null; then
    echo "$(date): Application Healthy"
else
    echo "$(date): Application Unhealthy"
fi
```

Make it executable:
```bash
chmod +x health_check.sh
```

Test it manually:
```bash
./health_check.sh
```

Then (optionally) schedule it every 10 minutes:
```
*/10 * * * * /home/youruser/health_check.sh >> /home/youruser/health.log 2>&1
```

---

## Part 4: Hands-on Lab — Automate MERN Server Maintenance

Create three scripts:

### 1. backup.sh — Simulate a MongoDB backup
```bash
#!/bin/bash

TIMESTAMP=$(date +%F)
BACKUP_DIR="/backups/$TIMESTAMP"

mkdir -p "$BACKUP_DIR"

echo "$(date): Starting backup to $BACKUP_DIR"

# Simulate backup (replace with mongodump if MongoDB is installed)
echo "Backing up database..." >> "$BACKUP_DIR/backup.log"
echo "$(date): Backup complete"
```

### 2. cleanup.sh — Delete files older than 7 days
```bash
#!/bin/bash

TARGET_DIR="/tmp/test-cleanup"

# Create test directory and files if they don't exist
mkdir -p "$TARGET_DIR"
touch -t $(date -d '10 days ago' +%Y%m%d%H%M) "$TARGET_DIR/old-file.txt"
touch "$TARGET_DIR/new-file.txt"

echo "$(date): Cleaning files older than 7 days in $TARGET_DIR"

find "$TARGET_DIR" -type f -mtime +7 -delete

echo "$(date): Cleanup complete"

ls -la "$TARGET_DIR"
```

### 3. health_check.sh — Verify backend responds
```bash
#!/bin/bash

URL="http://localhost:3000/health"

if curl -fs "$URL" >/dev/null; then
    echo "$(date): Application Healthy"
else
    echo "$(date): Application Unhealthy"
fi
```

### Create cron schedules:

| Task | Schedule | Cron Expression |
|------|----------|----------------|
| Backup | Daily at 2:00 AM | `0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1` |
| Cleanup | Every Sunday at 3:00 AM | `0 3 * * 0 /opt/scripts/cleanup.sh >> /var/log/backup.log 2>&1` |
| Health Check | Every 5 minutes | `*/5 * * * * /opt/scripts/health_check.sh >> /var/log/health.log 2>&1` |

### Document how you would verify each job ran successfully:
- **Backup:** `cat /var/log/backup.log` and `ls /backups/<today-date>/`
- **Cleanup:** `cat /var/log/backup.log` and verify old files removed
- **Health Check:** `cat /var/log/health.log` — check for "Application Healthy"

---

## Part 5: Mini Assignment — Production Maintenance Schedule

Create a Production Maintenance Schedule for a MERN application.

Include:

| Task | Frequency | Cron Expression | Why this frequency |
|------|-----------|----------------|-------------------|
| **Daily database backups** | Daily | `0 2 * * *` | Captures daily changes; 2 AM is low traffic |
| **Weekly log cleanup** | Weekly (Sunday) | `0 3 * * 0` | Keep a week of logs; Sunday is low activity |
| **Health checks** | Every 5 minutes | `*/5 * * * *` | Early detection of failures |
| **Monthly archive cleanup** | Monthly (1st) | `0 0 1 * *` | Move old archives to cold storage monthly |
| **SSL renewal verification** | Weekly | `0 4 * * 1` | Certs expire in ~90 days; weekly check catches issues |
| **Weekly package update review** | Weekly | `0 5 * * 0` | Review but don't auto-upgrade production |

### Explain why each task is scheduled at its chosen frequency:

1. **Daily backups** — Balances between data-loss window (max 24h) and resource usage. 2 AM because traffic is lowest.
2. **Weekly log cleanup** — Give logs a week to be useful for troubleshooting before cleanup.
3. **Health checks every 5 min** — Quick detection of failures without excessive load on the server.
4. **Monthly archive** — Move old data out monthly to keep primary storage lean and fast.
5. **Weekly SSL check** — Certificates expire in ~90 days; checking weekly catches expiry well in advance.
6. **Weekly update review** — Review available updates but don't auto-upgrade production without a maintenance process (avoids breaking changes).

---

## Part 6: Knowledge Check

Answer the following questions:

1. **What is the difference between `>>` and `>` in cron redirection?**

   <details>
   <summary>Answer</summary>

   `>>` **appends** to the file, preserving existing content. `>` **overwrites** the file. For logs, `>>` is preferred so I don't lose previous entries.
   </details>

2. **Why does `2>&1` matter in cron jobs?**

   <details>
   <summary>Answer</summary>

   It redirects standard error (2) to standard output (1), so both errors and normal output go to the same log file. Without it, errors might be lost.
   </details>

3. **Why might a script work manually but fail under cron?**

   <details>
   <summary>Answer</summary>

   Cron runs with a minimal environment (limited PATH, no shell env vars), from a different working directory, and may have permission restrictions. Absolute paths and explicit interpreters solve most issues.
   </details>

4. **How would you schedule a job at 3:30 PM every Friday?**

   <details>
   <summary>Answer</summary>

   `30 15 * * 5 command` — Minute 30, hour 15 (3 PM), any day of month, any month, day 5 (Friday).
   </details>

5. **What is the purpose of using a lock file in scheduled scripts?**

   <details>
   <summary>Answer</summary>

   It prevents overlapping executions. If a previous run hasn't finished, the script detects the lock file and exits, avoiding two instances running concurrently.
   </details>

---

## Submission Checklist

- [ ] Created `~/devops-course/day12/` workspace
- [ ] Added and verified a test cron job
- [ ] Removed the test job
- [ ] Created and tested `health_check.sh`
- [ ] Created `backup.sh`, `cleanup.sh`, and `health_check.sh`
- [ ] Scheduled all three jobs with cron
- [ ] Documented how to verify each job
- [ ] Created the production maintenance schedule
- [ ] Answered the knowledge check questions
