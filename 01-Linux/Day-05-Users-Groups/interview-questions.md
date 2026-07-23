# 💼 Interview Questions (Users & Groups) — with Detailed Answers & Examples

## Beginner

### 1. What is the difference between the root user and a normal user?
**Answer:**
- **Root (UID 0):** Has full, unrestricted access to the entire system. Can read/write/execute any file, install software, create/delete users, and stop any process.
- **Normal user:** Has restricted access. Can only modify files in their own home directory and needs `sudo` for administrative tasks.

**Example:**
```bash
# Normal user cannot write to system directories
touch /etc/test.conf
# Output: touch: cannot touch '/etc/test.conf': Permission denied

# Normal user can write to their home directory
touch ~/test.txt  # ✅ Works

# Root can write anywhere
sudo touch /etc/test.conf  # ✅ Works (with sudo)
```

---

### 2. What does `whoami` display?
**Answer:** `whoami` displays the **username** of the current user logged into the terminal session.

**Example:**
```bash
whoami
# Output: ubuntu
```

---

### 3. What is a Linux group?
**Answer:** A group is a **collection of user accounts**. Groups simplify permission management by allowing you to grant permissions to multiple users at once.

**Example:**
```
# Instead of granting access to each user individually:
# (Bad) chmod 755 alice, chmod 755 bob, chmod 755 charlie

# Create a group and add users:
sudo groupadd developers
sudo usermod -aG developers alice
sudo usermod -aG developers bob
sudo usermod -aG developers charlie

# Grant access once — all members inherit it
chgrp -R developers /shared/project
chmod -R 775 /shared/project
```

---

### 4. What does `sudo` do?
**Answer:** `sudo` (superuser do) allows a permitted user to execute a command as the **root user** (or another user) temporarily, without needing to log in as root.

**Examples:**
```bash
sudo apt update        # Install system updates (needs root)
sudo systemctl restart nginx  # Restart a service (needs root)
sudo -u bob whoami     # Run command as user bob
```

---

### 5. How do you create a new user?
**Answer:** Use `sudo useradd -m username` to create a user with a home directory, then `sudo passwd username` to set a password.

**Example:**
```bash
# Create user with home directory
sudo useradd -m john

# Set password (you'll be prompted)
sudo passwd john

# Verify the user was created
id john
# Output: uid=1002(john) gid=1002(john) groups=1002(john)

# Verify the home directory was created
ls -la /home/john
```

---

## Intermediate

### 6. Explain primary vs. secondary groups.
**Answer:**
- **Primary group:** The group assigned to files the user creates. Each user has exactly one primary group (usually same as username).
- **Secondary groups:** Additional groups the user belongs to for shared access. A user can belong to multiple secondary groups.

**Example:**
```bash
# View user's groups
id ubuntu
# Output: uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),27(sudo),999(docker)

# Primary group: ubuntu (gid=1000)
# Secondary groups: sudo, docker

# When ubuntu creates a file, it belongs to the primary group:
touch test.txt
ls -l test.txt
# Output: -rw-rw-r-- 1 ubuntu ubuntu 0 test.txt
#                                          ^ ubuntu (primary group)
```

---

### 7. Why should production services avoid running as root?
**Answer:** Running services as root is dangerous because:

- **Full system access:** If the application is compromised, the attacker gains complete control.
- **Accidental damage:** A bug in your code could delete system files.
- **No audit trail:** Hard to distinguish which service performed an action.

**Example — Compromised Node.js as root:**
```javascript
// Malicious code in a compromised npm package
const { exec } = require('child_process');
exec('rm -rf /'); // Would delete everything if running as root!
```

**Secure approach — Dedicated user:**
```bash
# Create a dedicated service user
sudo useradd -r -s /bin/false nodeapp

# Run the app as this user
sudo -u nodeapp node app.js
```

---

### 8. How do you add an existing user to the docker group?
**Answer:**
```bash
sudo usermod -aG docker username
```

**Important:** You must log out and back in for the group change to take effect, or run `newgrp docker` in the current session.

**Example:**
```bash
# Add user ubuntu to docker group
sudo usermod -aG docker ubuntu

# Verify (may need to re-login)
groups ubuntu
# Output: ubuntu sudo docker
```

---

### 9. What information does the `id` command provide?
**Answer:** `id` displays the **user ID (UID)**, **primary group ID (GID)**, and **all group memberships** for the current or specified user.

**Example:**
```bash
id ubuntu
# Output: uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),27(sudo),999(docker)

# Breakdown:
# uid=1000(ubuntu)  → User ID and username
# gid=1000(ubuntu)  → Primary group ID and name
# groups=...        → All group memberships (primary + secondary)
```

---

### 10. Why are service accounts important?
**Answer:** Service accounts are dedicated user accounts created specifically to run a particular application or service. They are important because:

- **Least privilege:** Each service gets only the permissions it needs.
- **Isolation:** If one service is compromised, others remain safe.
- **Auditability:** Clear which process owns which files/resources.
- **No interactive login:** Service accounts typically have `/sbin/nologin` or `/bin/false` as shell, preventing direct login.

**Example — Common service accounts:**
```bash
# View all service accounts (UID < 1000 typically)
grep -E '^[a-z]+:' /etc/passwd | head -10

# Output examples:
# www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
# mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/false
# jenkins:x:1001:1001::/var/lib/jenkins:/bin/bash
```

---

## Advanced

### 11. Design a secure user and group strategy for a production MERN application.
**Answer:**

**User layout:**
```
root
├── ubuntu         (SSH admin — can sudo)
├── deploy         (CI/CD pipeline — limited sudo)
├── nodeapp        (Node.js backend — no shell)
├── nginx          (Web server — no shell)
└── mongodb        (Database — no shell)
```

**Groups:**
```
admins     → ubuntu
deployers  → deploy, jenkins
developers → alice, bob, charlie
www-data   → nginx, nodeapp
```

**Implementation:**
```bash
# Create service users
sudo useradd -r -s /bin/false nodeapp
sudo useradd -r -s /bin/false nginx

# Create groups
sudo groupadd deployers
sudo groupadd developers

# Add users
sudo usermod -aG deployers deploy
sudo usermod -aG developers alice
sudo usermod -aG developers bob

# Set ownership
sudo chown -R nodeapp:www-data /var/www/mern-app
sudo chown -R mongodb:mongodb /var/lib/mongodb

# Set permissions
sudo chmod 755 /var/www/mern-app
sudo chmod 644 /var/www/mern-app/backend/*.js
sudo chmod 600 /var/www/mern-app/backend/.env
```

---

### 12. Explain the principle of least privilege with practical examples.
**Answer:** The **principle of least privilege** states that every user, process, or system should have only the **minimum permissions** necessary to perform its function.

**Practical examples:**

| Scenario | ❌ Bad Approach | ✅ Least Privilege |
|----------|----------------|-------------------|
| Web server | Run as root | Run as `www-data` user |
| Deployment | `chmod 777 /var/www` | `chmod 755` with group ownership |
| CI/CD pipeline | Full sudo access | Limited sudo for specific commands |
| Database | Root DB user | Application-specific DB user |
| SSH access | Share root password | Each admin has own key + sudo |

**Example — Limited sudo:**
```bash
# Give deploy user sudo access only for restarts
# In /etc/sudoers.d/deployer:
deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart backend
deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
```
This lets the deploy user restart services but nothing else.

---

### 13. How would you audit user accounts on a Linux server?
**Answer:**

**Step 1 — List all human users:**
```bash
# Find users with UID >= 1000 (typically human users)
awk -F: '$3 >= 1000 {print $1, $3}' /etc/passwd
```

**Step 2 — Check who has sudo access:**
```bash
# List all sudoers
sudo cat /etc/sudoers
# And files in sudoers.d
sudo ls -la /etc/sudoers.d/
```

**Step 3 — Check recently logged-in users:**
```bash
last -10
```

**Step 4 — Check currently active users:**
```bash
who
w
```

**Step 5 — Check for users with empty passwords or no shell:**
```bash
# Users with no password set
sudo awk -F: '($2 == "" || $2 == "!") {print $1}' /etc/shadow

# Users with login shells (not service accounts)
awk -F: '{print $1, $7}' /etc/passwd | grep -v nologin$ | grep -v /bin/false
```

**Step 6 — Automated audit script:**
```bash
#!/bin/bash
# user-audit.sh
echo "=== User Audit Report ==="
echo "Date: $(date)"
echo ""
echo "=== Human Users (UID >= 1000) ==="
awk -F: '$3 >= 1000 {print $1 " (UID: " $3 ")"}' /etc/passwd
echo ""
echo "=== Users with Sudo Access ==="
grep -r "ALL" /etc/sudoers.d/ 2>/dev/null || echo "(Checking /etc/sudoers)"
echo ""
echo "=== Currently Logged In ==="
who
```

---

### 14. How can incorrect group memberships become a security risk?
**Answer:** Incorrect group memberships can lead to both **over-permissioning** and **under-permissioning**:

**Over-permissioning risks:**

| Scenario | Risk |
|----------|------|
| User added to `docker` group unnecessarily | Can run containers as root, potentially escaping to host |
| User in `sudo` group without need | Can execute any command as root |
| User in `admin` group | Can read all system logs and configs |

**Example — Docker privilege escalation:**
```bash
# If a developer is in docker group unnecessarily:
docker run -v /:/host -it ubuntu bash
# Now they have root access to the entire host filesystem!
```

**Under-permissioning risks:**
- CI/CD pipeline can't deploy because the user isn't in the right group.
- Developers can't access shared logs.
- Application can't write to uploads directory.

**Best practices:**
- Review group memberships quarterly.
- Remove users from groups when they change roles.
- Use `groups username` to audit.

---

### 15. Why do CI/CD tools often use dedicated service accounts?
**Answer:** CI/CD tools (Jenkins, GitLab Runner, GitHub Actions) use dedicated service accounts for:

**1. Security isolation:**
```bash
# Jenkins runs as 'jenkins' user, not root
sudo -u jenkins whoami
# Output: jenkins

# If a build script is compromised, it can't modify system files
rm -rf /var/log     # ❌ Permission denied (runs as jenkins)
```

**2. Audit trail:**
```bash
# Every action is attributed to the service account
# In system logs:
Mar 15 02:30:15 server sudo: jenkins : TTY=unknown ; USER=root ; COMMAND=/usr/bin/systemctl restart backend
```

**3. Limited permissions:**
```bash
# Give the CI user ONLY the permissions it needs
# /etc/sudoers.d/jenkins:
jenkins ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart *
jenkins ALL=(ALL) NOPASSWD: /usr/bin/systemctl reload nginx
```

**4. SSH key management:**
```bash
# CI uses its own SSH key (not a personal key)
ls -la ~jenkins/.ssh/
# -rw------- 1 jenkins jenkins 1675 id_rsa   (600 — secure)
```

**Example — Jenkins service account setup:**
```bash
# Create dedicated user
sudo useradd -r -m -d /var/lib/jenkins -s /bin/bash jenkins

# Set up SSH for deployments
sudo -u jenkins ssh-keygen -t rsa -b 4096 -f ~jenkins/.ssh/id_rsa -N ""
sudo chmod 600 ~jenkins/.ssh/id_rsa

# Limit sudo access
echo "jenkins ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart backend" | sudo tee /etc/sudoers.d/jenkins
