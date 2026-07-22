# 👨‍💻 Coding / Scripting Exercise

## 13. 👨‍💻 Coding / Scripting Exercise

Create the following structure:

```
secure-app/
├── deploy.sh
├── .env
├── logs/
└── uploads/
```

**Tasks:**

1. Make `deploy.sh` executable.
2. Set `.env` to `600`.
3. Set `logs/` to `755`.
4. Verify permissions using `ls -l` and `ls -ld`.

**Expected commands:**

```bash
# Create the structure
mkdir -p secure-app/logs secure-app/uploads
touch secure-app/deploy.sh secure-app/.env

# Add sample content
echo '#!/bin/bash' > secure-app/deploy.sh
echo 'echo "Deployment Started"' >> secure-app/deploy.sh
echo "DATABASE_URL=postgres://localhost:5432/mydb" > secure-app/.env

# Make deploy.sh executable
chmod +x secure-app/deploy.sh

# Secure .env to 600 (owner read/write only)
chmod 600 secure-app/.env

# Set directory permissions
chmod 755 secure-app/logs
chmod 755 secure-app/uploads

# Verify permissions
ls -l secure-app/
ls -ld secure-app/logs secure-app/uploads
```

**Expected output of `ls -l`:**

```
-rwxr-xr-x 1 ubuntu ubuntu   45 deploy.sh
-rw------- 1 ubuntu ubuntu   44 .env
drwxr-xr-x 2 ubuntu ubuntu 4096 logs/
drwxr-xr-x 2 ubuntu ubuntu 4096 uploads/
```

---

# 🔬 Hands-on Lab

## 14. 🧪 Hands-on Lab
### Lab: Secure a MERN Deployment Directory

**Goal:** Practice securing a deployment directory without over-permissioning files.

### Instructions

**Step 1** — Create the MERN directory structure:

```bash
mkdir -p ~/mern-demo/backend ~/mern-demo/frontend ~/mern-demo/uploads
touch ~/mern-demo/.env
```

**Step 2** — View current permissions (before):

```bash
ls -lR ~/mern-demo
```

**Step 3** — Apply these permissions:

| Path | Permission | Command |
|------|-----------|---------|
| `backend/` | 755 | `chmod 755 ~/mern-demo/backend` |
| `frontend/` | 755 | `chmod 755 ~/mern-demo/frontend` |
| `uploads/` | 775 | `chmod 775 ~/mern-demo/uploads` |
| `.env` | 600 | `chmod 600 ~/mern-demo/.env` |

**Step 4** — Create and make a deployment script executable:

```bash
touch ~/mern-demo/deploy.sh
echo '#!/bin/bash' > ~/mern-demo/deploy.sh
echo 'echo "Deploying MERN application..."' >> ~/mern-demo/deploy.sh
chmod +x ~/mern-demo/deploy.sh
```

**Step 5** — Run the deployment script:

```bash
~/mern-demo/deploy.sh
```
Expected output: `Deploying MERN application...`

**Step 6** — Record the output of `ls -l` after changing permissions:

```bash
ls -lR ~/mern-demo
```

### Expected Final Output

```
/home/ubuntu/mern-demo/:
total 12
drwxr-xr-x 2 ubuntu ubuntu 4096 backend/
-rwxr-xr-x 1 ubuntu ubuntu 40   deploy.sh
-rw------- 1 ubuntu ubuntu 0    .env
drwxr-xr-x 2 ubuntu ubuntu 4096 frontend/
drwxrwxr-x 2 ubuntu ubuntu 4096 uploads/
```

---

# 📋 Mini Assignment

## 15. 📋 Mini Assignment
### MERN Deployment Security Guide

Write a short deployment security guide for a MERN application that answers the following questions:

---

### ✅ Sample Solution: MERN Deployment Security Guide

#### 1. Which files should have `600` permissions and why?

**Files with 600:**

- **`.env`** — Contains database URLs, API keys, JWT secrets, and other credentials. Only the application user should read these.
- **SSH private keys** (`id_rsa`, `deploy_key`) — If compromised, an attacker could access servers. SSH refuses to use keys with open permissions.
- **TLS/SSL certificates** (`server.key`) — The private key must remain secret to ensure encrypted communication.
- **Database configuration files** (`database.yml`, `config/db.js` with embedded credentials) — Contains connection strings with passwords.

**Why 600:** Only the owner (application user) can read and write. No other user on the system can access sensitive data.

```bash
chmod 600 .env
chmod 600 ~/.ssh/id_rsa
```

#### 2. Which directories should generally use `755`?

**Directories with 755:**

- **`/var/www/mern-app/`** — Root application directory; needs to be traversable by the web server.
- **`backend/`** — Contains source code; the Node.js runtime needs to read files inside.
- **`frontend/`** — Contains static assets served by Nginx; needs to be readable.
- **`logs/`** — Needs to be accessible for log rotation and reading logs.
- **`node_modules/`** — Needs to be traversable by Node.js to load dependencies.

**Why 755:** Owner can read/write/traverse (7), group and others can read/traverse (5) but cannot modify files. This prevents accidental changes while allowing the application to function.

```bash
find /var/www/mern-app -type d -exec chmod 755 {} \;
```

#### 3. Why should executable scripts have the execute bit set?

Without the execute bit (`x`), even if a file contains valid shell commands, the operating system will block it from running:

```bash
# Without execute bit
./deploy.sh
# Output: bash: ./deploy.sh: Permission denied

# After setting execute bit
chmod +x deploy.sh
./deploy.sh
# Output: Deployment started
```

**Why it matters:** Deployment scripts, CI/CD pipeline scripts, and startup scripts all need the execute bit. If you forget to set it, your deployment fails immediately.

**Best practice:** Set executable permissions explicitly:
```bash
chmod 755 deploy.sh   # rwxr-xr-x — executable but readable by all
# or
chmod 700 deploy.sh   # rwx------ — executable only by owner (more secure)
```

#### 4. Why is `777` a poor solution to permission problems?

**The "quick fix" trap:**
```
chmod 777 -R /var/www/mern-app
```

This is the most common — and most dangerous — fix for permission errors.

**Why it's bad:**

| Risk | Explanation |
|------|-------------|
| 🔓 **Any user can modify files** | A compromised low-privilege user can alter your application code |
| 🛡️ **No security boundary** | There's no difference between trusted users and attackers |
| 🐚 **If a script is compromised** | An attacker can write malicious code to any file |
| 📁 **Upload directories become weapons** | Anyone can upload executable files (e.g., PHP shells) |
| 📋 **PCI/SOC2 compliance fails** | Security audits will flag 777 as a critical finding |

**The right approach:** Identify the specific permission issue and fix it precisely:

```bash
# Problem: Can't write to logs
chmod 755 logs/                 # Fix: ensure directory is writable by owner

# Problem: Script won't run
chmod +x deploy.sh              # Fix: add execute bit

# Problem: Can't read file
chmod 644 app.js                # Fix: make readable

# Problem: Wrong owner
sudo chown -R www-data:www-data # Fix: correct ownership
```

#### 5. How would you verify permissions before going live?

**Pre-deployment checklist:**

```bash
# 1. Check overall permissions
ls -lR /var/www/mern-app

# 2. Check specific sensitive files
ls -l .env
# Expected: -rw------- (600)

# 3. Check SSH key permissions
ls -l ~/.ssh/id_rsa
# Expected: -rw------- (600)

# 4. Find any files with dangerous permissions
find /var/www/mern-app -perm /o+w -type f
# Lists files writable by others — should be empty

# 5. Find any 777 directories
find /var/www/mern-app -type d -perm 777
# Should return nothing

# 6. Check directory permissions are correct
ls -ld /var/www/mern-app/backend
# Expected: drwxr-xr-x (755)

# 7. Verify scripts are executable
ls -l deploy.sh
# Expected: -rwxr-xr-x (755)

# 8. Quick test
./deploy.sh   # Should run without permission errors
```

**Automated verification script:**
```bash
#!/bin/bash
# pre-deploy-permission-check.sh

APP_DIR="/var/www/mern-app"
ERRORS=0

echo "🔍 Checking permissions for $APP_DIR..."

# Check .env
if [ "$(stat -c %a $APP_DIR/.env)" != "600" ]; then
    echo "❌ .env permissions should be 600"
    ERRORS=$((ERRORS+1))
fi

# Find world-writable files
WORLD_WRITABLE=$(find $APP_DIR -perm /o+w -type f | wc -l)
if [ "$WORLD_WRITABLE" -gt 0 ]; then
    echo "❌ Found $WORLD_WRITABLE world-writable files"
    ERRORS=$((ERRORS+1))
fi

# Find 777 directories
BAD_DIRS=$(find $APP_DIR -type d -perm 777 | wc -l)
if [ "$BAD_DIRS" -gt 0 ]; then
    echo "❌ Found $BAD_DIRS directories with 777 permissions"
    ERRORS=$((ERRORS+1))
fi

if [ "$ERRORS" -eq 0 ]; then
    echo "✅ All permission checks passed!"
else
    echo "⚠️  Found $ERRORS permission issues to fix."
fi
```

---

**Running the security check:**
```bash
bash pre-deploy-permission-check.sh
