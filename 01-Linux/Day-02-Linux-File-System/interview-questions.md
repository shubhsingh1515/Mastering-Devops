# 💼 Interview Questions (Linux File System) — with Detailed Answers & Examples

## Beginner

### 1. What is the root directory?
**Answer:** The root directory is the top-most directory in the Linux filesystem, represented by `/`. Everything in Linux starts from `/`.

**Example:** Unlike Windows which has drive letters like `C:` or `D:`, Linux has a single root `/` from which all other directories branch.

---

### 2. What is stored in `/home`?
**Answer:** `/home` stores personal files and directories for normal (non-administrator) users.

**Example:** 
- `/home/alice`
- `/home/bob`
- `/home/ubuntu`

---

### 3. What is the purpose of `/etc`?
**Answer:** `/etc` stores system configuration files.

**Example:**
- `/etc/hosts` — maps hostnames to IP addresses
- `/etc/passwd` — user account information
- `/etc/ssh/sshd_config` — SSH server configuration

---

### 4. Where are system logs commonly stored?
**Answer:** System logs are commonly stored in `/var/log`.

**Example:** `/var/log/syslog`, `/var/log/auth.log`

---

### 5. What command creates a symbolic link?
**Answer:** `ln -s`

**Example:**
```bash
ln -s original.txt shortcut.txt
```
This creates a symbolic link named `shortcut.txt` pointing to `original.txt`.

---

## Intermediate

### 6. Explain the difference between `/root` and `/home`.
**Answer:**
- `/root` is the home directory for the **root user** (administrator).
- `/home` contains home directories for **regular users**.

Both serve the same purpose (personal files, configs) but for different user types.

**Example:**
- `/root` → for the `root` user
- `/home/ubuntu` → for the `ubuntu` user

---

### 7. Why is `/var` important for DevOps?
**Answer:** `/var` stores **variable data** that changes frequently. For DevOps, the most important subdirectory is `/var/log`, which contains logs for system services and applications.

**DevOps scenario:** When troubleshooting an application crash, the first place to check is `/var/log`.

**Example:**
```bash
cd /var/log
ls -l
cat syslog
```

---

### 8. What are symbolic links used for?
**Answer:** Symbolic links (soft links) are used to create shortcuts or aliases to files or directories.

**Common uses:**
- Pointing to the latest version of a deployment (e.g., `current -> release-v2.3`)
- Creating convenient access paths to deeply nested files
- Sharing files across directories without duplication

**Example:**
```bash
ln -s /home/ubuntu/backend/releases/v2.3 /home/ubuntu/backend/current
```

---

### 9. Why does Linux treat many resources as files?
**Answer:** Linux follows the philosophy that "everything is a file." This provides a **consistent interface** for interacting with diverse resources (hardware, processes, devices) using the same system calls and tools (open, read, write, close).

**Example:** Reading CPU information:
```bash
cat /proc/cpuinfo
```
Even CPU data is accessible through a file-like interface.

---

### 10. When would you inspect `/proc`?
**Answer:** You would inspect `/proc` to get real-time information about running processes and kernel parameters.

**Examples:**
- `cat /proc/cpuinfo` — CPU details
- `cat /proc/meminfo` — memory usage
- `cat /proc/uptime` — system uptime

---

## Advanced

### 11. Compare hard links and symbolic links in terms of inodes and filesystem limitations.
**Answer:**
- **Hard links:** Both filenames share the same inode (same data on disk). Hard links cannot cross filesystem boundaries and cannot link to directories.
- **Symbolic links:** They have their own inode and store the path to the target. They can cross filesystems and point to directories. If the target is deleted, the symlink becomes broken.

**Example:**
```bash
ln fileA hardlink_to_A    # Both share inode 1234
ln -s fileA symlink_to_A  # New inode pointing to "fileA"
```

---

### 12. Why should applications avoid relying on `/tmp` for persistent data?
**Answer:** 
- `/tmp` is regularly cleaned by the system (e.g., on reboot).
- Files in `/tmp` can be deleted without warning when disk space is low.
- It is world-writable, posing security risks for sensitive data.

**Alternative:** Use `/var/tmp` for persistent temporary data, or a dedicated application directory under `/var`.

---

### 13. How does understanding the FHS help in containerized environments?
**Answer:** The FHS provides a **standardized directory layout**. When building Docker images, you know:
- Where to put application code (`/opt` or `/app`)
- Where configs should go (`/etc`)
- Where logs will be written (`/var/log`)
- Where temporary data belongs (`/tmp`)

**Example Dockerfile structure:**
```dockerfile
FROM ubuntu:22.04
COPY app.jar /opt/myapp/app.jar
COPY config.yml /etc/myapp/config.yml
```

---

### 14. Describe a troubleshooting workflow using `/etc`, `/var/log`, and `/proc`.
**Answer:**
1. **Check logs** in `/var/log` to identify error messages.
2. **Verify configuration** in `/etc` for the relevant service.
3. **Inspect `/proc`** to check resource usage (CPU, memory, processes).
4. **Fix the issue** (edit config, restart service, clear temp files).
5. **Verify** by checking logs again.

**Example:** Nginx crash:
```bash
tail -100 /var/log/nginx/error.log
vim /etc/nginx/nginx.conf
cat /proc/meminfo
systemctl restart nginx
```

---

### 15. How would you organize a production deployment directory for a Node.js application?
**Answer:** A common structure:
```
/opt/myapp/
├── current -> releases/v2.3.0   # Symlink to active release
├── releases/
│   ├── v2.2.0/
│   ├── v2.3.0/
│       ├── app.js
│       ├── package.json
│       ├── node_modules/
│       └── config/
├── logs/                         # Application logs
├── shared/                       # Files shared across releases
├── .env                          # Environment config
└── scripts/
    ├── deploy.sh
    └── rollback.sh
```

**Why this layout:** It enables zero-downtime deployments by switching the symlink and reloading the process.

