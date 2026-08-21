# DevOps Mentorship Program - Day 20

## Sunday Weekly Revision - Linux Phase Final Assessment

- **Focus:** Full Linux-phase revision, production troubleshooting, quiz, practical assessment, interview preparation, and Month 1 project review

### Linux Phase Complete

- Day 1 - Linux Fundamentals
- Day 2 - Linux File System
- Day 3 - Shell Commands
- Day 4 - File Permissions and Ownership
- Day 5 - Users and Groups
- Day 6 - Linux Networking
- Day 7 - Processes
- Day 8 - Services and systemctl
- Day 9 - Package Managers
- Day 10 - SSH
- Day 11 - Bash Scripting
- Day 12 - Cron
- Day 13 - Log Management
- Day 14 - Archives and Backups
- Day 15 - Weekly Revision
- Day 16 - Storage and Disk Management
- Day 17 - Performance Monitoring
- Day 18 - Security and Hardening
- Day 19 - Networking and Troubleshooting
- Day 20 - Linux Phase Final Revision and Assessment

**Linux Phase Status:** 19 / 19 topics complete


## 1) Quick Review (With Meaning)

### Q1. What is the difference between `df` and `du`?
- `df`: Shows filesystem-level capacity and free/used space (how full the partition is).
- `du`: Shows directory/file-level usage (which folder is consuming space).

Why this matters in production:
- Use `df` to detect partition pressure.
- Use `du` to find the source of that pressure.

Practical example:
- If users report upload failures, run `df -h` first to check whether `/var` or `/` is full.
- If disk usage is high, run `du -sh /var/* | sort -h` to identify heavy directories.

### Q2. What does `systemctl enable nginx` do?
It configures Nginx to start automatically at boot by creating required systemd links.

Why this matters:
- Service survival after reboot is part of reliability.

Operational note:
- `enable` changes boot behavior.
- `start` changes current runtime state.
- In production, you usually need both: `enable` and `start`.

### Q3. Difference between `connection refused` vs `connection timed out`?
- Refused: Destination is reachable, but no service accepted the connection on that port.
- Timed out: No response within time; often firewall/routing/reachability/filtering issues.

Why this matters:
- Refused usually points to process/port/service issue.
- Timed out usually points to network path/security controls.

Troubleshooting shortcut:
- Refused: check `systemctl status`, `ss -tulpn`, process crash loops.
- Timed out: check firewall (`ufw`/security groups), routes, DNS, upstream reachability.

### Q4. Why should a Node.js production app not run as `root`?
If the app is compromised, root privileges allow full system takeover.

Why this matters:
- Always apply least privilege and run app with a restricted user.

Security depth:
- Running as non-root limits blast radius.
- File permissions, sudo scope, and service account ownership should align.

### Q5. What command checks inode usage?
`df -i`

Why this matters:
- You can have free disk blocks but still fail to create files due to inode exhaustion.

Common real-world cause:
- Millions of tiny files (cache/temp/session artifacts) consume inodes faster than disk space.

---

## 2) Linux Phase Capability Map

At this stage, Linux knowledge should work as a connected operations system, not isolated commands.

```text
Linux Server
    |
    +-- Users
    |   +-- Permissions
    |
    +-- Network
    |   +-- Ports
    |
    +-- Storage
    |   +-- Filesystems
    |
    +-- Processes
    +-- systemd
    +-- Applications
        +-- Logs
        +-- Backups
            +-- Automation
            +-- Security
```

Key transition:
- Old mindset: "I know `systemctl`."
- New mindset: "I can combine services, logs, processes, networking, storage, and permissions to resolve production incidents."

What mastery looks like:
- You do not stop at one green status output.
- You verify end-to-end behavior from socket to HTTP to dependency response.
- You correlate symptoms across logs, metrics, and process state.

---

## 3) Linux Interview Rapid-Fire (15-30 Seconds Each)

### 1. Service does not start. What do you do?
Flow:
1. `systemctl status <service>`
2. `journalctl -u <service>`
3. Validate configuration file syntax
4. Check dependencies and ordering
5. Check ports/process collisions

What interviewer is evaluating:
- Whether you follow a deterministic sequence.
- Whether you can distinguish service-manager issues from app-level failures.
- Whether you avoid random restarts before gathering evidence.

### 2. Full disk investigation flow
1. `df -h` for block usage
2. `df -i` for inode usage
3. `du` to find heavy directories
4. `find` for old logs/temp/large files
5. Clean safely and validate service health

Good answer signal:
- You mention safe cleanup strategy (archive, rotate, delete with validation).
- You mention post-cleanup verification for impacted services.

### 3. 502 from Nginx troubleshooting
1. Check Nginx status/logs
2. Check Node.js service status
3. Verify Node is listening on expected port
4. `curl http://localhost:<node-port>/health`
5. Read app logs and dependency logs (DB/cache)

Strong diagnostic reasoning:
- 502 means proxy received an invalid/missing upstream response.
- Root cause may be app crash, slow app, wrong upstream config, or dependency latency.

### 4. SSH hardening checklist
- Key-based authentication
- Protect private keys
- Least privilege user model
- Avoid direct root login
- Restrict network exposure
- Disable password auth where suitable
- Monitor auth logs

Security objective:
- Reduce brute-force surface and privilege escalation paths.

### 5. Slow Node.js app troubleshooting
Start with:
- `top`
- `free -h`
- `uptime`
- `ps aux --sort=-%cpu`
- `ps aux --sort=-%mem`

Then analyze:
- Logs and error rates
- Disk I/O behavior
- Network latency/connections
- Database response patterns
- App-level workload/spikes

Interview-quality framing:
- Separate symptom from bottleneck.
- Show that high CPU is not always the root cause; it can be a consequence.

---

## 4) Final Linux Quiz

1. Which command shows service logs?
- A. `systemctl logs`
- B. `journalctl -u service`
- C. `service-log`
- D. `tail systemd`

2. Which command shows listening sockets?
- A. `ss -tuln`
- B. `listen`
- C. `ports -a`
- D. `netstat-files`

3. Which command checks memory?
- A. `df -h`
- B. `du -sh`
- C. `free -h`
- D. `memcheck`

4. What does `$?` represent?
- A. PID
- B. Previous command exit status
- C. Current user
- D. CPU usage

5. Which command creates a gzip compressed tar archive?
- A. `tar -czf`
- B. `tar -xzf`
- C. `gzip -tar`
- D. `zip -tar`

6. Which file contains SSH server configuration?
- A. `/etc/ssh/sshd_config`
- B. `~/.ssh/config`
- C. `/etc/ssh/public`
- D. `/var/ssh/server`

7. What does `*/5 * * * *` mean?
- A. Every five hours
- B. Every five minutes
- C. Every fifth day
- D. Every five months

8. What does `127.0.0.1` represent?
- A. Default gateway
- B. Loopback
- C. Broadcast
- D. Public IP

9. Which command shows CPU-heavy processes?
- A. `ps aux --sort=-%cpu`
- B. `df --cpu`
- C. `cpu top`
- D. `top --disk`

10. Which principle says a process should receive only the privileges it needs?
- A. Root-first
- B. Least privilege
- C. Maximum access
- D. Shared credentials

11. Can a filesystem have free space but still fail to create files?
- A. No
- B. Yes, inode exhaustion is possible

12. Why use `curl http://localhost:3000/health` while debugging Nginx?
- A. It bypasses reverse proxy and tests Node directly
- B. It restarts Nginx
- C. It checks disk space
- D. It tests SSH

13. Which command checks DNS resolution?
- A. `dig`
- B. `df`
- C. `du`
- D. `ps`

14. Why avoid immediate `kill -9` for a high-CPU Node process?
- A. It does not work
- B. It blocks graceful shutdown and destroys troubleshooting evidence
- C. It only works on Windows
- D. It changes DNS

15. Primary purpose of log rotation?
- A. Encrypt logs
- B. Prevent uncontrolled log growth
- C. Restart Node
- D. Change ownership

### Answer Key
`B A C B A A B B A B B A A B B`

### Scoring Guide
- 14-15: Excellent readiness
- 12-13: Strong readiness
- 10-11: Review weak areas
- 7-9: Needs reinforcement
- <7: Revisit Linux phase

How to improve after scoring:
- If low in process/service questions: redo Days 7 and 8 with live practice.
- If low in storage/logging: redo Days 13, 14, and 16 with command drills.
- If low in network/security: redo Days 6, 10, 18, and 19 with scenario practice.

---

## 5) Practical Assessment - Production Incident

### Scenario
Production MERN path:

```text
Internet
  |
  v
Nginx :443
  |
  v
Node.js :3000
  |
  v
MongoDB :27017
```

Users report:
- Intermittent 502 errors
- Some requests take around 10 seconds

You have SSH access.

### Your Mission
Determine:
- Nginx health
- Node.js health
- Node port listening state
- Local app response
- MongoDB health
- CPU and memory pressure
- Disk and inode health
- Network/firewall influence
- Log signals
- Most probable root cause

### Practical Decision Flow (What to Conclude at Each Step)

1. **Nginx status and logs**
- If Nginx is down, restore service and check why it failed.
- If Nginx is up but logs show upstream timeout/refused, move to Node checks.

2. **Node service and listening port**
- If Node service is inactive/crashing, inspect `journalctl` and app logs.
- If service is active but port missing, validate bind address and app startup behavior.

3. **Local health probe**
- If `curl localhost:3000/health` is slow/failing, issue is likely app or dependency-side.
- If local probe is healthy but users still fail, investigate Nginx config, TLS, or external path.

4. **MongoDB and dependency checks**
- If app logs show DB timeouts, validate MongoDB service health and connection latency.

5. **Resource pressure**
- High load + high CPU queue: possible compute saturation.
- Low free memory + swap pressure: memory contention.
- High disk utilization or inode exhaustion: write-path failures and latency.

6. **Network and firewall**
- Validate expected interface/routes/firewall rules.
- Distinguish host firewall from cloud security boundary issues.

7. **Root cause statement**
- Always produce one evidence-backed sentence:
  - Symptom
  - Direct technical cause
  - Supporting evidence
  - Immediate mitigation
  - Long-term prevention

### Suggested Investigation Commands

#### Services
```bash
sudo systemctl status nginx
sudo systemctl status myapp
sudo systemctl status mongod
```

#### Ports
```bash
sudo ss -tulpn
```

#### Application probe
```bash
curl -v http://localhost:3000/health
```

#### Logs
```bash
journalctl -u myapp -n 100
sudo tail -100 /var/log/nginx/error.log
journalctl -u mongod -n 100
```

#### Performance
```bash
uptime
free -h
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

#### Storage
```bash
df -h
df -i
```

#### Network
```bash
ip addr
ip route
sudo ufw status verbose
```

### Example Root Cause Write-up

"Intermittent 502 errors were caused by Node.js request latency spikes due to MongoDB connection pool exhaustion. Evidence: Nginx error logs showed upstream timeout, `curl localhost:3000/health` intermittently took >8s, and app logs showed MongoDB wait queue warnings. Immediate mitigation was restarting the app and temporarily scaling worker count; prevention includes tuning pool size, adding DB latency alerts, and implementing backpressure."

---

## 6) Most Important Skill: Reasoning Over Raw Output

Do not stop at command execution. Convert output into conclusions.

Example 1:
- `systemctl status myapp` = active
- This does not guarantee app health.
- Continue with socket check, HTTP health endpoint, then app logs.

Example 2:
- `df -h` shows normal usage
- Storage can still fail due to inode exhaustion.
- Always cross-check with `df -i`.

Example 3:
- `ping` fails
- That does not always mean app unreachable (ICMP may be blocked while TCP/HTTPS works).

Operational maturity = evidence chain + accurate interpretation.

### Common Troubleshooting Mistakes to Avoid

- Restarting services repeatedly before collecting logs.
- Assuming green service status means healthy app behavior.
- Checking only disk blocks and skipping inode checks.
- Treating timeout and refused as the same network problem.
- Ignoring dependency health (database, DNS, third-party APIs).

---

## 7) Mock Interview Framework

Question:
"You are on-call. A production MERN app is slow and occasionally returns 502. What do you do?"

Strong response structure:
1. Confirm impact, scope, and timeline.
2. Trace request path: DNS -> Nginx -> Node -> MongoDB.
3. Validate service states (`systemctl status`).
4. Validate listening ports (`ss -tulpn`).
5. Test app locally (`curl localhost:3000/health`).
6. Inspect logs (`journalctl`, `tail`, `grep`).
7. Review resources (`top`, `free`, disk, I/O).
8. Check dependencies and network behavior.
9. Apply low-risk mitigation backed by evidence.
10. Define prevention: monitoring, alerts, capacity, config/app fixes.

What makes this answer strong:
- It is ordered, testable, and low-risk.
- It communicates engineering discipline under pressure.
- It balances diagnosis, mitigation, and prevention.

---

## 8) Month 1 Cumulative Project - Final Assessment

### Project Theme
Production MERN Linux Server demonstrating complete operations lifecycle.

### Target Architecture

```text
Internet
  |
80 / 443
  |
  v
Nginx
  |
  v
Node.js API
  |
  v
MongoDB
```

### Security Baseline
- SSH keys
- Dedicated admin user
- Controlled `sudo`
- Least privilege services
- Firewall restrictions
- Minimal exposure

### Reliability Baseline
- systemd-managed services
- Health checks
- Structured logging
- Backups
- Tested restore path

### Monitoring Baseline
- CPU
- RAM
- Disk blocks
- Inodes
- Network state
- Service and app health

### Automation Baseline
- `deploy.sh`
- `backup.sh`
- `health_check.sh`
- `disk_health.sh`
- `performance_check.sh`
- `network_health.sh`

### Minimum Evidence to Demonstrate Project Completeness

- Screenshot or output of active systemd services for app and Nginx.
- Nginx virtual host config with reverse proxy and TLS.
- Health endpoint returning success locally and through Nginx.
- Backup and restore test evidence (not just backup creation).
- Firewall policy showing least exposed ports.
- Monitoring outputs for CPU, memory, disk, and inode checks.

---

## 9) Month 1 Project Interview Preparation

Prompt to answer confidently:
"How would you transform a fresh Ubuntu server into a secure production MERN server?"

Expected logical flow:
1. Update system packages
2. Create users and access model
3. Harden SSH
4. Apply permission model
5. Install runtime/dependencies
6. Deploy application
7. Configure Node service (systemd)
8. Configure Nginx reverse proxy
9. Apply firewall rules
10. Protect secrets/config
11. Set up logging
12. Configure backups and restore validation
13. Add health checks
14. Add monitoring/alerts
15. Test failure and recovery scenarios

Do not memorize this mechanically. Explain the reasoning behind order and risk reduction.

Interview tip:
- Speak in phases: harden, deploy, verify, monitor, recover.
- Mention validation after each phase to show production thinking.

---

## 10) Linux Phase Readiness Check

You are ready for next stages if you can explain and demonstrate:
- Navigation and filesystem operations
- Permissions and ownership
- Users and groups
- Processes and process control
- Services and systemd
- Package management
- SSH security and remote admin
- Bash scripting
- Cron automation
- Logging and analysis
- Backups and restore concepts
- Storage and disk diagnosis
- Performance troubleshooting
- Security hardening
- Networking diagnostics

Self-check rubric:
- Can you diagnose a 502 without guessing?
- Can you prove whether issue is network, service, app, or dependency?
- Can you explain least privilege implementation decisions?
- Can you recover from disk/inode incidents safely?
- Can you describe both mitigation and long-term prevention?

---

## Linux Phase Complete

Next progression:

```text
Linux
  -> Docker and Containers
  -> Git and CI/CD
  -> Cloud
  -> Infrastructure as Code
  -> Configuration Management
  -> Monitoring
  -> Kubernetes
  -> Production DevOps Project
```

Your Linux foundation remains active in every future phase.

Important transition:
- From managing individual servers
- To packaging, deploying, automating, and operating applications at scale
