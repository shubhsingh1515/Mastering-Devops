# Day 13 — Commands: Linux Log Management & Troubleshooting

## Viewing Logs

| Command | Purpose | Example |
|---------|---------|---------|
| `cat file` | Display an entire file | `cat app.log` |
| `head file` | Show the first lines | `head app.log` |
| `tail file` | Show the last lines | `tail app.log` |
| `tail -f file` | Follow live updates | `tail -f app.log` |
| `tail -n N file` | Show the last N lines | `tail -n 20 app.log` |
| `less file` | Browse large files | `less app.log` |

## Searching Logs with grep

| Command | Purpose | Example |
|---------|---------|---------|
| `grep pattern file` | Search for text | `grep ERROR app.log` |
| `grep -i pattern file` | Case-insensitive search | `grep -i error app.log` |
| `grep -c pattern file` | Count matches | `grep -c ERROR app.log` |
| `grep -n pattern file` | Show line numbers | `grep -n ERROR app.log` |

## Viewing Service Logs (systemd / journalctl)

| Command | Purpose | Example |
|---------|---------|---------|
| `journalctl -u service` | View service logs | `journalctl -u myapp` |
| `journalctl -u service -n N` | Latest N entries | `journalctl -u myapp -n 20` |
| `journalctl -u service -f` | Live view | `journalctl -u myapp -f` |

## Common Log Locations

| File | Purpose |
|------|---------|
| `/var/log/syslog` | General system activity |
| `/var/log/auth.log` | Authentication and login events |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/nginx/access.log` | Nginx web requests |
| `/var/log/nginx/error.log` | Nginx errors |
| `/var/log/mongodb/mongod.log` | MongoDB activity |

## Log Levels

| Level | Meaning |
|-------|---------|
| `DEBUG` | Detailed diagnostic information |
| `INFO` | Normal operation |
| `WARN` | Something unexpected but not fatal |
| `ERROR` | An operation failed |
| `FATAL` | Critical failure; may stop the app |

## Quick Reference

```bash
# 1. View the last lines of a log
tail app.log

# 2. Follow a log in real time
tail -f app.log

# 3. Search for errors
grep -i error app.log

# 4. Count errors
grep -c "ERROR" app.log

# 5. View a systemd service's logs (last 50)
journalctl -u myapp -n 50

# 6. Browse a large log with search (press / to search, q to quit)
less app.log
```

## Practical Log Pipeline

```bash
# Create a sample log
cat > app.log <<EOF
INFO Server started
ERROR User authentication failed
WARN Slow query detected
EOF

# Search for errors
grep ERROR app.log

# Watch for new entries live
tail -f app.log

# Summary counts
wc -l app.log
grep -c "ERROR" app.log
grep -c "WARN" app.log
