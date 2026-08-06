# Day 12 — Interview Questions: Scheduling Tasks with Cron

## Beginner

### Q1: What is cron?
**cron** is a Linux scheduler daemon that automatically executes commands or scripts at specified times or intervals. It runs in the background and checks every minute whether any scheduled jobs need to run. It's used for automating recurring tasks like backups, log cleanup, and health checks.

### Q2: How do you edit a user's crontab?
`crontab -e` — This opens the current user's crontab file in the default text editor, where I can add, edit, or remove scheduled jobs.

### Q3: What does `*/10` mean in the minute field?
`*/10` in the minute field means **every 10 minutes**. The `*` means "any" and the `/10` means "every 10 units". So `*/10 * * * * command` runs the command every 10 minutes.

### Q4: How do you list scheduled jobs?
`crontab -l` — This displays all cron jobs scheduled for the current user.

### Q5: Why should you use absolute paths in cron jobs?
Because cron runs with a **minimal environment** — it doesn't have the full PATH that my interactive shell has. Using absolute paths (like `/usr/bin/script.sh` or `/opt/scripts/backup.sh`) ensures the command is found even with the minimal environment.

---

## Intermediate

### Q6: Explain each field of a cron expression.
A cron expression has 5 fields:
```
* * * * * command
│ │ │ │ └── Day of week (0-7, Sunday=0 or 7)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```
Each field specifies when the command should run. `*` means "any value". For example, `0 2 * * *` means "at minute 0, hour 2, every day" = every day at 2:00 AM.

### Q7: Why can scripts work manually but fail under cron?
Common reasons:
1. **PATH differences** — Cron has a minimal PATH, so commands like `node` may not be found without absolute paths.
2. **Working directory** — Cron runs from the user's home directory, not the script's location.
3. **Environment variables** — Cron doesn't inherit my shell's environment variables.
4. **Permissions** — The script may not be executable, or the cron user may not have access.
5. **Output not logged** — Script errors are invisible without redirection.

### Q8: How do you capture cron job output?
Redirect output to a log file:
```
0 1 * * * /home/ubuntu/backup.sh >> /var/log/backup.log 2>&1
```
- `>>` appends standard output to the log
- `2>&1` redirects standard error to the same log
This captures both success messages and errors for troubleshooting.

### Q9: What maintenance tasks are commonly automated?
Common cron-automated tasks:
- **Database backups** (daily)
- **Log cleanup/rotation** (weekly)
- **Health checks** (every few minutes)
- **SSL certificate renewal** (monthly or when needed)
- **Temporary file cleanup** (daily/weekly)
- **Package updates review** (weekly)
- **Monitoring scripts**
- **Report generation** (daily/weekly)

### Q10: How would you schedule a job every weekday at 9:30 AM?
```
30 9 * * 1-5 command
```
- `30` = minute 30
- `9` = hour 9 (9 AM)
- `*` = any day of month
- `*` = any month
- `1-5` = Monday through Friday (days 1-5)

---

## Advanced

### Q11: Design a backup schedule for a production MERN application.
A comprehensive backup schedule:
```
# Daily MongoDB backup at 2:00 AM
0 2 * * * /opt/scripts/mongo_backup.sh >> /var/log/backup.log 2>&1

# Daily database backup verification at 2:30 AM
30 2 * * * /opt/scripts/verify_backup.sh >> /var/log/backup.log 2>&1

# Weekly full backup (Sunday) at 1:00 AM
0 1 * * 0 /opt/scripts/full_backup.sh >> /var/log/backup.log 2>&1

# Monthly archive to cold storage at 12:00 AM on the 1st
0 0 1 * * /opt/scripts/archive_backup.sh >> /var/log/backup.log 2>&1
```
The daily backup captures recent changes, the weekly full backup ensures a complete snapshot, and the monthly archive moves old backups to cold storage. Verification ensures backups are actually usable.

### Q12: How would you prevent overlapping backup jobs?
Use a **lock file** to prevent concurrent execution:
```bash
#!/bin/bash
LOCK_FILE="/tmp/backup.lock"

# Exit if already running
if [ -f "$LOCK_FILE" ]; then
    echo "$(date): Backup already running, exiting"
    exit 0
fi

# Create lock file
touch "$LOCK_FILE"

# Run backup
mongodump --out "/backups/$(date +%F)"

# Remove lock file
rm -f "$LOCK_FILE"
```
This ensures if one backup takes longer than the interval, a new one won't start until the previous finishes.

### Q13: What security considerations apply to scheduled scripts?
Security considerations:
1. **Run scripts with least privilege** — Don't schedule as root unless necessary.
2. **Protect scripts** — Set proper permissions (750, owner-only write).
3. **Protect credentials** — Don't hardcode passwords in scripts; use environment variables or secret files.
4. **Secure backups** — Encrypt sensitive backups.
5. **Log monitoring** — Monitor logs for failures or suspicious activity.
6. **Validate input** — Ensure scripts can't be exploited.
7. **Limit write access** — Only the owner should be able to modify scheduled scripts.

### Q14: When would you use systemd timers instead of cron?
Use **systemd timers** when:
1. **Need better logging** — Timer output goes to the journal (`journalctl`).
2. **Need dependency management** — Timers can wait for other units/services.
3. **Need missed-run handling** — systemd can run missed jobs after system downtime.
4. **Need resource limits** — Timers can set CPU/memory limits.
5. **Standardize on systemd** — If the whole system uses systemd, timers are consistent.
6. **Need on-demand or calendar events** — More flexible scheduling options.

Cron is still simpler and fine for basic recurring jobs. systemd timers are more powerful for complex scenarios.

### Q15: How would you monitor failed cron jobs?
Monitoring strategies:
1. **Always log output** — Redirect to a log file: `>> /var/log/job.log 2>&1`.
2. **Check exit codes in the script** — Have the script report success/failure.
3. **Use `logger`** — Write failures to the system journal: `|| logger "Backup failed"`.
4. **Email on failure** — Configure MAILTO in crontab, or use `mail` in the script.
5. **Monitoring tools** — Use a tool like Nagios, Zabbix, or a notification service (Slack, PagerDuty).
6. **Regular review** — Check logs daily/weekly.
7. **Health check scripts** — A separate script that verifies backups exist and are recent.
