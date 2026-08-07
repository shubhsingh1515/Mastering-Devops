# Day 13 — Interview Questions: Linux Log Management & Troubleshooting

## Beginner

### Q1: Why are logs important?
Logs provide a **chronological record of events** that helps diagnose problems, understand system behavior, audit security, and troubleshoot production issues without guessing. They are the first evidence a DevOps engineer checks when something goes wrong.

### Q2: Where are Linux system logs commonly stored?
Under **`/var/log/`** — for example `/var/log/syslog`, `/var/log/auth.log`, and `/var/log/kern.log`.

### Q3: What does `tail -f` do?
It displays the **last lines** of a file and **follows** the file, showing new entries in real time as they are appended. It's ideal for watching live application behavior.

### Q4: What is the purpose of `grep`?
`grep` **searches files for matching text patterns**, letting me filter log files to find relevant lines quickly — for example `grep -i error app.log`.

### Q5: What is `journalctl`?
`journalctl` is the tool used to **query logs collected by systemd's journal**, including logs for systemd-managed services, e.g., `journalctl -u myapp`.

---

## Intermediate

### Q6: Explain different log levels.
Log levels indicate severity:
- **DEBUG** — Detailed diagnostic information
- **INFO** — Normal operation
- **WARN** — Something unexpected but not fatal
- **ERROR** — An operation failed
- **FATAL** — Critical failure; the application may stop

Filtering by level helps focus on the most important entries.

### Q7: Why is log rotation necessary?
Without rotation, logs grow indefinitely and **fill up the disk**, causing outages. Rotation **compresses old logs**, creates new ones, and **deletes very old ones** to keep disk usage manageable and preserve a usable history.

### Q8: How would you investigate repeated login failures?
Check `/var/log/auth.log` for authentication attempts, search for `failed` or `invalid` entries, look for patterns (e.g., same IP, brute force), and correlate with application logs to determine whether it's an attack, a credential issue, or a service problem.

### Q9: How do you search for errors in a log file?
Use `grep ERROR app.log`, or case-insensitive `grep -i error app.log`. Add `-n` for line numbers. For a live managed service, use `journalctl -u service -n 50`.

### Q10: What are the advantages of structured logging?
Structured logging (e.g., JSON) makes logs **machine-readable**, easier to parse, filter, and aggregate in centralized platforms, and enables richer querying, correlation, and alerting.

---

## Advanced

### Q11: Design a logging strategy for a production MERN application.
A robust strategy includes:
1. **Separate Nginx access and error logs** — For request traffic and failures.
2. **Structured application logs** — Request IDs, timestamps, user context, log levels.
3. **MongoDB logs** — Slow queries, connection errors, replication.
4. **Centralized aggregation** — ELK/Fluentd to search and correlate.
5. **Log rotation and retention** — Based on compliance and storage.
6. **Alerting** — On ERROR/FATAL patterns and error spikes.

### Q12: How would you investigate intermittent API failures?
Correlate application logs with Nginx access/error logs and server metrics. Look for timeouts, resource exhaustion, slow queries, and patterns tied to specific times or endpoints. Use `journalctl` and `tail -f` during reproduction, and add **request IDs** to trace the full path through the stack.

### Q13: What are the risks of logging sensitive information?
Logs may be accessible to others and retained for long periods. Logging passwords, API keys, or personal data creates a **security and compliance risk**. Secrets should be **masked or excluded** entirely.

### Q14: How do centralized logging platforms improve operations?
They **aggregate logs from many servers** into one place, enable fast search and correlation, provide dashboards and alerts, support retention policies, and help identify system-wide issues rather than isolated events.

### Q15: Describe a log-based incident response workflow.
1. **Detect** the incident (alert or user report).
2. **Capture and preserve** logs before acting.
3. **Identify** the relevant log source (app, Nginx, DB, system).
4. **Search/filter** for errors around the incident time.
5. **Determine** the root cause.
6. **Apply** a fix.
7. **Verify** in logs that the issue is resolved.
8. **Document** and add monitoring/alerts to prevent recurrence.
