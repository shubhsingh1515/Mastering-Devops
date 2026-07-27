# Day 08: Linux Services & systemctl

## 📅 Phase 1: Linux
- **Topic:** Linux Services & systemctl
- **Estimated Time:** 25–30 minutes
- **Goal:** Learn how Linux manages long-running applications (services), how to control them using `systemctl`, and how to troubleshoot production services like Nginx, Docker, MongoDB, and Node.js.

---

## 📊 Progress Tracker
- ✅ **Phase:** Linux
- ✅ **Completed Lessons:** Day 1 – Day 7
- 📖 **Today's Lesson:** Day 8 – Linux Services & systemctl
- 🎯 **Next Lesson:** Day 9 – (Upcoming)

---

## 🔁 Quick Review (Days 1–7)

Before diving into services, let me quickly test my recall of the previous lessons:

1. **Which command shows all running processes?**
   ✅ `ps -ef` or `ps aux`

2. **Which command checks listening ports?**
   ✅ `ss -tuln`

3. **What permission should a `.env` file have?**
   ✅ `600` (rw-------)

4. **Which directory stores system logs?**
   ✅ `/var/log`

5. **What is the purpose of `sudo`?**
   ✅ Execute commands with administrative (root) privileges.

---

## 🎯 Learning Objectives

By the end of this lesson, I'll be able to:

- Understand what Linux services (daemons) are.
- Understand the role of **systemd** as the service manager.
- **Start, stop, restart, and reload** services using `systemctl`.
- **Enable and disable** services at boot.
- **Check service status** to determine if a service is running, stopped, or failed.
- **View service logs** with `journalctl`.
- Apply these concepts to **production MERN deployments**.

---

## 🤔 Why This Topic Is Important

Imagine this production environment:

```
React Frontend
      ↓
    Nginx
      ↓
Node.js Backend
      ↓
   MongoDB
```

Each component runs as a **service**.

If the backend crashes, I don't manually run:

```bash
node server.js
```

Instead, I use:

```bash
sudo systemctl restart myapp
```

**Every production Linux server depends on services.** Understanding how to manage them is a core DevOps skill.

---

## 📖 Theory Explained in Simple Language

### What is a Service?

A **service** (also called a **daemon**) is a background program that starts automatically and keeps running.

**Examples of services:**
- **Nginx** — web server
- **Docker** — container runtime
- **SSH** — secure shell access
- **MongoDB** — database
- **Jenkins** — CI/CD automation

> Unlike a command I run manually in the terminal, a service continues running even after I log out.

### What is systemd?

**systemd** is the **service manager** used by most modern Linux distributions (Ubuntu, Debian, CentOS, Rocky Linux, etc.).

**Responsibilities of systemd:**
- Starting services at boot
- Stopping services
- Restarting services
- Monitoring failed services
- Managing dependencies between services
- Handling service logs

**Important:** systemd is typically **PID 1** — the first userspace process started by the kernel. Every other process is a child of systemd.

```
Linux Kernel
     ↓
  systemd (PID 1)
     ↓
  All other processes
```

### systemctl

**systemctl** is the command-line tool I use to interact with systemd.

Think of it as the **remote control for my server's services**.

#### Common Service Commands

| Command | What it does |
|---------|-------------|
| `systemctl status nginx` | Show whether nginx is running, stopped, or failed |
| `systemctl start nginx` | Start a stopped service |
| `systemctl stop nginx` | Stop a running service |
| `systemctl restart nginx` | Stop and start again (full restart) |
| `systemctl reload nginx` | Re-read config without fully stopping |
| `systemctl enable nginx` | Configure service to start automatically at boot |
| `systemctl disable nginx` | Remove automatic startup at boot |
| `systemctl is-enabled nginx` | Check if service is configured for boot startup |

#### Understanding Service Status Output

When I run:

```bash
systemctl status ssh
```

I see something like:

```
● ssh.service
   Loaded: loaded (/usr/lib/systemd/system/ssh.service; enabled)
   Active: active (running)
   Main PID: 732 (sshd)
```

**Important fields explained:**

| Field | Meaning |
|-------|---------|
| **Loaded** | Is the service unit file installed and loaded? |
| **Active** | Is it running, stopped, failed, or activating? |
| **Main PID** | The primary Process ID of the service |

**Common active states:**
- `active (running)` — ✅ Service is operating normally
- `active (exited)` — ✅ Service ran and completed (for one-time tasks)
- `inactive (dead)` — ❌ Service is stopped
- `failed` — ❌ Service has encountered an error

#### Viewing Logs with journalctl

Most services managed by systemd write their logs to the **system journal**.

**View all logs for a service:**
```bash
journalctl -u nginx
```

**View the last 20 entries:**
```bash
journalctl -u nginx -n 20
```

**Follow logs live (like `tail -f`):**
```bash
journalctl -u nginx -f
```

> Press `Ctrl + C` to stop following logs.

### Service Unit Files

Service definitions are stored in **unit files**.

**Common locations:**
| Path | Purpose |
|------|---------|
| `/etc/systemd/system/` | Custom/user-defined services |
| `/lib/systemd/system/` | System-installed services (from packages) |

**Example of a simple unit file for a Node.js backend:**
```ini
[Unit]
Description=Node.js MERN Backend
After=network.target

[Service]
ExecStart=/usr/bin/node /opt/mern/server.js
Restart=always
User=nodeapp
WorkingDirectory=/opt/mern

[Install]
WantedBy=multi-user.target
```

**Breaking down the unit file:**

| Section | Purpose |
|---------|---------|
| `[Unit]` | Metadata and dependencies (description, ordering) |
| `[Service]` | How to run the service (command, user, restart policy) |
| `[Install]` | When to start the service (boot target) |

**Key directives:**
- `ExecStart=` — The command to run the service
- `Restart=always` — Automatically restart if the process crashes
- `User=` — Which Linux user to run the service as (security!)
- `After=` — Start this service only after specified targets are reached

> I'll learn to create my own unit files later in the course after covering Bash scripting.

---

## 🌍 Real-World Example

### Scenario: Node.js API Won't Start

Users report that `https://api.company.com` returns **502 Bad Gateway**.

**My troubleshooting workflow:**

**Step 1 — Check the backend service:**
```bash
sudo systemctl status myapp
```

**Step 2 — Read recent logs:**
```bash
journalctl -u myapp -n 50
```

**Step 3 — Restart if needed:**
```bash
sudo systemctl restart myapp
```

**Step 4 — Confirm it's listening on the correct port:**
```bash
ss -tuln | grep 3000
```

**Step 5 — Test locally:**
```bash
curl http://localhost:3000
```

> This sequence mirrors real production incident response. Every DevOps engineer follows a similar pattern.

---

## 🏗️ Architecture Diagram (ASCII)

```
          Linux Kernel
               │
            systemd
           (PID 1)
               │
     ┌─────────┼──────────┐
     │         │          │
   nginx     docker      ssh
               │
        nodeapp.service
               │
        Node.js Backend
```

---

## 🛠️ Step-by-Step Practical Implementation

*These examples assume I'm using Ubuntu or another systemd-based distribution.*

### Step 1 — List Running Services
```bash
systemctl list-units --type=service
```
This shows all active services currently loaded by systemd.

### Step 2 — Check SSH Status
```bash
sudo systemctl status ssh
```
> On some systems the service may be named `sshd` instead of `ssh`.

### Step 3 — Check Docker (if installed)
```bash
sudo systemctl status docker
```
> If Docker is not installed, the command will show an error — that's expected.

### Step 4 — Restart a Service
```bash
sudo systemctl restart ssh
```
> ⚠️ **Caution:** Restarting SSH on a remote server while I'm connected via SSH could disconnect me if the configuration has errors. Always validate config before restarting SSH.

### Step 5 — Check Boot Configuration
```bash
systemctl is-enabled docker
```
Expected output: `enabled` or `disabled`

### Step 6 — View Service Logs
```bash
journalctl -u ssh -n 20
```
Shows the last 20 log entries for the SSH service.

### Step 7 — Follow Logs Live
```bash
journalctl -u ssh -f
```
Exit with `Ctrl + C`.

---

## 💻 Commands with Explanations

| Command | Purpose | Explanation |
|---------|---------|-------------|
| `systemctl status service` | Show service status | Displays active state, PID, recent logs |
| `systemctl start service` | Start a service | Activates a stopped service |
| `systemctl stop service` | Stop a service | Deactivates a running service |
| `systemctl restart service` | Restart a service | Full stop followed by start |
| `systemctl reload service` | Reload configuration | Re-reads config without fully stopping (if supported) |
| `systemctl enable service` | Start at boot | Creates symlink in multi-user.target |
| `systemctl disable service` | Disable auto-start | Removes the boot symlink |
| `systemctl is-enabled service` | Check boot status | Returns `enabled`, `disabled`, or `static` |
| `systemctl list-units --type=service` | List active services | Shows all loaded service units |
| `journalctl -u service` | View service logs | Displays the journal for a specific service |
| `journalctl -u service -n 50` | View last N log lines | Limit output to recent entries |
| `journalctl -u service -f` | Follow logs live | Real-time log monitoring (like `tail -f`) |
| `systemctl daemon-reload` | Reload systemd config | Required after modifying unit files |

---

## ✅ Best Practices

1. **Prefer `reload` over `restart` when supported** — Reloading reduces downtime because the service doesn't fully stop.
2. **Check service status before AND after making changes** — Always verify the current state before acting, and confirm the new state afterward.
3. **Read logs before restarting a failed service** — Logs contain the root cause. If I restart without reading logs, I lose the evidence.
4. **Enable only the services your server actually needs** — Fewer services = smaller attack surface + less resource usage.
5. **Test configuration files before restarting critical services** — For example, `nginx -t` tests Nginx config before applying it.
6. **Use `Restart=always` for critical production services** — This ensures automatic recovery after crashes.

---

## ❌ Common Mistakes

1. **Restarting services without checking logs first** — I might lose critical error messages that explain why the service failed.
2. **Disabling critical services accidentally** — Always double-check which service I'm targeting with `disable`.
3. **Editing service unit files without running `daemon-reload`** — After modifying a unit file, I must run `sudo systemctl daemon-reload` for changes to take effect.
4. **Restarting SSH remotely without validating configuration** — A typo in SSH config can lock me out of the server.
5. **Running long-lived production applications manually** — If I run `node server.js` in a terminal and the terminal closes, the app dies. Production apps should always be managed services.

---

## 💼 Interview Questions

### Beginner

**Q1: What is a Linux service?**
> A **service** (or daemon) is a background program that runs continuously, starts automatically at boot, and does not require a logged-in user session. Examples include Nginx, SSH, Docker, and MongoDB.

**Q2: What is systemd?**
> **systemd** is the service manager and init system used by most modern Linux distributions. It is responsible for starting, stopping, and managing services, as well as handling dependencies between them. It is typically **PID 1** — the first process started by the kernel.

**Q3: Which command checks service status?**
> `sudo systemctl status <service-name>` — For example, `sudo systemctl status nginx`.

**Q4: How do you restart a service?**
> `sudo systemctl restart <service-name>` — This fully stops and then starts the service.

**Q5: What is the difference between restart and reload?**
> **restart** fully stops the service and starts it again. **reload** tells the service to re-read its configuration files without completely stopping, which reduces downtime. Not all services support reload.

### Intermediate

**Q6: Why should production applications run as services?**
> Running applications as managed services provides:
> - **Automatic startup at boot** — The app starts when the server restarts.
> - **Auto-restart on failure** — With `Restart=always`, the service recovers from crashes.
> - **Log management** — Logs are collected and rotated systematically.
> - **Process isolation** — Each service runs under its own user (security).
> - **Dependency management** — Services start in the correct order.

**Q7: What is PID 1 and why is it important?**
> **PID 1** is the first process started by the kernel during boot. In modern Linux systems, this is **systemd**. It is the parent of all other processes and is responsible for initializing the system, managing services, and handling orphaned child processes.

**Q8: How do you enable a service at boot?**
> `sudo systemctl enable <service-name>` — This creates a symlink in the appropriate target directory (e.g., `multi-user.target.wants/`), telling systemd to start the service automatically at boot.

**Q9: What does journalctl provide?**
> `journalctl` provides access to the **systemd journal** — a centralized logging system that collects logs from all services managed by systemd. With `-u <service>`, I can filter logs for a specific service. With `-f`, I can follow logs in real-time.

**Q10: How would you verify that a service started successfully?**
> 1. Run `sudo systemctl status <service>` and look for `Active: active (running)`
> 2. Check the Main PID and verify the process exists: `ps -ef | grep <PID>`
> 3. Use `journalctl -u <service> -n 20` to check for error messages
> 4. Verify the service is listening on its expected port: `ss -tuln | grep <port>`

### Advanced

**Q11: Design a systemd service for a Node.js backend.**
> Here's a production-grade unit file:
> ```ini
> [Unit]
> Description=MERN Backend API
> After=network.target mongodb.service
> Requires=mongodb.service
> 
> [Service]
> ExecStart=/usr/bin/node /opt/mern-app/server.js
> WorkingDirectory=/opt/mern-app
> User=nodeapp
> Group=nodeapp
> Restart=always
> RestartSec=5
> Environment=NODE_ENV=production
> EnvironmentFile=/opt/mern-app/.env
> 
> [Install]
> WantedBy=multi-user.target
> ```
> Key design decisions:
> - `User=nodeapp` — Least privilege (not root)
> - `Restart=always` — Auto-recovery
> - `After=mongodb.service` — Ensure DB starts before the app
> - `EnvironmentFile` — Load secrets without embedding them in the unit file

**Q12: Explain why Restart=always can improve service availability.**
> `Restart=always` tells systemd to **automatically restart the service if it crashes or exits unexpectedly**. This means if the Node.js backend encounters a transient error (e.g., a temporary database connection timeout), it will be restarted without manual intervention. Combined with `RestartSec=5` (a 5-second delay between restarts), this prevents rapid restart loops while maintaining high availability.

**Q13: Describe a troubleshooting workflow for a failed service.**
> 1. **Check status:** `sudo systemctl status <service>` — Is it failed? What's the error?
> 2. **Read logs:** `journalctl -u <service> -n 50 --no-pager` — Find the root cause
> 3. **Check dependencies:** Is a required service (like MongoDB or network) running?
> 4. **Verify config:** Did a recent config change cause the failure?
> 5. **Test manually:** Try running the command manually to see the error directly
> 6. **Fix and restart:** Address the issue, then `sudo systemctl restart <service>`
> 7. **Confirm:** Verify the service is running and listening on its port

**Q14: When is reload preferable to restart?**
> **Reload** is preferable when I need to apply configuration changes without dropping active connections or causing downtime. For example, when updating Nginx's configuration:
> - `reload` gracefully spawns new worker processes with the new config and shuts down old ones
> - `restart` would terminate all connections momentarily
> Reload is ideal for web servers, load balancers, and reverse proxies where uptime matters.

**Q15: How does systemd improve operational reliability compared to manually starting applications?**
> systemd provides:
> - **Automatic restart on failure** — No manual intervention needed after crashes
> - **Dependency-based ordering** — Services start in the right sequence
> - **Resource limits** — Can restrict CPU/memory per service
> - **Log centralization** — All service logs in one place via journald
> - **Process supervision** — systemd watches the main PID and acts on its exit
> - **Boot-time parallelism** — Services start in parallel where possible, reducing boot time
> - **Socket/timer activation** — Services can start on-demand when needed

---

## 📝 Quiz (10 Questions)

Let me test my understanding:

**1. What is a Linux service?**
<details>
<summary>Answer</summary>

A **service** (daemon) is a background program that runs continuously, starts automatically, and manages itself without a logged-in user session.

</details>

**2. Which command checks service status?**
<details>
<summary>Answer</summary>

`systemctl status <service-name>` — For example, `sudo systemctl status nginx`.

</details>

**3. Which command starts a service?**
<details>
<summary>Answer</summary>

`sudo systemctl start <service-name>`

</details>

**4. Which command restarts a service?**
<details>
<summary>Answer</summary>

`sudo systemctl restart <service-name>`

</details>

**5. Which command enables startup at boot?**
<details>
<summary>Answer</summary>

`sudo systemctl enable <service-name>`

</details>

**6. Which command views service logs?**
<details>
<summary>Answer</summary>

`journalctl -u <service-name>` — For example, `journalctl -u nginx -n 20`.

</details>

**7. What is PID 1 on most modern Linux systems?**
<details>
<summary>Answer</summary>

**systemd** — It's the first process started by the kernel and manages all other services.

</details>

**8. What does `systemctl is-enabled` display?**
<details>
<summary>Answer</summary>

Whether a service is configured to start automatically at boot. Possible values: `enabled`, `disabled`, or `static`.

</details>

**9. Which command lists active services?**
<details>
<summary>Answer</summary>

`systemctl list-units --type=service`

</details>

**10. What is the difference between reload and restart?**
<details>
<summary>Answer</summary>

**reload** re-reads configuration without fully stopping the service (if supported), maintaining uptime. **restart** fully stops and starts the service, causing brief downtime.

</details>

---

## 👨‍💻 Coding / Scripting Exercise

### service_check.sh

I created a script that checks the status of important services:

```bash
#!/bin/bash

echo "=== Active Services ==="
systemctl list-units --type=service --state=running

echo
echo "=== SSH Status ==="
systemctl status ssh --no-pager

echo
echo "=== Docker Status ==="
systemctl status docker --no-pager
```

**To use it:**
```bash
chmod +x service_check.sh
./service_check.sh
```

> If Docker isn't installed, the script will show an error for Docker — that's expected and I can still check the other services.

**What I learned:**
- `--no-pager` prevents the output from being displayed page-by-page (no need to press `q` to quit)
- `--state=running` filters services to only those currently active
- The script gives me a quick bird's-eye view of critical services on any server

---

## 🧪 Hands-on Lab

### Lab: Investigate a Production Service

**Objective:** Become comfortable inspecting services before modifying them.

**Steps I completed:**

1. **List running services:**
   ```bash
   systemctl list-units --type=service --state=running
   ```

2. **Chose a service (e.g., `ssh` or `cron`):**
   - I'll use `ssh` as my example service.

3. **Check its status:**
   ```bash
   sudo systemctl status ssh
   ```

4. **View the last 20 log entries:**
   ```bash
   journalctl -u ssh -n 20 --no-pager
   ```

5. **Check whether it's enabled at boot:**
   ```bash
   systemctl is-enabled ssh
   ```

**My recorded results:**

| Field | Value |
|-------|-------|
| **Service name** | ssh |
| **Active state** | active (running) |
| **Main PID** | (varies by system) |
| **Boot status** | enabled |
| **Recent logs** | Connection attempts, auth failures, etc. |

---

## 📋 Mini Assignment

### Troubleshooting Checklist for a systemd-managed Node.js Backend

My Node.js backend is managed by systemd. Here's my troubleshooting checklist:

#### 1. How do I verify the service is running?
```bash
sudo systemctl status myapp
```
Look for `Active: active (running)`. If it shows `Active: failed`, the service has crashed.

#### 2. Where do I inspect logs?
```bash
journalctl -u myapp -n 50 --no-pager
```
For real-time monitoring:
```bash
journalctl -u myapp -f
```
Also check application-specific logs:
```bash
tail -50 /var/log/myapp/error.log
```

#### 3. How do I restart the service safely?
```bash
# Step 1: Check logs first (never restart blindly)
journalctl -u myapp -n 50

# Step 2: If needed, restart
sudo systemctl restart myapp

# Step 3: Verify it came back up
sudo systemctl status myapp
```

#### 4. How do I verify it's listening on the correct port?
```bash
ss -tuln | grep 3000
```
Expected output: `LISTEN 0.0.0.0:3000`

Also test locally:
```bash
curl -I http://localhost:3000
```

#### 5. What checks would I perform before escalating the incident?

| Check | Command | Why |
|-------|---------|-----|
| **Disk space** | `df -h` | Full disk = app can't write logs |
| **Memory usage** | `free -h` | Out of memory = app gets killed |
| **CPU usage** | `top -bn1 \| head -10` | High CPU = possible infinite loop |
| **Dependency services** | `systemctl status mongodb` | If MongoDB is down, the app can't connect |
| **Recent config changes** | `ls -lt /opt/myapp/` | Check if files were recently modified |
| **Network connectivity** | `ping -c 4 database.example.com` | App might fail due to network issues |

---

## 📚 Recommended Documentation

I should read the manual pages for deeper understanding:

```bash
man systemctl          # All systemctl commands and options
man journalctl         # Log viewing options and filtering
man systemd.service    # Service unit file format
man systemd.unit       # General unit file configuration
man systemd.exec       # Execution environment configuration
```

**Key options I discovered from `--help`:**
```bash
systemctl --help        # Quick reference
journalctl --help       # Log filtering options
```

---

## 📖 Summary

Today I learned:

✅ **What Linux services (daemons) are** — Background programs that run continuously.

✅ **The role of systemd** — The service manager (PID 1) that controls all services.

✅ **How to use systemctl** — Start, stop, restart, reload, enable, disable, and check status.

✅ **How to view service logs with journalctl** — Including real-time log following.

✅ **How to troubleshoot production services systematically** — Status → Logs → Restart → Verify.

✅ **How these concepts apply to MERN deployments** — Node.js, Nginx, MongoDB all run as managed services.

### Key Takeaways

```
Problem → systemctl status → journalctl -u → Fix → systemctl restart → ss -tuln → curl localhost
```

Managing services is a **core DevOps responsibility**. Almost every production deployment, monitoring stack, and CI/CD agent I'll work with runs as a managed service.

---

## 📝 Reflection

**What I found most important:**
- The difference between `reload` and `restart` (uptime matters!)
- Always checking logs **before** restarting a failed service
- The importance of `Restart=always` for production resilience
- How systemd's dependency system ensures services start in the right order

**What I need more practice with:**
- Writing custom unit files (coming later in the course)
- Using `journalctl` filtering options effectively
- Understanding all the fields in `systemctl status` output

---

## 🔗 Connecting to the Bigger Picture

Services connect everything I've learned so far:

- **Users & Groups** → Each service runs as a dedicated user (least privilege)
- **File Permissions** → Service files need proper permissions (644, 755, 600)
- **Networking** → Services listen on ports (verified with `ss -tuln`)
- **Processes** → Each service is a process with a PID (viewed with `ps -ef`)
- **Logs** → Service logs live in `/var/log/` and the journal

All of these concepts come together when managing production services.

---

*"In DevOps, services are the building blocks of production. Mastering systemctl means mastering how those blocks are managed."*
