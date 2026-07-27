# Day 08 — Quiz: Linux Services & systemctl

## Questions

1. **What is a Linux service (daemon)?**

   <details>
   <summary>Answer</summary>

   A background program that runs continuously, starts automatically at boot, and does not require a logged-in user session. Examples: Nginx, SSH, Docker, MongoDB.
   </details>

2. **Which command checks the status of a service?**

   <details>
   <summary>Answer</summary>

   `sudo systemctl status <service-name>` — Shows whether the service is active (running), inactive (stopped), or failed.
   </details>

3. **Which command starts a service?**

   <details>
   <summary>Answer</summary>

   `sudo systemctl start <service-name>` — Activates a stopped service.
   </details>

4. **Which command restarts a service?**

   <details>
   <summary>Answer</summary>

   `sudo systemctl restart <service-name>` — Fully stops and then starts the service again.
   </details>

5. **Which command enables a service to start automatically at boot?**

   <details>
   <summary>Answer</summary>

   `sudo systemctl enable <service-name>` — Creates a symlink so systemd starts the service at boot.
   </details>

6. **Which command views logs for a specific service?**

   <details>
   <summary>Answer</summary>

   `journalctl -u <service-name>` — Displays the systemd journal entries for that service.
   </details>

7. **What is PID 1 on most modern Linux systems?**

   <details>
   <summary>Answer</summary>

   **systemd** — It is the first process started by the kernel and manages all other services and processes.
   </details>

8. **What does `systemctl is-enabled <service>` display?**

   <details>
   <summary>Answer</summary>

   Whether the service is configured to start automatically at boot. Possible values: `enabled`, `disabled`, or `static`.
   </details>

9. **Which command lists all active services?**

   <details>
   <summary>Answer</summary>

   `systemctl list-units --type=service` — Shows all loaded service units and their current states.
   </details>

10. **What is the difference between `reload` and `restart`?**

    <details>
    <summary>Answer</summary>

    **reload** tells the service to re-read its configuration files without fully stopping, minimizing downtime. **restart** fully stops the service and starts it again, causing brief downtime. Not all services support reload.
    </details>

## Bonus Questions

11. **Where are custom systemd service unit files stored?**

    <details>
    <summary>Answer</summary>

    `/etc/systemd/system/` — This is where user-defined/custom services should be placed.
    </details>

12. **What must be run after editing a unit file?**

    <details>
    <summary>Answer</summary>

    `sudo systemctl daemon-reload` — This tells systemd to reload the unit file changes.
    </details>

13. **What is the purpose of `Restart=always` in a unit file?**

    <details>
    <summary>Answer</summary>

    It tells systemd to automatically restart the service if it crashes or exits unexpectedly, improving availability.
    </details>

14. **Why should production applications run as managed services instead of manually?**

    <details>
    <summary>Answer</summary>

    Managed services provide: automatic startup at boot, auto-restart on failure, log management, process isolation (dedicated user), and dependency ordering.
    </details>

15. **Which option in `journalctl` follows logs in real-time?**

    <details>
    <summary>Answer</summary>

    `-f` (follow) — Example: `journalctl -u nginx -f`. Behaves like `tail -f`.
    </details>
