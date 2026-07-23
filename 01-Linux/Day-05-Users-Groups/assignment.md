# 👨‍💻 Coding / Scripting Exercise

## 13. 👨‍💻 Coding / Scripting Exercise

Write a Bash script named `user_audit.sh` that prints:

1. Current user (`whoami`)
2. User ID and groups (`id`)
3. Logged-in users (`who`)
4. Current user's group memberships (`groups`)

**Example script (`user_audit.sh`):**

```bash
#!/bin/bash
echo "Current User:"
whoami

echo

echo "User Details:"
id

echo

echo "Logged-in Users:"
who

echo

echo "Groups:"
groups
```

**Make it executable and run:**

```bash
chmod +x user_audit.sh
./user_audit.sh
```

**Expected output (example):**

```
Current User:
ubuntu

User Details:
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),27(sudo)

Logged-in Users:
ubuntu   pts/0        2025-01-15 10:30 (192.168.1.100)

Groups:
ubuntu sudo docker
```

---

# 🔬 Hands-on Lab

## 14. 🧪 Hands-on Lab
### Lab: Simulate a Multi-User Development Server

**Goal:** Practice basic user and group administration.

**Instructions:**

**Step 1 — Create a group named `devteam`:**
```bash
sudo groupadd devteam
```

**Step 2 — Create two users: `alice` and `bob`:**
```bash
sudo useradd -m alice
sudo passwd alice
# Enter a password when prompted

sudo useradd -m bob
sudo passwd bob
# Enter a password when prompted
```

**Step 3 — Add both users to the `devteam` group:**
```bash
sudo usermod -aG devteam alice
sudo usermod -aG devteam bob
```

**Step 4 — Verify memberships:**

```bash
id alice
```
Expected:
```
uid=1001(alice) gid=1001(alice) groups=1001(alice),1004(devteam)
```

```bash
id bob
```
Expected:
```
uid=1002(bob) gid=1002(bob) groups=1002(bob),1004(devteam)
```

```bash
groups alice
```
Expected: `alice : alice devteam`

```bash
groups bob
```
Expected: `bob : bob devteam`

**Step 5 — Verify the group exists:**
```bash
grep devteam /etc/group
```
Expected: `devteam:x:1004:alice,bob`

**Step 6 — Create a shared directory for the team:**
```bash
sudo mkdir /shared-project
sudo chgrp devteam /shared-project
sudo chmod 775 /shared-project
ls -ld /shared-project
```
Expected: `drwxrwxr-x 2 root devteam 4096 /shared-project`

**Step 7 — Cleanup (optional — run after practice):**
```bash
sudo userdel -r alice
sudo userdel -r bob
sudo groupdel devteam
sudo rm -rf /shared-project
```

> ⚠️ If you're using a production or shared system, perform this lab in a virtual machine or disposable environment.

---

# 📋 Mini Assignment

## 15. 📋 Mini Assignment
### MERN Deployment User Strategy

You are the DevOps engineer for a startup deploying a MERN application.

Design a secure user strategy that answers:

1. **Which user should log in via SSH?**
2. **Which user should run the Node.js backend?**
3. **Which user should own Nginx?**
4. **Should MongoDB run as root?**
5. **Which users require sudo privileges, and why?**

Create a simple diagram illustrating your proposed user layout.

---

### ✅ Sample Solution: MERN Deployment User Strategy

#### 1. Which user should log in via SSH?

**Answer:** A dedicated admin user (e.g., `ubuntu` or `admin`) should be the only SSH login user.

```bash
# Create the admin user
sudo useradd -m -s /bin/bash admin
sudo passwd admin

# Add to sudo group for administrative tasks
sudo usermod -aG sudo admin

# Copy SSH public key for key-based authentication
sudo mkdir -p /home/admin/.ssh
sudo cp ~/.ssh/authorized_keys /home/admin/.ssh/
sudo chown -R admin:admin /home/admin/.ssh
sudo chmod 700 /home/admin/.ssh
sudo chmod 600 /home/admin/.ssh/authorized_keys
```

**Why:** Using a single non-root user for SSH access:
- Provides accountability (all actions logged).
- Root login is disabled (security best practice).
- Passwordless sudo can be configured for convenience.

#### 2. Which user should run the Node.js backend?

**Answer:** A dedicated service account called `nodeapp`.

```bash
# Create service account (no login shell)
sudo useradd -r -s /bin/false nodeapp

# Create home directory for the app
sudo mkdir -p /var/www/mern-app/backend
sudo chown -R nodeapp:nodeapp /var/www/mern-app/backend

# Application files
sudo find /var/www/mern-app/backend -type f -exec chmod 644 {} \;
sudo find /var/www/mern-app/backend -type d -exec chmod 755 {} \;
sudo chmod 600 /var/www/mern-app/backend/.env

# Run the Node.js process as this user
sudo -u nodeapp node /var/www/mern-app/backend/app.js
```

**Why:**
- If the Node.js app is compromised, the attacker only has `nodeapp` permissions.
- Cannot modify system files or read other services' data.
- Service account cannot log in interactively (`/bin/false`).

#### 3. Which user should own Nginx?

**Answer:** The built-in `www-data` user (or `nginx` user on some distributions).

```bash
# Verify www-data exists
id www-data
# Output: uid=33(www-data) gid=33(www-data) groups=33(www-data)

# Set ownership of web root
sudo chown -R www-data:www-data /var/www/mern-app/frontend

# Nginx worker processes already run as www-data
sudo grep "^user" /etc/nginx/nginx.conf
# Output: user www-data;
```

**Why:**
- `www-data` already has minimal permissions by default.
- Nginx worker processes run as `www-data`, not root.
- If Nginx is compromised, the attacker only gets `www-data` access.

#### 4. Should MongoDB run as root?

**Answer:** **No.** MongoDB should run as its own dedicated user (`mongodb`).

```bash
# Verify MongoDB service user
id mongodb
# Output: uid=111(mongodb) gid=115(mongodb) groups=115(mongodb)

# MongoDB data directory ownership
ls -la /var/lib/mongodb
# Expected: drwxr-xr-x 4 mongodb mongodb 4096 /var/lib/mongodb

# MongoDB log directory ownership
ls -la /var/log/mongodb
# Expected: drwxr-xr-x 2 mongodb mongodb 4096 /var/log/mongodb
```

**Why:**
- MongoDB's package installer automatically creates the `mongodb` user.
- Running MongoDB as root violates the principle of least privilege.
- If MongoDB has a vulnerability, the attacker only gets `mongodb` permissions.

#### 5. Which users require sudo privileges, and why?

| User | Sudo Access | Reason |
|------|-------------|--------|
| `admin` (or `ubuntu`) | Full sudo (`sudo ALL`) | Needs to install software, manage services, configure the system |
| `deploy` (CI/CD) | Limited sudo | Needs only specific commands for deployment |

**Limited sudo for CI/CD (`/etc/sudoers.d/deployer`):**
```
# Allow restarting backend and nginx
deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart backend
deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

# Allow copying new build artifacts
deploy ALL=(ALL) NOPASSWD: /usr/bin/cp -r /tmp/build /var/www/mern-app/
```

**Why limited sudo:**
- CI/CD pipeline doesn't need full root access.
- If the CI system is compromised, the attacker can only restart services, not take over the server.
- Follows the principle of least privilege.

---

### User Layout Diagram

```
                    Production Server
                    ─────────────────
                           │
                    ┌──────┴──────┐
                    │             │
                SSH Login      System Services
                    │             │
                 admin        ┌───┴──────────────┐
                    │         │       │          │
               (Full sudo)  nodeapp  www-data  mongodb
                              │        │          │
                         Node.js    Nginx     MongoDB
                         Backend    Server    Database

        Users without sudo:
        ───────────────────
        alice (developer)
        bob (developer)
            └── Both in `developers` group for shared access
```

### Permission Summary

| Component | User | Group | Permissions |
|-----------|------|-------|-------------|
| `/var/www/mern-app/backend` | nodeapp | www-data | 755 (dirs), 644 (files), 600 (.env) |
| `/var/www/mern-app/frontend` | www-data | www-data | 755 (dirs), 644 (files) |
| `/var/log/mern/` | nodeapp | adm | 755 |
| `/etc/nginx/` | root | root | 644 (configs), root-owned for security |
| `/etc/mongod.conf` | root | root | 644 |
