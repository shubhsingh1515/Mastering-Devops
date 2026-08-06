# Day 12 — Commands: Scheduling Tasks with Cron

## Crontab Management

| Command | Purpose | Example |
|---------|---------|---------|
| `crontab -e` | Edit scheduled jobs | `crontab -e` |
| `crontab -l` | List scheduled jobs | `crontab -l` |
| `crontab -r` | Remove all scheduled jobs | `crontab -r` |
| `date` | Show current system time | `date` |
| `logger` | Write messages to system log | `logger "Backup failed"` |

## Cron Expression Format

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0–7, Sunday = 0 or 7)
│ │ │ └──── Month (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)
```

## Common Cron Expressions

| Expression | Meaning |
|------------|---------|
| `0 2 * * *` | Every day at 2:00 AM |
| `0 * * * *` | Every hour |
| `*/5 * * * *` | Every 5 minutes |
| `30 3 * * 0` | Every Sunday at 3:30 AM |
| `30 9 * * 1-5` | Every weekday at 9:30 AM |
| `0 0 1 * *` | First day of month at midnight |
| `@daily` | Every day at midnight |
| `@hourly` | Every hour |
| `@weekly` | Every Sunday at midnight |
| `@monthly` | First day of month at midnight |
| `@reboot` | Run once at system startup |

## Special Cron Syntax

| Syntax | Meaning | Example |
|--------|---------|---------|
| `*` | Any value | `* * * * *` = every minute |
| `*/N` | Every N units | `*/10` = every 10 minutes |
| `1-5` | Range | `1-5` = days 1-5 (Mon-Fri) |
| `1,15` | Specific values | `1,15` = 1st and 15th |

## Logging Cron Output

```cron
# Append output and errors to a log file
0 1 * * * /home/ubuntu/backup.sh >> /var/log/backup.log 2>&1
```

- `>>` — Append (don't overwrite)
- `2>&1` — Redirect stderr to stdout

## Quick Reference

```bash
# 1. Edit crontab
crontab -e

# 2. Add a job (example: daily backup at 2 AM)
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# 3. List jobs to verify
crontab -l

# 4. Test the script manually first
/opt/scripts/backup.sh

# 5. Check the log
cat /var/log/backup.log
