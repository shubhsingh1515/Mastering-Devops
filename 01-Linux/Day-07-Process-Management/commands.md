# Linux Commands (Day 7 — Process Management)

## Viewing Processes
| Command | Purpose |
|---------|---------|
| `ps` | Show processes for the current terminal |
| `ps -ef` | Show all running processes (full format) |
| `ps aux` | Show all processes with CPU/memory usage |
| `ps -ef \| grep node` | Find a specific process by name |

## Live Monitoring
| Command | Purpose |
|---------|---------|
| `top` | Live process monitor (CPU, memory, load) |
| `htop` | Interactive process viewer (user-friendly) |
| `uptime` | Show system uptime and load averages |

## Job Control
| Command | Purpose |
|---------|---------|
| `command &` | Start a process in the background |
| `jobs` | List background jobs |
| `fg` | Bring a background job to the foreground |
| `Ctrl + C` | Stop a foreground process |
| `Ctrl + Z` | Suspend a foreground process |
| `bg` | Resume a suspended job in the background |

## Terminating Processes
| Command | Purpose |
|---------|---------|
| `kill PID` | Send SIGTERM (graceful termination) |
| `kill -9 PID` | Send SIGKILL (force termination) |
| `kill -15 PID` | Explicitly send SIGTERM |
| `kill -2 PID` | Send SIGINT (like Ctrl+C) |
| `pkill name` | Kill processes by name |
| `killall name` | Kill all processes with a given name |

## Quick Examples

**Find a Node.js process:**
```bash
ps -ef | grep node
```

**Check system load:**
```bash
uptime
```

**Start a process in the background:**
```bash
sleep 300 &
```

**List background jobs:**
```bash
jobs
```

**Bring latest job to foreground:**
```bash
fg
```

**Gracefully stop a process:**
```bash
kill 2511
```

**Force stop if unresponsive:**
```bash
kill -9 2511
```

**Kill all Node.js processes:**
```bash
pkill node
```

**Monitor system resources:**
```bash
top
```

**Quick system health check:**
```bash
echo "=== Uptime ===" && uptime && echo "=== Top 5 CPU Processes ===" && ps aux --sort=-%cpu | head -6 && echo "=== Top 5 Memory Processes ===" && ps aux --sort=-%mem | head -6
