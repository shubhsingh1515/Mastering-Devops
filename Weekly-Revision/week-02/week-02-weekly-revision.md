# DevOps Mentorship Program — Week 2 Sunday Revision

> **Sunday Special:** Reinforcing Days 1–7 (Linux Fundamentals through Processes)

---

## 📊 Progress Tracker
- **Phase:** Linux
- **Lessons Completed:** Day 1 (Linux Fundamentals) → Day 7 (Linux Processes)
- **Phase Progress:** 7 / 19 (Linux)
- **Next New Lesson:** Day 8 — Linux Services & systemctl

---

## 🔁 Quick Review Questions

Let me test myself without looking at my notes first.

### 1. Which directory stores configuration files?
<details>
<summary>Answer</summary>

**`/etc`** — This is where system-wide configuration files live, like `sshd_config`, `nginx.conf`, and `hosts`.

</details>

### 2. What permission should a private SSH key have?
<details>
<summary>Answer</summary>

**`600`** (rw-------). Only the owner should be able to read and write. If the permissions are too open (like 644), SSH will refuse to use the key for security reasons.

</details>

### 3. Which command shows listening ports?
<details>
<summary>Answer</summary>

**`ss -tuln`** — Shows TCP (`-t`) and UDP (`-u`) listening (`-l`) sockets with numeric (`-n`) output. The older alternative is `netstat -tuln`.

</details>

### 4. What command displays all running processes?
<details>
<summary>Answer</summary>

**`ps -ef`** or **`ps aux`** — Both give a full snapshot of all running processes across the system.

</details>

### 5. What command follows a growing log file?
<details>
<summary>Answer</summary>

**`tail -f`** — Displays the last few lines and then waits for new lines to be appended in real-time. Press `Ctrl + C` to stop.

</details>

---

## 📚 Week 1 Summary (My Personal Notes)

### Linux Fundamentals

I learned the core architecture of Linux:

```
Application
   ↓
Shell (Bash)
   ↓
Linux Kernel
   ↓
Hardware
```

Key takeaways:
- **Kernel** manages CPU, memory, storage, devices, networking
- **Shell** (Bash) interprets my commands and passes them to the kernel
- **Terminal** is where I type commands
- **Absolute paths** start from `/` (root)
- **Relative paths** start from my current directory

---

### Linux File System (FHS)

I now know the Linux directory layout like the back of my hand:

```
/
├── etc      → Configuration files
├── home     → User home directories
├── var      → Variable data (logs, caches)
├── usr      → Installed software & libraries
├── root     → Root user's home directory
├── proc     → Virtual filesystem for processes
├── tmp      → Temporary files (auto-cleared)
├── boot     → Boot files & kernel images
├── dev      → Device files
└── opt      → Optional/third-party software
```

**Crucial distinction I must remember:**
- `/` = root directory (top of the tree)
- `/root` = root user's home directory (they are NOT the same!)

I also learned:
- **Everything is a file** in Linux (text files, directories, devices, processes)
- **File types** identified by the first character in `ls -l`: `-` (file), `d` (directory), `l` (symlink)
- **Symbolic links** (soft links) are like shortcuts — if the original is deleted, the link breaks
- **Hard links** point to the same inode — deleting one doesn't remove the data until all links are gone

---

### Shell Commands (My Daily Toolbox)

These are my everyday commands that I must know by muscle memory:

| Command | What it does | Real-world use |
|---------|-------------|----------------|
| `pwd` | Print working directory | "Where am I?" |
| `ls -l` | List files with details | Check permissions, sizes |
| `cd ..` | Go up one directory | Navigate the tree |
| `cp -r` | Copy recursively | Backup directories |
| `mv` | Move or rename | Organize files |
| `rm -r` | Delete recursively | Clean up (⚠️ careful!) |
| `find . -name "*.js"` | Search for files | Find configs, logs |
| `cat` | Display entire file | Quick view of small files |
| `less` | Scroll through file | Reading large log files |
| `head -20` | First 20 lines | Check top of file |
| `tail -f` | Follow logs in real-time | **My go-to for monitoring** |
| `df -h` | Disk space in human-readable | "Is the disk full?" |
| `du -sh` | Directory size | "What's eating space?" |
| `history` | Command history | "What did I run?" |

**Pipes (`|`)** — One of my favorite Linux features:
```bash
history | tail        # Last 10 commands
ps -ef | grep node    # Find Node.js processes
ls -l | less          # Scroll through large listings
```

**Redirection:**
- `>` overwrites a file
- `>>` appends to a file

---

### File Permissions & Ownership

I now understand the permission model completely:

```
-rwxr-xr--  1  ubuntu  developers  2450  app.sh
│  │  │  │    │  │       │           │     │
│  │  │  │    │  │       │           │     └─ Filename
│  │  │  │    │  │       │           └─ File size
│  │  │  │    │  │       └─ Group
│  │  │  │    │  └─ Owner
│  │  │  │    └─ Link count
│  │  │  └─ Others permissions
│  │  └─ Group permissions
│  └─ Owner permissions
└─ File type (- = file, d = directory, l = symlink)
```

**The permission triads:**
- **Owner (user)** — the person who owns the file
- **Group** — users who belong to the file's group
- **Others** — everyone else

**Numeric permissions (octal) — I must memorize these:**

| Number | Permissions | Meaning |
|--------|-------------|---------|
| 7 | rwx | Read, write, execute |
| 6 | rw- | Read, write |
| 5 | r-x | Read, execute |
| 4 | r-- | Read only |
| 0 | --- | No permissions |

**Essential combinations for DevOps:**

| Octal | Use Case |
|-------|----------|
| **755** | Executable scripts, directories (rwxr-xr-x) |
| **644** | Source code, config files (rw-r--r--) |
| **600** | **SSH keys, .env secrets** (rw-------) |
| **700** | Private scripts (rwx------) |
| **775** | Shared group directories |
| **777** | ⚠️ **NEVER USE** — completely open |

**Key commands:**
```bash
chmod 755 script.sh     # Change permissions
chown ubuntu:dev file   # Change owner:group
chgrp developers file   # Change group only
```

---

### Users & Groups

I learned why user management is critical in production:

**Types of users:**
- **Root** (UID 0) — full control, used only for administrative tasks
- **Normal users** — limited, use `sudo` for admin actions
- **Service accounts** — dedicated users for each service (`nginx`, `mongodb`, `nodeapp`)

**Production principle — Least Privilege:**
> Each service runs under its own restricted user. If one is compromised, the attacker doesn't get the whole server.

```
root
├── nginx      (web server)
├── nodeapp    (Node.js)
├── mongodb    (database)
├── jenkins    (CI/CD)
└── ubuntu     (SSH login)
```

**Key commands I practiced:**
```bash
whoami              # Who am I?
id                  # My UID, GID, groups
sudo useradd -m user    # Create user with home dir
sudo passwd user        # Set password
sudo userdel -r user    # Delete user and home dir
sudo groupadd devs      # Create group
sudo usermod -aG devs user  # Add user to group (⚠️ never forget -a!)
groups user             # View user's groups
```

---

### Networking Commands

I now have a systematic approach to troubleshooting network issues:

```flow
Problem → Connectivity → DNS → Ports → Application
```

**My networking toolkit:**

| Command | What it tells me |
|---------|-----------------|
| `ip addr` | My IP addresses and network interfaces |
| `ip route` | Default gateway / routing table |
| `ping -c 4 google.com` | Is the internet reachable? |
| `curl -I http://localhost:3000` | Is my app responding? |
| `ss -tuln` | Which ports are listening? |
| `nslookup example.com` | DNS resolution |
| `dig example.com` | Detailed DNS info |
| `hostname` | What's this server called? |
| `wget https://...` | Download files |

**Real-world troubleshooting flow for a MERN backend:**
```bash
# 1. Is the server reachable?
ping api.company.com

# 2. DNS working?
nslookup api.company.com

# 3. HTTP response?
curl -I https://api.company.com

# 4. SSH in and check ports
ss -tuln | grep 3000

# 5. Test locally
curl http://localhost:3000
```

---

### Processes

I now understand the difference between a program and a process:
- **Program** = file on disk (`node`, `nginx`)
- **Process** = running instance of a program

**Key concepts:**
- **PID** — unique Process ID
- **Parent/Child** — processes can spawn other processes
- **Foreground** — blocks my terminal (`Ctrl + C` to stop)
- **Background** — runs independently (`command &`)

**Process management commands:**

| Command | Purpose |
|---------|---------|
| `ps -ef` | All running processes (snapshot) |
| `top` | Live monitoring (press `q` to quit) |
| `ps -ef \| grep node` | Find specific process |
| `kill <PID>` | Graceful stop (SIGTERM) |
| `kill -9 <PID>` | Force kill (SIGKILL) — last resort |
| `pkill node` | Kill by name |
| `jobs` | List background jobs |
| `fg` | Bring job to foreground |
| `uptime` | System load averages |

**SIGNALS:**
- **SIGTERM** (kill) — "Please stop gracefully"
- **SIGKILL** (kill -9) — "Stop NOW, no cleanup"

---

## 🏗️ Real-World Scenario Assessment

> **Scenario:** Production server with Nginx → Node Backend → MongoDB
> **Users report:** The website is slow.
> **As the on-call DevOps engineer, here's my investigation order:**

### Step 1 — Check if the server is reachable
```bash
ping <server-ip>
```
*Why:* First confirm the server is actually online and network connectivity is working.

### Step 2 — Verify the web service
```bash
curl -I http://localhost
```
*Why:* Check if Nginx is responding with the expected HTTP status codes.

### Step 3 — Check listening ports
```bash
ss -tuln
```
*Why:* Verify that Nginx (port 80/443), Node (port 3000), and MongoDB (port 27017) are actually listening.

### Step 4 — Inspect running processes
```bash
ps -ef | grep -E "nginx|node|mongo"
top
```
*Why:* Look for high CPU/memory usage, zombie processes, or missing processes.

### Step 5 — Review recent logs
```bash
tail -50 /var/log/nginx/access.log
tail -50 /var/log/nginx/error.log
tail -50 /var/log/application/backend.log
```
*Why:* Logs often reveal the root cause — errors, timeouts, stack traces.

### Step 6 — Check disk usage
```bash
df -h
du -sh /var/log
```
*Why:* A full disk can cause applications to fail or become extremely slow.

### Step 7 — Verify permissions
```bash
ls -l /var/www/mern-app/
ls -l /etc/nginx/
```
*Why:* If deployment files have wrong permissions, services may not be able to read them.

---

## 🧪 Practical Assessment

I'll document how I completed each task in my Linux environment.

### Task 1 — Create Directory Structure
```bash
mkdir -p week1-assessment/{backend,frontend,logs,configs,scripts}
```
This creates:
```
week1-assessment/
├── backend
├── frontend
├── logs
├── configs
└── scripts
```

### Task 2 — Create deploy.sh and Make It Executable
```bash
touch week1-assessment/scripts/deploy.sh
chmod +x week1-assessment/scripts/deploy.sh
```
Now `deploy.sh` has `755` permissions (rwxr-xr-x).

### Task 3 — Create .env with Secure Permissions
```bash
touch week1-assessment/configs/.env
chmod 600 week1-assessment/configs/.env
```
Permission `600` means only the owner can read/write — perfect for secrets.

### Task 4 — Verify Everything
```bash
ls -lR week1-assessment/
```

### Task 5 — Find Every Shell Script
```bash
find . -name "*.sh"
```
This recursively searches from the current directory for all files ending in `.sh`.

### Task 6 — Display Directory Size
```bash
du -sh week1-assessment/
```

### Task 7 — Show IP, Hostname, Running Processes
```bash
ip addr                # IP address
hostname               # Hostname
ps -ef                 # Running processes
```

---

## 🎤 Mock Interview — My Answers

### Beginner

**Q1: Difference between Linux and Ubuntu?**
> **Linux** is the kernel — the core operating system component that manages hardware. **Ubuntu** is a complete Linux distribution (distro) that packages the Linux kernel together with applications, package managers, and desktop environments to create a usable operating system.

**Q2: Difference between a file and a directory?**
> A **file** stores data (text, code, images, binaries). A **directory** is a container that holds files and other directories (like a folder). In `ls -l`, files start with `-` and directories start with `d`.

**Q3: Explain chmod.**
> `chmod` (change mode) modifies file/directory permissions. It can be used in symbolic mode (`chmod +x script.sh`) or numeric mode (`chmod 755 script.sh`). It controls read (`r`), write (`w`), and execute (`x`) permissions for the owner, group, and others.

**Q4: Purpose of /etc.**
> `/etc` stores system-wide configuration files. Examples: `/etc/ssh/sshd_config` (SSH server config), `/etc/hosts` (static hostname mapping), `/etc/nginx/nginx.conf` (Nginx config). This is the first place I look when troubleshooting configuration issues.

**Q5: Purpose of tail -f.**
> `tail -f` follows a file in real-time, displaying new lines as they're appended. I use this constantly to monitor live log files during production incidents. Press `Ctrl + C` to stop.

### Intermediate

**Q6: Difference between 644, 755, and 600?**
| Permission | Numeric | Owner | Group | Others | Use Case |
|-----------|---------|-------|-------|--------|----------|
| 644 | rw-r--r-- | Read/Write | Read | Read | Source code, configs |
| 755 | rwxr-xr-x | Full | Read/Execute | Read/Execute | Executables, directories |
| 600 | rw------- | Read/Write | Nothing | Nothing | SSH keys, `.env` secrets |

**Q7: Difference between kill and kill -9?**
> **`kill <PID>`** sends SIGTERM (signal 15) — a graceful termination request that allows the process to clean up resources, save state, and exit properly. **`kill -9 <PID>`** sends SIGKILL — an immediate, forced termination that the process cannot ignore or handle. The process dies instantly with no cleanup. I always try `kill` first and only use `kill -9` if the process doesn't respond.

**Q8: Why shouldn't applications run as root?**
> If an application running as **root** gets compromised, the attacker gains **full control** of the server — they can read any file, install malware, delete everything, and pivot to other systems. Running as a dedicated user with **least privilege** limits the blast radius. If `nodeapp` gets compromised, the attacker only gets access to what `nodeapp` owns, not the entire system.

**Q9: Difference between curl and wget?**
> Both download/transfer data, but they have different strengths. **`curl`** is better for testing APIs and HTTP responses — I use it with `-I` for headers or to check localhost endpoints. **`wget`** is better for downloading files and recursive downloads. `curl` works with more protocols and is more scriptable; `wget` is simpler for single-file downloads.

**Q10: How would you troubleshoot a Node.js application that won't start?**
> My systematic approach:
> 1. **Verify the process:** `ps -ef | grep node` — is it running?
> 2. **Check logs:** `tail -50 /var/log/app/backend.log` — any error messages?
> 3. **Confirm permissions:** `ls -l /var/www/app/.env` — does the service user have proper access?
> 4. **Check listening ports:** `ss -tuln | grep 3000` — is the port available?
> 5. **Validate configuration:** `cat /etc/app/config.json` — are database URLs correct?
> 6. **Check disk space:** `df -h` — is the disk full?

### Advanced

**Q11: Explain the Linux permission model.**
> Linux uses a **DAC (Discretionary Access Control)** model. Every file and directory has:
> - An **owner** (user)
> - A **group**
> - Three sets of permissions: for the **owner** (user), the **group**, and **others**
> 
> Each permission set has three bits: **read (4)**, **write (2)**, **execute (1)**. These combine to form octal values (0–7).
> 
> For **files**: read = view contents, write = modify, execute = run as a program.
> For **directories**: read = list contents, write = create/delete files inside, execute = enter the directory.
> 
> The first character of `ls -l` shows the file type (`-` file, `d` directory, `l` symlink). The special commands are `chmod` (change permissions), `chown` (change owner), and `chgrp` (change group).

**Q12: How would you secure a production Linux server running a MERN application?**
> Here's my security approach:
> 
> **1. Dedicated Service Users**
> - Each service gets its own user: `nginx`, `nodeapp`, `mongodb`
> - No service runs as root
> - Service accounts have no login shell
> 
> **2. Least Privilege**
> - Files: 755 for directories/executables, 644 for code, 600 for secrets
> - `.env` files have `600` permissions
> - SSH keys have `600` permissions
> 
> **3. SSH Security**
> - Disable root login (`PermitRootLogin no`)
> - Use SSH key authentication (passwordless)
> - Restrict SSH to specific users
> 
> **4. User Management**
> - Each admin has their own account (no shared logins)
> - Use `sudo` for administrative tasks
> - Remove unused accounts
> 
> **5. Log Monitoring**
> - Monitor `/var/log/` for suspicious activity
> - Use `tail -f` and centralized logging
> 
> **6. Regular Updates**
> - Keep the system patched
> - Update Node.js, Nginx, MongoDB regularly
> 
> **7. Network Security**
> - Only required ports are open (22/SSH, 80/443 for web, 27017 only from app server)
> - Use a firewall (like `ufw` or `iptables`)

---

## 📝 Comprehensive Quiz (10 Questions)

Let me test my knowledge:

**1. Which command shows your current directory?**
<details>
<summary>Answer</summary>

`pwd` (Print Working Directory)

</details>

**2. Which directory stores system logs?**
<details>
<summary>Answer</summary>

`/var/log` — Contains logs for applications, system services, and the kernel.

</details>

**3. Which command changes ownership?**
<details>
<summary>Answer</summary>

`chown` (Change Owner). Example: `sudo chown ubuntu:developers app.js`

</details>

**4. Which permission is recommended for .env files?**
<details>
<summary>Answer</summary>

**600** (rw-------). Only the owner can read and write. This prevents other users on the system from accessing environment variables containing secrets.

</details>

**5. Which command shows your IP address?**
<details>
<summary>Answer</summary>

`ip addr` (or the older `ifconfig`). Shows network interfaces and their assigned IP addresses.

</details>

**6. Which command displays running processes?**
<details>
<summary>Answer</summary>

`ps -ef` or `ps aux` — Shows a snapshot of all running processes.

</details>

**7. Which command follows a log file?**
<details>
<summary>Answer</summary>

`tail -f <filename>` — Displays new log entries in real-time.

</details>

**8. Which command lists listening ports?**
<details>
<summary>Answer</summary>

`ss -tuln` — Shows TCP and UDP listening ports numerically.

</details>

**9. Which command creates a user?**
<details>
<summary>Answer</summary>

`sudo useradd -m <username>` — Creates a user with a home directory.

</details>

**10. What is the purpose of sudo?**
<details>
<summary>Answer</summary>

`sudo` (Superuser Do) allows a permitted user to execute commands with administrative (root) privileges. It's a security mechanism that avoids logging in directly as root.

</details>

---

## 💻 Coding Exercise — system_health.sh

I created a system health check script that gives a quick overview of the server's state:

```bash
#!/bin/bash

echo "===== HOSTNAME ====="
hostname

echo
echo "===== CURRENT USER ====="
whoami

echo
echo "===== UPTIME ====="
uptime

echo
echo "===== DISK USAGE ====="
df -h

echo
echo "===== MEMORY USAGE ====="
free -h

echo
echo "===== TOP PROCESSES ====="
ps -ef --sort=-%cpu | head -15

echo
echo "===== LISTENING PORTS ====="
ss -tuln
```

**To use it:**
```bash
chmod +x system_health.sh
./system_health.sh
```

**New command learned:** `free -h` — displays RAM usage in human-readable format. I'll study system monitoring in detail later, but this is useful to start recognizing now.

---

## 🧪 Hands-on Lab — "MERN Backend Is Unreachable"

**Scenario:** My teammate says: *"The MERN backend is unreachable."*

Here's my systematic investigation using only commands I've learned:

### Step 1 — Check hostname
```bash
hostname
```
*Why:* Confirm which server I'm on (especially when managing multiple servers).

### Step 2 — Check IP address
```bash
ip addr
```
*Why:* Verify the server has a valid IP and the network interface is up.

### Step 3 — Verify Node.js process
```bash
ps -ef | grep node
```
*Why:* Check if the Node.js application is actually running. The process might have crashed.

### Step 4 — Verify listening ports
```bash
ss -tuln | grep 3000
```
*Why:* Even if the process is running, it might not be listening on the expected port (maybe it crashed and restarted on a different port).

### Step 5 — Inspect the latest logs
```bash
tail -50 /var/log/backend/app.log
```
*Why:* Logs often contain the exact error — database connection failure, invalid config, port in use, etc.

### Step 6 — Confirm deployment file permissions
```bash
ls -l /var/www/mern-app/.env
ls -l /var/www/mern-app/backend/app.js
```
*Why:* If the service user can't read the `.env` or execute the app, it will fail silently.

### Step 7 — Verify available disk space
```bash
df -h
```
*Why:* A full disk can prevent the application from writing logs, creating temporary files, or functioning at all.

---

## 📋 Mini Assignment — Linux Operations Runbook

I created this one-page quick-reference guide for myself:

---

# 🐧 My Linux Operations Runbook

## Essential Navigation Commands
| Command | Description |
|---------|-------------|
| `pwd` | Show current directory |
| `ls -la` | List all files with details |
| `cd /path` | Change directory |
| `cd ..` | Go up one level |
| `cd ~` | Go to home directory |
| `mkdir -p a/b/c` | Create nested directories |
| `find . -name "*.js"` | Search for files |
| `history` | View command history |

## Filesystem Layout
```
/etc      → Configuration files
/var/log  → Log files
/home     → User home directories
/usr      → Installed software
/tmp      → Temporary files (cleared on boot)
/proc     → Process & kernel information
```

## Permission Guidelines
| Octal | String | Use Case |
|-------|--------|----------|
| **755** | rwxr-xr-x | Directories, executable scripts |
| **644** | rw-r--r-- | Source code, config files |
| **600** | rw------- | SSH keys, .env files (SECRETS!) |
| **700** | rwx------ | Private scripts |
| **777** | rwxrwxrwx | ⛔ NEVER USE |

**Key Commands:** `chmod`, `chown`, `chgrp`

## User Management
```bash
sudo useradd -m username    # Create user
sudo passwd username         # Set password
sudo usermod -aG group user  # Add to group
sudo userdel -r username     # Delete user
sudo groupadd groupname      # Create group
groups username              # View groups
```

## Networking Commands
```bash
ip addr        # IP addresses
ping -c 4 url  # Test connectivity
curl -I url    # HTTP headers
ss -tuln       # Listening ports
nslookup url   # DNS resolution
dig url        # Detailed DNS
```

## Process Management
```bash
ps -ef              # All processes
ps -ef | grep node  # Find specific process
top                 # Live monitoring (q to quit)
kill <PID>          # Graceful stop (SIGTERM)
kill -9 <PID>       # Force stop (SIGKILL) — last resort
jobs                # Background jobs
fg                  # Bring job to foreground
uptime              # System load averages
```

## Production Troubleshooting Workflow
```
1. Hostname     → hostname
2. IP           → ip addr
3. Connectivity → ping
4. DNS          → nslookup
5. Ports        → ss -tuln
6. Processes    → ps -ef | top
7. Logs         → tail -f /var/log/app.log
8. Disk         → df -h
9. Permissions  → ls -l
```
---

*"Learning Linux is not about memorizing commands — it's about understanding how the system works so you can figure out what you need, when you need it."*


