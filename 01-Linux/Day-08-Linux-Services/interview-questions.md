# Day 08 — Interview Questions: Linux Services & systemctl

## Beginner

### Q1: What is a Linux service?
A **service** (also called a daemon) is a background program that runs continuously, starts automatically at boot, and does not require a logged-in user session. Examples include Nginx (web server), SSH (remote access), Docker (container runtime), and MongoDB (database).

### Q2: What is systemd?
**systemd** is the service manager and init system used by most modern Linux distributions (Ubuntu, Debian, CentOS, Rocky Linux). It is responsible for starting, stopping, and managing services, handling dependencies between them, and collecting logs. It is typically **PID 1** — the first userspace process started by the kernel.

### Q3: Which command checks service status?
```bash
sudo systemctl status <service-name>
```
For example: `sudo systemctl status nginx` shows whether Nginx is active (running), inactive (stopped), or failed.

### Q4: How do you restart a service?
```bash
sudo systemctl restart <service-name>
```
This fully stops the service and starts it again. It should only be done after checking logs to understand why the service might have failed.

### Q5: What is the difference between restart and reload?
- **restart** fully stops the service and starts it again, causing brief downtime
- **reload** tells the service to re-read its configuration files without fully stopping, minimizing downtime
- Not all services support reload (check the service documentation)

---

## Intermediate

### Q6: Why should production applications run as services?
Running applications as managed services provides several benefits:
1. **Automatic startup at boot** — The app comes back up after a server reboot
2. **Auto-restart on failure** — With `Restart=always`, the service recovers from crashes automatically
3. **Log management** — Logs are collected centrally via journald
4. **Process isolation** — Each service runs under its own dedicated user (security)
5. **Dependency management** — Services start in the correct order (e.g., MongoDB before the Node.js app)

### Q7: What is PID 1 and why is it important?
**PID 1** is the first process started by the kernel during boot. In modern Linux systems, this is **systemd**. It is important because:
- It is the parent of all other processes
- It manages service startup, shutdown, and monitoring
- It adopts orphaned child processes
- If PID 1 crashes, the system panics

### Q8: How do you enable a service at boot?
```bash
sudo systemctl enable <service-name>
```
This creates a symlink from the service unit file into the `multi-user.target.wants/` directory, telling systemd to start the service automatically at boot.

### Q9: What does journalctl provide?
`journalctl` provides access to the **systemd journal** — a centralized logging system that collects:
- Logs from all services managed by systemd
- Kernel messages
- Authentication logs

Useful options:
- `-u <service>` — Filter logs for a specific service
- `-n 50` — Show last 50 lines
- `-f` — Follow logs in real-time (like `tail -f`)
- `--no-pager` — Output without pagination

### Q10: How would you verify that a service started successfully?
1. Run `sudo systemctl status <service>` and look for `Active: active (running)`
2. Check the Main PID and verify: `ps -ef | grep <PID>`
3. Review recent logs: `journalctl -u <service> -n 20 --no-pager`
4. Verify the port: `ss -tuln | grep <port>`
5. Test the endpoint: `curl -I http://localhost:<port>`

---

## Advanced

### Q11: Design a systemd service for a Node.js backend.
```ini
[Unit]
Description=MERN Backend API
After=network.target mongodb.service
Requires=mongodb.service

[Service]
ExecStart=/usr/bin/node /opt/mern-app/server.js
WorkingDirectory=/opt/mern-app
User=nodeapp
Group=nodeapp
Restart=always
RestartSec=5
Environment=NODE_ENV=production
EnvironmentFile=/opt/mern-app/.env

[Install]
WantedBy=multi-user.target
```
Key design decisions:
- `User=nodeapp` — Runs as a dedicated service user (least privilege)
- `Restart=always` — Automatic recovery from crashes
- `After=mongodb.service` — Ensures database starts before the app
- `EnvironmentFile` — Loads secrets from a separate file (not embedded in the unit)

### Q12: Explain why Restart=always can improve service availability.
`Restart=always` tells systemd to **automatically restart the service if it crashes or exits unexpectedly**, regardless of the exit code. This means transient errors (e.g., database connection timeout, memory spike) don't cause prolonged downtime. Combined with `RestartSec=5` (a 5-second delay between restart attempts), this prevents rapid restart loops while maintaining availability. For critical production services, this is essential for meeting uptime SLAs.

### Q13: Describe a troubleshooting workflow for a failed service.
1. **Check status:** `sudo systemctl status <service>` — Determine if it's failed and see the error message
2. **Read logs:** `journalctl -u <service> -n 50 --no-pager` — Understand the root cause
3. **Check dependencies:** Are required services (database, network) running?
4. **Verify configuration:** Did a recent change cause the failure? Check config files
5. **Test manually:** Run the command from `ExecStart` directly to see the error
6. **Fix the issue:** Address the root cause (config fix, disk cleanup, dependency restart)
7. **Restart the service:** `sudo systemctl restart <service>`
8. **Confirm:** Verify it's running (`status`) and listening (`ss -tuln`)

### Q14: When is reload preferable to restart?
**Reload** is preferable when applying configuration changes to services that handle active connections, such as:
- **Web servers** (Nginx, Apache) — reloading spawns new worker processes with the new config and gracefully shuts down old ones
- **Reverse proxies** — avoids dropping in-flight requests
- **Load balancers** — maintains uptime during config updates

Reload should be used when:
- The service supports reload (check with `--help` or man page)
- The configuration change is minor (e.g., adding a new virtual host)
- Minimizing downtime is critical

### Q15: How does systemd improve operational reliability compared to manually starting applications?
systemd provides several reliability improvements:
1. **Automatic restart** — Services recover from crashes without human intervention
2. **Dependency ordering** — Services start in the correct sequence, preventing race conditions
3. **Process supervision** — systemd monitors the main PID and acts on its exit status
4. **Resource limits** — Can restrict CPU, memory, and file descriptors per service
5. **Log centralization** — All service logs are collected via journald for easier debugging
6. **Boot-time parallelism** — Independent services start simultaneously, reducing boot time
7. **Socket/timer activation** — Services can start on-demand when needed
8. **Sandboxing** — Can restrict service capabilities (read-only filesystem, no network, etc.)

These features make production systems more resilient and easier to operate compared to manually starting applications in terminal sessions.
