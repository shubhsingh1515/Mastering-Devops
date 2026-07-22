# 💼 Interview Questions (File Permissions & Ownership) — with Detailed Answers & Examples

## Beginner

### 1. What do r, w, and x represent?
**Answer:**
- **r (Read, value 4)** — Allows viewing the contents of a file or listing a directory.
- **w (Write, value 2)** — Allows modifying a file or creating/deleting files in a directory.
- **x (Execute, value 1)** — Allows running a file as a program or entering (`cd`) a directory.

**Examples:**
```bash
cat app.js       # Requires r permission
echo "hi" >> log # Requires w permission
./deploy.sh      # Requires x permission
cd /var/www      # Requires x permission on directory
```

---

### 2. What does `chmod` do?
**Answer:** `chmod` (change mode) modifies the **permissions** of a file or directory. You can use either **symbolic mode** (e.g., `+x`) or **numeric mode** (e.g., `755`).

**Examples:**
```bash
chmod +x script.sh    # Symbolic: add execute
chmod 755 script.sh   # Numeric: rwxr-xr-x
chmod 600 .env        # Numeric: rw-------
```

---

### 3. What is the difference between `chown` and `chgrp`?
**Answer:**
- `chown` changes the **owner** (and optionally the **group**) of a file.
- `chgrp` changes only the **group** of a file.

**Examples:**
```bash
sudo chown ubuntu app.js              # Change owner to ubuntu
sudo chown ubuntu:developers app.js   # Change owner to ubuntu, group to developers
sudo chgrp developers app.js          # Change group only to developers
```

---

### 4. Why does `./script.sh` fail without execute permission?
**Answer:** When you run `./script.sh`, the shell needs to **execute** the file. Without the **execute (x) permission** bit set, the operating system blocks execution, resulting in:

```
bash: ./script.sh: Permission denied
```

**Fix:**
```bash
chmod +x script.sh
./script.sh   # Now works
```

---

### 5. What does `755` mean?
**Answer:** `755` is a numeric (octal) permission that breaks down as:
- **Owner:** 7 (rwx) — full read, write, execute
- **Group:** 5 (r-x) — read and execute
- **Others:** 5 (r-x) — read and execute

**Use case:** Executable scripts and directories that need to be accessible by everyone.

**Example:**
```bash
chmod 755 deploy.sh
# Result: -rwxr-xr-x
```

---

## Intermediate

### 6. Explain the permission string `-rw-r--r--`.
**Answer:** Breaking it down:

```
- rw- r-- r--
│  │   │   └── Others: r-- (read only)
│  │   └── Group: r-- (read only)
│  └── Owner: rw- (read + write, no execute)
└── Regular file (not a directory)
```

This is **644** in octal. It means:
- **Owner** can read and write the file.
- **Group** members can only read.
- **Everyone else** can only read.

**Example:** Typical source code file permissions.

---

### 7. When would you use `600` instead of `644`?
**Answer:** Use **600** when the file contains **sensitive information** that should only be readable by the owner. Use **644** when the file needs to be publicly readable but only writable by the owner.

**Examples of 600 (sensitive):**
```bash
chmod 600 ~/.ssh/id_rsa       # SSH private key
chmod 600 .env                # Environment variables with secrets
chmod 600 config/database.yml # Database credentials
```

**Examples of 644 (non-sensitive):**
```bash
chmod 644 app.js              # Source code
chmod 644 index.html          # Static HTML
chmod 644 README.md           # Documentation
```

**Why it matters:** A world-readable private SSH key (`644`) causes SSH to refuse connection:
```
Permissions 0644 for 'id_rsa' are too open.
```

---

### 8. Why is `777` considered insecure?
**Answer:** `777` gives **full read, write, and execute** permissions to **everyone** (owner, group, and others). This means:
- Any user on the system can **modify** the file.
- Any user can **execute** it.
- Any user can **delete** it.

**Security risks:**
- A malicious user could modify your executable to include harmful code.
- If it's a configuration file, anyone can change settings.
- If it's a directory (e.g., uploads/), anyone can upload malicious files.

**Better alternatives:**
```bash
chmod 755 script.sh   # Owner can write; everyone can read/execute
chmod 775 uploads/    # Owner and group can write; others can read/execute
chmod 644 config.js   # Owner can write; everyone can read
```

---

### 9. How do directory permissions differ from file permissions?
**Answer:** Although the same r/w/x symbols are used, their meaning differs:

| Permission | On Files | On Directories |
|-----------|----------|----------------|
| **r** (read) | View file contents (`cat`) | List directory contents (`ls`) |
| **w** (write) | Modify file contents (`echo >>`) | Create/delete/rename files (`touch`, `rm`, `mv`) |
| **x** (execute) | Run file as a program (`./script.sh`) | Enter/access directory (`cd`, access files inside) |

**Critical point:** A directory needs **execute (x)** permission to be entered — having only read (r) lets you list files but not access them.

**Example:**
```bash
mkdir testdir
chmod 444 testdir          # Read only on directory
ls testdir                 # ✅ Works (lists files)
cd testdir                 # ❌ Permission denied (no x)
cat testdir/file.txt       # ❌ Permission denied (no x on dir)
```

---

### 10. How would you troubleshoot a "Permission denied" error?
**Answer:** Follow these steps:

**Step 1 — Check current permissions:**
```bash
ls -l path/to/file
```

**Step 2 — Check directory permissions (you need x to access):**
```bash
ls -ld path/to/directory
```

**Step 3 — Check ownership:**
```bash
ls -l
# Or
stat path/to/file
```

**Step 4 — Check your user/group:**
```bash
whoami
groups
```

**Step 5 — Apply appropriate fix:**
```bash
chmod +x script.sh        # Add execute if missing
chmod 600 .env            # Secure if too open
sudo chown user:group .   # Fix ownership if wrong
```

---

## Advanced

### 11. Design secure permissions for a Node.js application deployed under `/var/www`.
**Answer:** Here's a secure permission design for a MERN application:

**Directory structure:**
```
/var/www/mern-app/
├── backend/          (755, owner: www-data)
│   ├── app.js        (644)
│   ├── .env          (600)
│   └── node_modules/ (755)
├── frontend/         (755, owner: www-data)
│   ├── build/        (755)
│   └── static/       (755)
├── uploads/          (775, owner: www-data, group: developers)
└── logs/             (755, owner: www-data)
```

**Explanation:**
- **644** for source files — web server can read, only owner can write.
- **600** for `.env` — only the application user can read secrets.
- **755** for directories — web server can traverse them.
- **775** for uploads/ — shared group write access for multiple engineers.
- **Owner www-data** — the web server/Nginx user can serve files.
- **No 777 anywhere** — least privilege principle.

**Apply with:**
```bash
sudo chown -R www-data:www-data /var/www/mern-app
find /var/www/mern-app -type d -exec chmod 755 {} \;
find /var/www/mern-app -type f -exec chmod 644 {} \;
chmod 600 /var/www/mern-app/backend/.env
chmod 775 /var/www/mern-app/uploads
```

---

### 12. How would you manage permissions for shared deployment directories used by multiple engineers?
**Answer:** Use **group-based permissions**:

**Strategy:**
1. Create a dedicated group (e.g., `deployers`).
2. Add all engineers to this group.
3. Set group ownership on the deployment directory.
4. Use `775` to give the group write access.

**Implementation:**
```bash
# Create group
sudo groupadd deployers

# Add users to group
sudo usermod -aG deployers alice
sudo usermod -aG deployers bob

# Set ownership and permissions
sudo chgrp -R deployers /var/www/mern-app
sudo chmod -R 775 /var/www/mern-app

# Ensure new files inherit group
sudo chmod g+s /var/www/mern-app   # SetGID bit
```

The **SetGID bit** (`g+s`) ensures new files created in the directory inherit the group (`deployers`), so all team members can modify them.

---

### 13. Why should application services run under dedicated users rather than root?
**Answer:** Running services as **root** is dangerous because:

- **Full system access** — if the application is compromised, the attacker gains complete control.
- **Accidental damage** — a bug in your code could delete system files.
- **No audit trail** — hard to distinguish which service performed an action.

**Best practice:** Each service gets its own user with minimal permissions.

**Example — Node.js service:**
```bash
# Create dedicated user
sudo useradd -r -s /bin/false nodeapp

# Set ownership
sudo chown -R nodeapp:nodeapp /var/www/mern-app

# Configure systemd to run as this user
# In /etc/systemd/system/backend.service:
# [Service]
# User=nodeapp
# Group=nodeapp
```

---

### 14. Explain the principle of least privilege and how it applies to DevOps.
**Answer:** The **principle of least privilege** states that every user, process, or system should have only the **minimum permissions** necessary to perform its function.

**How it applies in DevOps:**

| Scenario | Common Mistake | Least Privilege Approach |
|----------|---------------|-------------------------|
| Application files | `chmod 777 /var/www` | `chmod 755` dirs, `644` files |
| CI/CD pipeline | Running as root | Dedicated deploy user |
| SSH keys | `chmod 644 id_rsa` | `chmod 600 id_rsa` |
| Database access | Global read/write | Specific table permissions |
| Container | Running as root | `USER node` in Dockerfile |

**Benefits:**
- 🔒 **Security** — limits damage from breaches
- 🐛 **Error prevention** — prevents accidental changes
- 🔍 **Auditability** — clear who did what

---

### 15. How can incorrect permissions break a CI/CD pipeline?
**Answer:** Incorrect permissions can break CI/CD pipelines in many ways:

**Common failure scenarios:**

**1. Script not executable:**
```bash
# Pipeline error
./deploy.sh: Permission denied

# Fix
chmod +x deploy.sh
```

**2. Cannot read .env (wrong ownership):**
```bash
# Pipeline error
Error: Cannot read /var/www/app/.env

# Fix
sudo chown -R pipeline-user:pipeline-group /var/www/app
```

**3. Cannot write logs:**
```bash
# Application error
Error: EACCES: permission denied, open '/var/log/app/error.log'

# Fix
chmod 755 /var/log/app
chown appuser:appuser /var/log/app
```

**4. SSH key too open:**
```bash
# Pipeline error
Permissions 0644 for 'deploy-key' are too open.

# Fix
chmod 600 deploy-key
```

**5. Cannot access directory (missing x):**
```bash
# Error
cd /var/www/app: Permission denied

# Fix
chmod 755 /var/www/app
```

**Best practices for CI/CD:**
- Always set permissions explicitly in deployment scripts.
- Verify permissions with `ls -l` before critical steps.
- Use dedicated deployment users with appropriate group memberships.
- Store secrets with `600` permissions.
- Make scripts executable (`chmod +x`) and commit them.
