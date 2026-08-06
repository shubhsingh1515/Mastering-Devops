# Day 12: Scheduling Tasks with Cron

## 🎯 Learning Objectives

By the end of this lesson, I'll be able to:

- **Understand what cron is** and how the cron daemon works.
- **Read and write cron expressions**.
- **Schedule recurring jobs** at specific times.
- **Manage my personal crontab** (edit, list, remove).
- **Redirect cron output** to log files.
- **Automate common DevOps maintenance tasks**.
- **Apply cron to MERN production environments**.

---

## 🤔 Why This Topic Is Important

Production servers require repetitive tasks:

- **Database backups**
- **Log cleanup**
- **Health checks**
- **SSL certificate renewal**
- **Temporary file cleanup**
- **Monitoring scripts**

Running these manually is unreliable — I might forget, or make a mistake. 

**cron automates them** so they run consistently at the right times, every time.

---

## 📖 Theory Explained

### What is Cron?

**cron** is a Linux scheduler that executes commands automatically at specified times.

**Examples of what I can schedule:**
- Every day at midnight
- Every Sunday
- Every 5 minutes
- Every month
- On specific days of the week

The cron daemon (`crond`) runs in the background and checks every minute whether any scheduled job needs to run.

### The Cron Format

A cron expression has **5 time fields** followed by the command to run:

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0–7, Sunday = 0 or 7)
│ │ │ └──── Month (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)
```

| Field | Allowed Values | Meaning |
|-------|---------------|---------|
| Minute | 0–59 | Minute of the hour |
| Hour | 0–23 | Hour of the day |
| Day of month | 1–31 | Day of the month |
| Month | 1–12 | Month of the year |
| Day of week | 0–7 | Day of the week (0 or 7 = Sunday) |

### Common Cron Examples

**Run every day at 2:00 AM:**
```
0 2 * * * /home/user/backup.sh
```

**Every hour:**
```
0 * * * * command
```

**Every 5 minutes:**
```
*/5 * * * * command
```

**Every Sunday at 3:30 AM:**
```
30 3 * * 0 command
```

**Every weekday (Mon–Fri) at 9:30 AM:**
```
30 9 * * 1-5 command
```

**Every first day of the month at midnight:**
```
0 0 1 * * command
```

### Special Cron Syntax

| Special String | Meaning |
|----------------|---------|
| `*` | Every value (any) |
| `*/5` | Every 5 units (step) |
| `1-5` | Range (Mon–Fri, 1–5) |
| `1,15` | Specific values (1 and 15) |
| `@daily` | Every day at midnight |
| `@hourly` | Every hour |
| `@weekly` | Every Sunday at midnight |
| `@monthly` | First day of month at midnight |
| `@reboot` | Run once at system startup |

### Managing Your Crontab

Each user has their own **crontab** (cron table) file that stores their scheduled jobs.

**Edit your cron jobs:**
```bash
crontab -e
```
This opens the crontab in the default editor.

**List your cron jobs:**
```bash
crontab -l
```

**Remove all your cron jobs:**
```bash
crontab -r
```

> ⚠️ **Warning:** `crontab -r` deletes **all** cron jobs for the current user. Use with caution!

### Logging Cron Output

**Without redirection (bad):**
```
0 1 * * * backup.sh
```
Any output goes nowhere, and I won't know if it failed.

**With redirection (better):**
```
0 1 * * * /home/ubuntu/backup.sh >> /var/log/backup.log 2>&1
```

**Explanation:**
- `>>` → **Append** output to the log file (instead of overwriting)
- `2>&1` → Redirect **errors** (stderr, file descriptor 2) to the same place as standard output (stdout, file descriptor 1)

This is **essential for troubleshooting** — I can review the log to confirm jobs ran successfully.

### Environment Variables

Cron runs with a **minimal environment** — it doesn't have all the PATH variables my interactive shell has.

**Instead of:**
```
node app.js
```

**Prefer (absolute paths):**
```
/usr/bin/node /opt/app/app.js
```

**Or set PATH explicitly at the top of my crontab:**
```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

This avoids the common problem where scripts work manually but fail under cron because commands can't be found.

---

## 🌍 Real-World MERN Example

### Daily Database Backup

**Create `backup.sh`:**
```bash
#!/bin/bash

TIMESTAMP=$(date +%F)

mkdir -p /backups

mongodump --out "/backups/$TIMESTAMP"
```

**Make it executable:**
```bash
chmod +x /opt/scripts/backup.sh
```

**Schedule it (every night at 2:00 AM):**
```
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

Every night at 2:00 AM, a timestamped MongoDB backup is created at `/backups/YYYY-MM-DD/`.

### Health Check Every 5 Minutes

```
*/5 * * * * curl -fs http://localhost:3000/health >/dev/null || logger "Backend health check failed"
```

- `curl -fs` — Fail silently if the endpoint isn't reachable
- `>/dev/null` — Discard normal output
- `|| logger "..."` — If the curl fails, write a message to the system log

If the health endpoint fails, a message is written to the system log, which I can review with `journalctl`.

---

## 🏗️ Architecture Diagram (ASCII)

```
             Cron Daemon
                   │
          Scheduled Time
                   │
             backup.sh
         ┌─────────┴─────────┐
         │                   │
   MongoDB Backup       backup.log
```

---

## 🛠️ Practical Exercise

### Step 1 — View existing cron jobs
```bash
crontab -l
```

### Step 2 — Open the crontab editor
```bash
crontab -e
```
If this is my first time, I'll be asked to choose an editor (I'll pick nano).

### Step 3 — Add a simple job
```
*/2 * * * * echo "Cron works" >> ~/cron-test.log
```
This appends the message `Cron works` to `~/cron-test.log` every two minutes.

### Step 4 — Wait a few minutes, then verify
```bash
cat ~/cron-test.log
```
Expected output:
```
Cron works
Cron works
```

### Step 5 — Remove the test job after confirming it works
```bash
crontab -e
```
Then delete the line and save.

---

## 💻 Important Commands

| Command | Purpose |
|---------|---------|
| `crontab -e` | Edit scheduled jobs |
| `crontab -l` | List scheduled jobs |
| `crontab -r` | Remove all scheduled jobs |
| `date` | Show current system time |
| `logger` | Write messages to the system log |

---

## ✅ Best Practices

1. **Use absolute paths in cron jobs** — Cron has a minimal PATH, so `/usr/bin/script.sh` is safer than `script.sh`.
2. **Redirect both standard output and errors to log files** — Using `>> log 2>&1` so I can review what happened.
3. **Test scripts manually before scheduling them** — Never schedule something I haven't verified works.
4. **Keep cron jobs idempotent where possible** — Safe to run multiple times without harm.
5. **Add comments to complex crontabs** — Use `#` to document what each job does.
6. **Monitor backup jobs regularly** — Don't assume they succeeded; check the logs.

---

## ❌ Common Mistakes

1. **Using relative paths** — The script works manually but fails under cron because cron's working directory is different.
2. **Forgetting executable permissions** — The script needs `chmod +x`.
3. **Assuming cron has the same environment as my shell** — PATH and env vars are minimal.
4. **Overwriting logs with `>` instead of appending with `>>`** — Using `>` destroys previous log entries.
5. **Scheduling overlapping long-running jobs** — If a job takes longer than its interval, multiple instances run concurrently and conflict.

---

## 💼 Interview Questions

### Beginner

**Q1: What is cron?**
> **cron** is a Linux scheduler daemon that automatically executes commands or scripts at specified times or intervals. It runs in the background and checks every minute whether any scheduled jobs need to run. It's used for tasks like backups, log cleanup, and health checks.

**Q2: How do you edit a user's crontab?**
> `crontab -e` — This opens the current user's crontab file in the default text editor, where I can add, edit, or remove scheduled jobs.

**Q3: What does `*/10` mean in the minute field?**
> `*/10` in the minute field means **every 10 minutes**. The `*` means "any" and the `/10` means "every 10 units". So `*/10 * * * * command` runs the command every 10 minutes.

**Q4: How do you list scheduled jobs?**
> `crontab -l` — This displays all cron jobs scheduled for the current user.

**Q5: Why should you use absolute paths in cron jobs?**
> Because cron runs with a **minimal environment** — it doesn't have the full PATH that my interactive shell has. Using absolute paths (like `/usr/bin/script.sh` or `/opt/scripts/backup.sh`) ensures the command is found even with the minimal environment.

### Intermediate

**Q6: Explain each field of a cron expression.**
> A cron expression has 5 fields:
> ```
> * * * * * command
> │ │ │ │ └── Day of week (0-7, Sunday=0 or 7)
> │ │ │ └──── Month (1-12)
> │ │ └────── Day of month (1-31)
> │ └──────── Hour (0-23)
> └────────── Minute (0-59)
> ```
> Each field specifies when the command should run. `*` means "any value". For example, `0 2 * * *` means "at minute 0, hour 2, every day" = every day at 2:00 AM.

**Q7: Why can scripts work manually but fail under cron?**
> Common reasons:
> 1. **PATH differences** — Cron has a minimal PATH, so commands like `node` may not be found without absolute paths
> 2. **Working directory** — Cron runs from the user's home directory, not the script's location
> 3. **Environment variables** — Cron doesn't inherit my shell's environment variables
> 4. **Permissions** — The script may not be executable, or the cron user may not have access
> 5. **Output not logged** — Script errors are invisible without redirection

**Q8: How do you capture cron job output?**
> Redirect output to a log file:
> ```
> 0 1 * * * /home/ubuntu/backup.sh >> /var/log/backup.log 2>&1
> ```
> - `>>` appends standard output to the log
> - `2>&1` redirects standard error to the same log
> This captures both success messages and errors for troubleshooting.

**Q9: What maintenance tasks are commonly automated?**
> Common cron-automated tasks:
> - **Database backups** (daily)
> - **Log cleanup/rotation** (weekly)
> - **Health checks** (every few minutes)
> - **SSL certificate renewal** (monthly or when needed)
> - **Temporary file cleanup** (daily/weekly)
> - **Package updates review** (weekly)
> - **Monitoring scripts**
> - **Report generation** (daily/weekly)

**Q10: How would you schedule a job every weekday at 9:30 AM?**
> ```
> 30 9 * * 1-5 command
> ```
> - `30` = minute 30
> - `9` = hour 9 (9 AM)
> - `*` = any day of month
> - `*` = any month
> - `1-5` = Monday through Friday (days 1-5)

### Advanced

**Q11: Design a backup schedule for a production MERN application.**
> A comprehensive backup schedule:
> ```
> # Daily MongoDB backup at 2:00 AM
> 0 2 * * * /opt/scripts/mongo_backup.sh >> /var/log/backup.log 2>&1
>
> # Daily database backup verification at 2:30 AM
> 30 2 * * * /opt/scripts/verify_backup.sh >> /var/log/backup.log 2>&1
>
> # Weekly full backup (Sunday) at 1:00 AM
> 0 1 * * 0 /opt/scripts/full_backup.sh >> /var/log/backup.log 2>&1
>
> # Monthly archive to cold storage at 12:00 AM on the 1st
> 0 0 1 * * /opt/scripts/archive_backup.sh >> /var/log/backup.log 2>&1
> ```
> The daily backup captures recent changes, the weekly full backup ensures a complete snapshot, and the monthly archive moves old backups to cold storage. Verification ensures backups are actually usable.

**Q12: How would you prevent overlapping backup jobs?**
> Use a **lock file** to prevent concurrent execution:
> ```bash
> #!/bin/bash
> LOCK_FILE="/tmp/backup.lock"
>
> # Exit if already running
> if [ -f "$LOCK_FILE" ]; then
>     echo "$(date): Backup already running, exiting"
>     exit 0
> fi
>
> # Create lock file
> touch "$LOCK_FILE"
>
> # Run backup
> mongodump --out "/backups/$(date +%F)"
>
> # Remove lock file
> rm -f "$LOCK_FILE"
> ```
> This ensures if one backup takes longer than the interval, a new one won't start until the previous finishes.

**Q13: What security considerations apply to scheduled scripts?**
> Security considerations:
> 1. **Run scripts with least privilege** — Don't schedule as root unless necessary
> 2. **Protect scripts** — Set proper permissions (750, owner-only write)
> 3. **Protect credentials** — Don't hardcode passwords in scripts; use environment variables or secret files
> 4. **Secure backups** — Encrypt sensitive backups
> 5. **Log monitoring** — Monitor logs for failures or suspicious activity
> 6. **Validate input** — Ensure scripts can't be exploited
> 7. **Limit write access** — Only the owner should be able to modify scheduled scripts

**Q14: When would you use systemd timers instead of cron?**
> Use **systemd timers** when:
> 1. **Need better logging** — Timer output goes to the journal (`journalctl`)
> 2. **Need dependency management** — Timers can wait for other units/services
> 3. **Need missed-run handling** — systemd can run missed jobs after system downtime
> 4. **Need resource limits** — Timers can set CPU/memory limits
> 5. **Standardize on systemd** — If the whole system uses systemd, timers are consistent
> 6. **Need on-demand or calendar events** — More flexible scheduling options
>
> Cron is still simpler and fine for basic recurring jobs. systemd timers are more powerful for complex scenarios.

**Q15: How would you monitor failed cron jobs?**
> Monitoring strategies:
> 1. **Always log output** — Redirect to a log file: `>> /var/log/job.log 2>&1`
> 2. **Check exit codes in the script** — Have the script report success/failure
> 3. **Use `logger`** — Write failures to the system journal: `|| logger "Backup failed"`
> 4. **Email on failure** — Configure MAILTO in crontab, or use `mail` in the script
> 5. **Monitoring tools** — Use a tool like Nagios, Zabbix, or a notification service (Slack, PagerDuty)
> 6. **Regular review** — Check logs daily/weekly
> 7. **Health check scripts** — A separate script that verifies backups exist and are recent

---

## 📝 Quiz (10 Questions)

Let me test my understanding:

**1. Which command edits your crontab?**
<details>
<summary>Answer</summary>

`crontab -e` — Opens the crontab in the default editor.

</details>

**2. Which command lists cron jobs?**
<details>
<summary>Answer</summary>

`crontab -l` — Lists all scheduled jobs for the current user.

</details>

**3. What does `0 0 * * *` represent?**
<details>
<summary>Answer</summary>

**Every day at midnight** — Minute 0, hour 0, every day of month, every month, every day of week.

</details>

**4. What does `*/5` mean?**
<details>
<summary>Answer</summary>

**Every 5 units** — For example, `*/5 * * * *` runs every 5 minutes.

</details>

**5. Why should cron jobs use absolute paths?**
<details>
<summary>Answer</summary>

Because cron runs with a **minimal environment** — it doesn't have the full PATH of my interactive shell, so relative paths may not resolve.

</details>

**6. What does `2>&1` do?**
<details>
<summary>Answer</summary>

**Redirects standard error (file descriptor 2) to standard output (file descriptor 1)** — So both errors and normal output go to the same place.

</details>

**7. Which command removes all cron jobs?**
<details>
<summary>Answer</summary>

`crontab -r` — ⚠️ Deletes all cron jobs for the current user.

</details>

**8. What service executes scheduled cron jobs?**
<details>
<summary>Answer</summary>

The **cron daemon** (`crond`) — It runs in the background and checks every minute for jobs to execute.

</details>

**9. Where should cron job output be logged?**
<details>
<summary>Answer</summary>

To a **dedicated log file** (e.g., `/var/log/backup.log`) using `>> log 2>&1`, so I can review and troubleshoot.

</details>

**10. Why should you test scripts before scheduling them?**
<details>
<summary>Answer</summary>

To **verify they work correctly before automation** — A scheduled script that fails silently is worse than not running it at all, because I might assume the task was done.

</details>

---

## 👨‍💻 Coding Exercise

### health_check.sh

```bash
#!/bin/bash

URL="http://localhost:3000/health"

if curl -fs "$URL" >/dev/null; then
    echo "$(date): Application Healthy"
else
    echo "$(date): Application Unhealthy"
fi
```

**Make it executable:**
```bash
chmod +x health_check.sh
```

**Test it manually:**
```bash
./health_check.sh
```

**Then (optionally) schedule it every 10 minutes:**
```
*/10 * * * * /home/youruser/health_check.sh >> /home/youruser/health.log 2>&1
```

**What I learned:**
- `curl -fs` — Fails silently (no output) if the request fails
- `>/dev/null` — Discards the normal curl output
- `$(date)` — Inserts the current timestamp into the log message

---

## 🧪 Hands-on Lab

### Lab: Automate MERN Server Maintenance

Create three scripts:

**1. `backup.sh`** — Simulate a MongoDB backup (or create a timestamped backup directory if MongoDB isn't installed):
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

**2. `cleanup.sh`** — Delete files older than 7 days from a test directory:
```bash
#!/bin/bash

TARGET_DIR="/tmp/test-cleanup"

echo "$(date): Cleaning files older than 7 days in $TARGET_DIR"

find "$TARGET_DIR" -type f -mtime +7 -delete

echo "$(date): Cleanup complete"
```

**3. `health_check.sh`** — Verify my backend responds successfully:
```bash
#!/bin/bash

URL="http://localhost:3000/health"

if curl -fs "$URL" >/dev/null; then
    echo "$(date): Application Healthy"
else
    echo "$(date): Application Unhealthy"
fi
```

**Create cron schedules for:**

| Task | Schedule | Cron Expression |
|------|----------|----------------|
| Backup | Daily at 2:00 AM | `0 2 * * * /opt/scripts/backup.sh` |
| Cleanup | Every Sunday at 3:00 AM | `0 3 * * 0 /opt/scripts/cleanup.sh` |
| Health Check | Every 5 minutes | `*/5 * * * * /opt/scripts/health_check.sh` |

**Document how I would verify each job ran successfully:**
- Check the log files: `cat /var/log/backup.log`, `cat /var/log/cleanup.log`
- Verify backups exist: `ls /backups/`
- Check the health log: `cat /home/youruser/health.log`

---

## 📋 Mini Assignment

### Production Maintenance Schedule for a MERN Application

Create a schedule that includes:

| Task | Frequency | Cron Expression | Why |
|------|-----------|----------------|-----|
| **Daily database backups** | Daily | `0 2 * * *` | Captures daily changes; low traffic at 2 AM |
| **Weekly log cleanup** | Weekly (Sunday) | `0 3 * * 0` | Removes old logs; low activity on Sunday |
| **Health checks** | Every 5 minutes | `*/5 * * * *` | Early detection of failures |
| **Monthly archive cleanup** | Monthly (1st) | `0 0 1 * *` | Moves old archives to cold storage |
| **SSL renewal verification** | Weekly | `0 4 * * 1` | Ensures certs are valid before expiry |
| **Weekly package update review** | Weekly | `0 5 * * 0` | Review updates; don't auto-upgrade production |

**Explain why each task is scheduled at its chosen frequency:**
- **Daily backups** — Balance between data loss window and resource usage
- **Weekly cleanup** — Give logs a week to be useful before cleanup
- **Health checks every 5 min** — Quick detection without excessive load
- **Monthly archive** — Move old data out monthly to keep primary storage lean
- **Weekly SSL check** — Certs expire in 90 days; weekly check catches issues early
- **Weekly update review** — Review but don't auto-upgrade production (needs a maintenance process)

---

## 📚 Recommended Documentation

I should read the manual pages for deeper understanding:

```bash
man crontab    # Crontab file format and options
man cron       # Cron daemon documentation
man logger     # Log message utility
```

---

## 📖 Summary

Today I learned:

✅ **How cron automates recurring tasks** — The cron daemon runs scheduled jobs automatically.

✅ **How to write cron expressions** — The 5-field format (minute, hour, day, month, day-of-week).

✅ **How to manage crontabs** — Edit (`crontab -e`), list (`crontab -l`), remove (`crontab -r`).

✅ **Why absolute paths matter** — Cron has a minimal environment.

✅ **How to log cron output** — Using `>> log 2>&1` for troubleshooting.

✅ **How to automate backups, health checks, and maintenance** — For MERN applications.

### Key Takeaways

```
crontab -e → Add job → Absolute paths → Log output → Monitor
```

Cron is one of the **simplest but most powerful automation tools** in Linux. Even with modern orchestration platforms, understanding cron is invaluable for system administration and production operations.

---

## 🔗 Connecting to the Bigger Picture

Cron connects with everything I've learned:

- **Bash Scripting** → Cron runs Bash scripts (`backup.sh`, `health_check.sh`)
- **Services** → Cron itself is a service managed by systemd
- **Processes** → Each scheduled job runs as a process
- **Permissions** → Scheduled scripts need executable permissions
- **Packages** → Install cron with `apt install cron`
- **SSH** → SSH backups can be scheduled via cron

Cron ties my scripting skills into automated, scheduled infrastructure management.

---

*"Cron turns manual, forgettable tasks into reliable, scheduled automation. It's the quiet workhorse of Linux operations."*
</content>
