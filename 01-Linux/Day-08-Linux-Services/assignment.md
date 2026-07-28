# Day 08 — Assignment: Linux Services & systemctl

## Objective
Apply what you've learned about systemd and systemctl to inspect services, create a monitoring script, and build a troubleshooting checklist for production incidents.

---

## Part 1: Coding Exercise — service_check.sh

Create a script named `service_check.sh` that inspects the status of critical system services.

### Requirements:
1. The script should display a header "=== Active Services ===" and list all running services
2. Check the status of SSH service
3. Check the status of Docker service (note: Docker may not be installed)
4. Use the `--no-pager` flag to avoid pagination

### Solution:

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

### To run:
```bash
chmod +x service_check.sh
./service_check.sh
```

> If Docker isn't installed, the script will show an error — that's expected.

---

## Part 2: Hands-on Lab — Investigate a Production Service

### Instructions:
1. List all running services on your system
2. Choose one service (e.g., `ssh`, `cron`, or `systemd-journald`)
3. Check its status
4. View the last 20 log entries
5. Check whether it's enabled at boot

### Record your findings in the table below:

| Field | Value |
|-------|-------|
| **Service name** | |
| **Active state** | |
| **Main PID** | |
| **Boot status** | |
| **Recent log entry** | |

---

## Part 3: Mini Assignment — Troubleshooting Checklist

Your Node.js backend is managed by systemd as a service named `myapp`.

Write a troubleshooting checklist answering the following:

### 1. How do you verify the service is running?

```
Command(s):
What to look for:
```

### 2. Where do you inspect logs?

```
Command(s):
Two different approaches:
```

### 3. How do you restart the service safely?

```
Step-by-step process:
```

### 4. How do you verify it's listening on the correct port?

```
Command(s):
Expected output:
```

### 5. What checks would you perform before escalating the incident?

| Check | Command | Why it matters |
|-------|---------|----------------|
| Disk space | | |
| Memory usage | | |
| CPU usage | | |
| Dependencies | | |
| Network connectivity | | |

---

## Part 4: Systemd Unit File Design

Design a systemd service unit file for a Node.js Express application with the following requirements:

- Application path: `/opt/api/server.js`
- Run as user: `nodeapp`
- Working directory: `/opt/api`
- Auto-restart on failure with 5-second delay
- Start only after network and MongoDB are available
- Load environment variables from `/opt/api/.env`
- Run in production mode

### Your Unit File:

```ini
[Unit]
Description=
After=
Requires=

[Service]
ExecStart=
WorkingDirectory=
User=
Restart=
RestartSec=
Environment=
EnvironmentFile=

[Install]
WantedBy=
```

### Solution:

```ini
[Unit]
Description=Node.js Express API
After=network.target mongodb.service
Requires=mongodb.service

[Service]
ExecStart=/usr/bin/node /opt/api/server.js
WorkingDirectory=/opt/api
User=nodeapp
Restart=always
RestartSec=5
Environment=NODE_ENV=production
EnvironmentFile=/opt/api/.env

[Install]
WantedBy=multi-user.target
```

---

## Part 5: Knowledge Check

Answer the following questions in your own words:

1. **What is the difference between `systemctl start` and `systemctl enable`?**

   <details>
   <summary>Answer</summary>

   `systemctl start` activates a service immediately in the current session. `systemctl enable` configures the service to start automatically at boot time. They do different things: start is for right now, enable is for future boots.
   </details>

2. **What happens if you modify a unit file but don't run `daemon-reload`?**

   <details>
   <summary>Answer</summary>

   systemd won't recognize the changes. The old configuration remains in effect until `sudo systemctl daemon-reload` is run to reload the unit files.
   </details>

3. **Why is `Restart=always` important for production services?**

   <details>
   <summary>Answer</summary>

   It ensures the service automatically recovers from crashes or unexpected exits without manual intervention, improving uptime and availability.
   </details>

