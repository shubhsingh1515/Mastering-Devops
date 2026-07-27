# Day 08 — Commands: Linux Services & systemctl

| Command | Purpose | Example |
|---------|---------|---------|
| `systemctl status <service>` | Show service status | `systemctl status nginx` |
| `systemctl start <service>` | Start a service | `sudo systemctl start nginx` |
| `systemctl stop <service>` | Stop a service | `sudo systemctl stop nginx` |
| `systemctl restart <service>` | Restart a service | `sudo systemctl restart nginx` |
| `systemctl reload <service>` | Reload configuration | `sudo systemctl reload nginx` |
| `systemctl enable <service>` | Start at boot | `sudo systemctl enable nginx` |
| `systemctl disable <service>` | Disable auto-start | `sudo systemctl disable nginx` |
| `systemctl is-enabled <service>` | Check boot status | `systemctl is-enabled nginx` |
| `systemctl list-units --type=service` | List active services | `systemctl list-units --type=service` |
| `systemctl list-units --type=service --state=running` | List running services | Filter by running state |
| `systemctl daemon-reload` | Reload systemd config | Required after editing unit files |
| `journalctl -u <service>` | View all logs for a service | `journalctl -u nginx` |
| `journalctl -u <service> -n 50` | View last N log lines | `journalctl -u nginx -n 50` |
| `journalctl -u <service> -f` | Follow logs in real-time | `journalctl -u nginx -f` |
| `journalctl -u <service> --no-pager` | View logs without pager | `journalctl -u nginx --no-pager` |

## Quick Reference

```bash
# Check if a service is running
sudo systemctl status <service>

# Restart a failed service (after checking logs!)
sudo systemctl restart <service>

# Make a service start on boot
sudo systemctl enable <service>

# View recent errors
journalctl -u <service> -n 20 --no-pager
