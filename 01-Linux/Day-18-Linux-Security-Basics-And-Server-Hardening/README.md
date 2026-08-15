# Day 18: Linux Security Basics & Server Hardening

## 📅 Phase 1: Linux
- Topic: Linux Security Basics & Server Hardening
- Estimated Time: 25–30 minutes
- Focus: Least privilege, sudo, SSH hardening, firewall basics, open ports, and securing a production MERN server.

> Today is Friday, August 14, 2026, so this is a new-topic day. Sunday’s lesson will be the weekly revision instead of a new topic.

---

## 📊 Progress Tracker

### ✅ Completed
- Day 1 — Linux Fundamentals
- Day 2 — Linux File System
- Day 3 — Shell Commands
- Day 4 — File Permissions & Ownership
- Day 5 — Users & Groups
- Day 6 — Linux Networking
- Day 7 — Processes
- Day 8 — Services & systemctl
- Day 9 — Package Managers
- Day 10 — SSH
- Day 11 — Bash Scripting
- Day 12 — Cron
- Day 13 — Log Management
- Day 14 — Archives & Backups
- Day 15 — Weekly Revision
- Day 16 — Storage & Disk Management
- Day 17 — Performance Monitoring
- Day 18 — Linux Security & Hardening

Linux Phase: 18 / 19

---

## 🔁 1. Quick Review

### Q1. Which command checks filesystem capacity?
```bash
df -h
```

### Q2. Which command shows the processes consuming the most CPU?
```bash
ps aux --sort=-%cpu | head
```

### Q3. Which command checks listening ports?
```bash
ss -tuln
```

### Q4. What does chmod 600 achieve for a private SSH key?
It gives the owner read/write access while denying access to group and other users.

### Q5. Which command shows recent logs for a systemd service?
```bash
journalctl -u <service> -n 50
```

---

## 1. 📚 Topic Name

### Linux Security & Server Hardening

Security in DevOps is not a single tool.

It is a collection of controls:

```text
Users
  ↓
Permissions
  ↓
SSH
  ↓
Firewall
  ↓
Services
  ↓
Updates
  ↓
Logs
  ↓
Monitoring
```

The goal is to reduce:

- Attack surface
- Unauthorized access
- Privilege escalation
- Accidental damage
- Exposure of production services

---

## 2. 🎯 Learning Objectives

By the end of today's lesson, you'll be able to:

- Explain least privilege.
- Use sudo safely.
- Understand Linux ownership and permissions in a security context.
- Harden SSH conceptually and practically.
- Configure basic firewall rules with ufw.
- Identify unnecessarily exposed ports.
- Secure a MERN production server.
- Answer common Linux security interview questions.

---

## 3. 🔐 Principle of Least Privilege

The principle is simple:

Give a user or process only the permissions it actually needs.

### Bad
```text
Node.js application
        ↓
       root
```

### Better
```text
Node.js application
        ↓
    nodeapp user
```

If the application is compromised, the attacker does not automatically receive unrestricted root privileges.

This is one of the most important security ideas in Linux and DevOps.

---

## 4. 👤 sudo

Instead of logging in directly as root:

```text
root
```

Use a normal administrative account:

```bash
sudo systemctl restart nginx
```

sudo temporarily grants the required elevated privilege for a command.

Check your sudo permissions:

```bash
sudo -l
```

The idea is to grant admin permission only when needed, not by default.

---

## 5. ⚠️ Why Running MERN as Root Is Dangerous

Suppose:

```text
Node.js
   ↓
root
```

And an application vulnerability allows arbitrary command execution.

The attacker may potentially execute commands with root privileges.

Prefer:

```text
Node.js
   ↓
nodeapp
   ↓
limited permissions
```

The same principle applies to:

- Nginx
- Cron jobs
- Backup scripts
- CI/CD agents
- Database services

Running services as a dedicated non-root user limits blast radius.

---

## 6. 🔑 SSH Hardening

From Day 10, you learned key-based SSH authentication.

A production server should generally prefer:

```text
SSH key
   ↓
Authentication
   ↓
Normal user
   ↓
sudo when required
```

Important SSH security controls include:

- Key-based authentication
- Strong key protection
- Disabling password authentication where appropriate
- Avoiding direct root SSH login
- Restricting SSH access through firewall/network controls
- Keeping OpenSSH patched

---

## 7. SSH Configuration

The server configuration is commonly:

```bash
/etc/ssh/sshd_config
```

Before changing SSH configuration, validate it:

```bash
sudo sshd -t
```

If the configuration is valid, restart or reload carefully.

On many systems:

```bash
sudo systemctl reload ssh
```

or the service may be named:

```bash
sshd
```

### Production warning
Never casually change SSH settings on a remote production server.

A bad configuration can lock you out.

A safe operational sequence is:

```text
Backup configuration
       ↓
Edit carefully
       ↓
Validate with sshd -t
       ↓
Keep existing session open
       ↓
Reload
       ↓
Test a new connection
```

---

## 8. 🔥 Firewall Basics

A firewall controls network traffic.

Ubuntu commonly uses:

```bash
ufw
```

Check status:

```bash
sudo ufw status
```

More detail:

```bash
sudo ufw status verbose
```

### Allow SSH
Before enabling the firewall remotely:

```bash
sudo ufw allow ssh
```

or:

```bash
sudo ufw allow 22/tcp
```

Then:

```bash
sudo ufw enable
```

### Allow HTTP/HTTPS
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

---

## 9. 🚫 Don't Expose Your Node.js Port Unnecessarily

Suppose your architecture is:

```text
Internet
   │
   ▼
 Nginx :443
   │
   ▼
 Node.js :3000
```

There is usually no reason for arbitrary Internet clients to connect directly to:

```text
:3000
```

Instead:

```text
Internet
   │
   ├── 80
   └── 443
        │
       Nginx
        │
      :3000
        │
      Node.js
```

The Node.js service can be bound to localhost or otherwise restricted so it is not unnecessarily Internet-facing.

---

## 10. 🌍 MERN Production Example

Imagine:

```text
Public IP
   │
   ├── 22  SSH
   ├── 80  HTTP
   ├── 443 HTTPS
   ├── 3000 Node.js
   └── 27017 MongoDB
```

This is a red flag.

You generally want to minimize public exposure.

A more appropriate architecture is:

```text
Internet
   │
   ├── 80
   └── 443
       │
      Nginx
       │
    Node.js
       │
    MongoDB
```

SSH access on port 22 should also be restricted appropriately for your environment.

MongoDB should not be casually exposed to the public Internet.

---

## 11. 🔍 Identify Open Ports

Use:

```bash
ss -tuln
```

Example output:

```text
LISTEN 0 128 0.0.0.0:22
LISTEN 0 511 0.0.0.0:80
LISTEN 0 511 0.0.0.0:443
LISTEN 0 511 127.0.0.1:3000
```

### Interpretation

```text
0.0.0.0:22
```
means the service is listening on all IPv4 interfaces.

Whereas:

```text
127.0.0.1:3000
```
means it is reachable only locally through that IPv4 loopback interface.

This distinction is extremely useful in production troubleshooting.

---

## 12. 🛡️ File Permissions as a Security Control

From earlier lessons:

```bash
chmod 600 .env
```

means:

- Owner → read/write
- Group → no access
- Others → no access

For an application:

```text
/opt/mern/
```

the application user should own only what it needs.

Do not solve every permission problem with:

```bash
chmod -R 777
```

That is a major security smell.

---

## 13. ❌ Why 777 Is Dangerous

```bash
chmod 777 file
```

gives:

- Owner → read/write/execute
- Group → read/write/execute
- Others → read/write/execute

Everyone can potentially modify the file.

Instead, determine:

- Who needs access
- What level of access do they need?

Then assign the minimum necessary permissions.

---

## 14. 🛠️ Practical Exercise

Check your current identity:

```bash
whoami
```

Check your groups:

```bash
groups
```

Check sudo privileges:

```bash
sudo -l
```

Check SSH directory permissions:

```bash
ls -ld ~/.ssh
```

Check private key permissions:

```bash
ls -l ~/.ssh/id_ed25519 2>/dev/null
```

Check listening ports:

```bash
ss -tuln
```

If ufw is installed:

```bash
sudo ufw status verbose
```

---

## 15. 💻 Basic UFW Production Model

For a server where Nginx is the public entry point:

```bash
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Then inspect:

```bash
sudo ufw status numbered
```

Only enable the firewall after ensuring your required administrative access is permitted.

---

## 16. 🔗 Connect Today's Lesson to Earlier Lessons

You now have a security chain across the curriculum:

```text
Users
 ↓
Groups
 ↓
Permissions
 ↓
SSH
 ↓
Firewall
 ↓
Services
 ↓
Logs
 ↓
Monitoring
```

For example:

```text
Unauthorized access attempt
          ↓
         SSH
          ↓
        logs
          ↓
    investigation
          ↓
 firewall / access controls
          ↓
       mitigation
```

This is how individual Linux topics become an operational security workflow.

---

## 17. 💼 Interview Preparation

### Beginner

#### Q1. What is least privilege?
Giving users and processes only the permissions necessary to perform their tasks.

#### Q2. Why shouldn't applications run as root?
A compromise of the application could otherwise provide the attacker with root-level privileges and greatly increase the impact.

#### Q3. What does a firewall do?
It controls network traffic according to defined rules, reducing unwanted network exposure.

### Intermediate

#### Q4. How would you secure SSH on a production server?
A strong answer:

```text
Use key-based authentication
        ↓
Protect private keys
        ↓
Avoid direct root login
        ↓
Disable password authentication where appropriate
        ↓
Restrict network access
        ↓
Keep OpenSSH patched
        ↓
Monitor authentication logs
```

#### Q5. A MERN server has ports 22, 80, 443, 3000, and 27017 open publicly. What concerns you?
The direct exposure of:

- 3000 → Node.js
- 27017 → MongoDB

would be a major concern.

I would determine whether these services need public access and restrict them to trusted networks/interfaces where possible.

---

## 18. 🎤 Advanced Interview Scenario

"You inherited an Ubuntu server running a MERN application. SSH, Node.js, MongoDB, and Nginx are all exposed to the Internet. What would you do?"

A strong answer should not be:

"Close all the ports."

Instead:

```text
Inventory services
      ↓
Identify required traffic
      ↓
Identify application dependencies
      ↓
Restrict unnecessary exposure
      ↓
Harden SSH
      ↓
Apply least privilege
      ↓
Protect secrets
      ↓
Configure firewall
      ↓
Verify connectivity
      ↓
Monitor logs
```

You want to preserve required functionality while reducing the attack surface.

---

## 19. 📝 Quiz

### 1. What principle says users should receive only necessary permissions?
A. Defense by obscurity
B. Least privilege
C. Maximum privilege
D. Root-first administration

### 2. What command checks listening ports?
A. ss -tuln
B. ports -a
C. netports
D. systemctl ports

### 3. What is the purpose of sudo?
A. Compress files
B. Temporarily execute commands with elevated privileges
C. Create SSH keys
D. Start the firewall

### 4. Why is chmod 777 generally inappropriate for production application files?
A. It makes them read-only
B. It prevents execution
C. It grants overly broad permissions
D. It encrypts the files

### 5. What file commonly contains the SSH server configuration?
A. /etc/ssh/sshd_config
B. /etc/ssh/public
C. /var/ssh/config
D. /etc/ssh/firewall

### 6. Which command validates SSH daemon configuration?
A. ssh --check
B. sshd -t
C. ssh-test
D. systemctl validate ssh

### 7. Which ports would typically be needed for an HTTPS Nginx reverse proxy?
A. 3000 only
B. 27017 only
C. 80 and/or 443
D. 3306

### 8. Why should MongoDB generally not be publicly exposed without a compelling reason and strong controls?
A. MongoDB cannot use networking
B. It unnecessarily increases attack surface and potential unauthorized database access
C. It makes Nginx slower
D. It disables SSH

### 9. What does 127.0.0.1:3000 imply?
A. Port 3000 is listening on every interface
B. Port 3000 is locally bound to loopback
C. Port 3000 is blocked
D. Port 3000 belongs to SSH

### 10. What should you do before enabling a firewall on a remote server?
A. Remove SSH rules
B. Ensure required management access is explicitly allowed
C. Reboot
D. Delete existing network interfaces

### ✅ Answer Key
- B
- A
- B
- C
- A
- B
- C
- B
- B
- B

---

## 20. 🧪 Practical Assessment

### Secure a Hypothetical MERN Server

#### Starting state
- SSH → public
- Nginx → public
- Node.js → public
- MongoDB → public
- Application → root
- .env → 644

Design the target state.

### Target architecture
```text
                 Internet
                    │
              ┌─────┴─────┐
              │           │
             HTTP       HTTPS
              │           │
              └─────┬─────┘
                    ▼
                  Nginx
                    │
                    ▼
             Node.js :3000
              localhost
                    │
                    ▼
              MongoDB
              restricted
```

SSH should be available only through an appropriate administrative access path.

### Your assessment
Explain how you would:

- Create or use a dedicated application user.
- Remove unnecessary root execution.
- Protect .env.
- Harden SSH.
- Configure firewall rules.
- Restrict Node.js exposure.
- Restrict MongoDB exposure.
- Verify open ports.
- Verify application functionality after hardening.
- Monitor authentication and service logs.

---

## 🏗️ Month 1 Cumulative Project Update

### Production MERN Linux Server
Your project now needs a security hardening section.

Add these requirements:

```text
Security
├── Dedicated application user
├── SSH key authentication
├── Protected private keys
├── No unnecessary root execution
├── Restricted application permissions
├── Protected .env
├── Firewall
├── Minimal public ports
├── Restricted MongoDB
└── Security log monitoring
```

Your production architecture should now resemble:

```text
                 Internet
                    │
                80 / 443
                    │
                    ▼
                  Nginx
                    │
                    ▼
               Node.js API
                    │
                    ▼
                 MongoDB
```

And alongside it:

```text
SSH ──→ Admin User ──→ sudo
                         │
                    systemctl
                         │
                    Operations
```

---

## 🎤 21. Final Interview Challenge

Answer aloud:

"How would you harden a Linux server hosting a production MERN application?"

Your answer should cover:

1. Users & least privilege
2. SSH hardening
3. File permissions
4. Firewall
5. Minimal exposed ports
6. Application isolation
7. Secret protection
8. Security updates
9. Logging and monitoring
10. Regular review

The strongest answers explain why each control exists, not just which command configures it.

---

## 📖 22. Today's Key Takeaways

You learned:

- ✅ Least privilege
- ✅ sudo
- ✅ Why production applications should not run as root
- ✅ SSH hardening
- ✅ ss for exposed and listening services
- ✅ ufw firewall fundamentals
- ✅ Restricting Node.js and MongoDB exposure
- ✅ Secure file permissions
- ✅ SSH configuration validation
- ✅ Production MERN server hardening
- ✅ Security-focused DevOps interview reasoning

The security mindset to remember is:

```text
Inventory
   ↓
Minimize
   ↓
Restrict
   ↓
Authenticate
   ↓
Authorize
   ↓
Monitor
   ↓
Patch
   ↓
Review
```
