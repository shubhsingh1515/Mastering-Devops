# Day 12 — Quiz: Scheduling Tasks with Cron

## Questions

1. **Which command edits your crontab?**

   <details>
   <summary>Answer</summary>

   `crontab -e` — Opens the current user's crontab in the default text editor.
   </details>

2. **Which command lists cron jobs?**

   <details>
   <summary>Answer</summary>

   `crontab -l` — Lists all scheduled jobs for the current user.
   </details>

3. **What does `0 0 * * *` represent?**

   <details>
   <summary>Answer</summary>

   **Every day at midnight** — Minute 0, hour 0, every day of month, every month, every day of week.
   </details>

4. **What does `*/5` mean?**

   <details>
   <summary>Answer</summary>

   **Every 5 units** — For example, `*/5 * * * *` runs a command every 5 minutes.
   </details>

5. **Why should cron jobs use absolute paths?**

   <details>
   <summary>Answer</summary>

   Because cron runs with a **minimal environment** — it doesn't have the full PATH of my interactive shell, so relative paths may not resolve correctly.
   </details>

6. **What does `2>&1` do?**

   <details>
   <summary>Answer</summary>

   **Redirects standard error (file descriptor 2) to standard output (file descriptor 1)** — So both errors and normal output go to the same place.
   </details>

7. **Which command removes all cron jobs?**

   <details>
   <summary>Answer</summary>

   `crontab -r` — ⚠️ Deletes all cron jobs for the current user. Use with caution!
   </details>

8. **What service executes scheduled cron jobs?**

   <details>
   <summary>Answer</summary>

   The **cron daemon** (`crond`) — It runs in the background and checks every minute whether any scheduled jobs need to execute.
   </details>

9. **Where should cron job output be logged?**

   <details>
   <summary>Answer</summary>

   To a **dedicated log file** (e.g., `/var/log/backup.log`) using `>> log 2>&1`, so I can review and troubleshoot.
   </details>

10. **Why should you test scripts before scheduling them?**

    <details>
    <summary>Answer</summary>

    To **verify they work correctly before automation**. A scheduled script that fails silently is worse than not running it, because I might assume the task was completed.
    </details>

## Bonus Questions

11. **What does the `*` character mean in a cron expression?**

    <details>
    <summary>Answer</summary>

    It means **"any value"** — the field can match any value. For example, `* * * * *` runs every minute.
    </details>

12. **How do you schedule a job every 10 minutes?**

    <details>
    <summary>Answer</summary>

    `*/10 * * * * command` — The `*/10` in the minute field runs every 10 minutes.
    </details>

13. **What is the special string `@reboot` used for?**

    <details>
    <summary>Answer</summary>

    It runs a command **once when the system boots**. Useful for startup tasks that should run after every reboot.
    </details>

14. **What does `>>` do in a cron job?**

    <details>
    <summary>Answer</summary>

    It **appends** the command output to the log file, preserving previous log entries. Using `>` (single) would overwrite the log.
    </details>

15. **How would you prevent overlapping long-running cron jobs?**

    <details>
    <summary>Answer</summary>

    Use a **lock file** — the script checks if a lock file exists and exits if it does, creating the lock file at the start and removing it at the end.
    </details>
