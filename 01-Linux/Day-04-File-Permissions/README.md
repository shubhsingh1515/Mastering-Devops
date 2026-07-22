# Day-04: Linux File Permissions & Ownership

## 🎯 Learning Objectives

By the end of today, you'll be able to:

- Understand Linux permission bits (r, w, x)
- Read permission strings from `ls -l`
- Modify permissions using `chmod`
- Change file ownership using `chown`
- Understand users and groups
- Learn numeric (octal) permissions
- Troubleshoot common permission issues
- Apply secure permissions to MERN deployments

---

## 🤔 Why This Topic Is Important

Imagine this scenario:

You deploy a Node.js backend.

```bash
systemctl start backend
```

It immediately fails. **Log:**

```
Permission denied
```

Or your deployment pipeline reports:

```
cannot execute file
```

Or SSH refuses your private key because:

```
Permissions 0644 for 'id_rsa' are too open.
```

Understanding permissions is essential for diagnosing and preventing these issues.

---

## 📖 Theory Explained in Simple Language

### Every File Has Three Things

Each file has:
- An **owner**
- A **group**
- **Permissions**

View them with:

```bash
ls -l
```

**Example:**
```
-rwxr-xr-- 1 ubuntu developers 2450 app.sh
```

Let's break it down.

### File Type

First character:

| Symbol | Type |
|--------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |

### Permission Structure

```
-rwxr-xr--
```

Break it into groups:

```
- rwx r-x r--
  │    │   └── Others
  │    └── Group
  └── Owner
```

### Permission Meanings

| Permission | Value | For Files | For Directories |
|-----------|-------|-----------|-----------------|
| Read (r) | 4 | View file contents (`cat`) | List files (`ls`) |
| Write (w) | 2 | Modify file (`echo >>`) | Create/delete files (`touch`, `rm`) |
| Execute (x) | 1 | Run as program (`./script.sh`) | Enter directory (`cd`) |

### Understanding Ownership

```
-rwxr-x--- ubuntu developers deploy.sh
```

- **Owner:** `ubuntu`
- **Group:** `developers`
- **Everyone else:** `others`

### Numeric Permissions (Octal)

Each permission has a value:

| Permission | Value |
|------------|-------|
| Read (r) | 4 |
| Write (w) | 2 |
| Execute (x) | 1 |

**Add them together:**

| Permission | Number | Meaning |
|------------|--------|---------|
| `rwx` | 7 | Read, write, execute |
| `rw-` | 6 | Read, write |
| `r-x` | 5 | Read, execute |
| `r--` | 4 | Read only |
| `---` | 0 | No permissions |

**Common combinations:**

| Octal | Owner | Group | Others | Use Case |
|-------|-------|-------|--------|----------|
| `755` | rwx | r-x | r-x | Executable scripts, directories |
| `644` | rw- | r-- | r-- | Configuration files, source code |
| `600` | rw- | --- | --- | Private keys, `.env` secrets |
| `700` | rwx | --- | --- | Private scripts |
| `775` | rwx | rwx | r-x | Shared group directories |
| `777` | rwx | rwx | rwx | ⚠️ Insecure — avoid |

### Using `chmod`

**Symbolic mode:**
```bash
chmod +x deploy.sh     # Add execute permission
chmod -w deploy.sh     # Remove write permission
chmod u+x deploy.sh    # Add execute for owner (u=user)
chmod g+w deploy.sh    # Add write for group
chmod o-r deploy.sh    # Remove read for others
```

**Numeric mode (octal):**
```bash
chmod 755 deploy.sh    # rwxr-xr-x
chmod 644 app.js       # rw-r--r--
chmod 600 .env         # rw-------
chmod 700 private.key  # rwx------
```

### Using `chown`

**Change owner:**
```bash
sudo chown ubuntu app.js
```

**Change owner and group:**
```bash
sudo chown ubuntu:developers app.js
```

**Change group only (`chgrp`):**
```bash
sudo chgrp developers app.js
```

### Directories Have Permissions Too

For directories:
- **r** → List files (`ls`)
- **w** → Create/delete files (`touch`, `rm`)
- **x** → Enter the directory (`cd`)

> Without **execute** permission on a directory, you cannot access it — even if you know the filename.

---

## 🌍 Real-World Example: MERN Deployment

```
/var/www/mern-app/
├── frontend/
├── backend/
│   ├── app.js          (644)
│   ├── .env            (600)
│   └── deploy.sh       (755)
└── uploads/            (775)
```

**Recommended permissions:**

| Path | Owner | Permissions | Reason |
|------|-------|-------------|--------|
| `/var/www/mern-app/` | www-data | 755 | Web server needs access |
| `backend/` | www-data | 755 | Traversable by server |
| `frontend/` | www-data | 755 | Static files served by web server |
| `uploads/` | www-data:developers | 775 | Shared write access |
| `.env` | www-data | 600 | Secrets — prevent other users reading |
| `app.js` | www-data | 644 | Readable by server, writable by owner |
| `deploy.sh` | ubuntu | 755 | Executable by deployment user |

---

## 🏗️ Architecture Diagram (ASCII)

```
                 app.js
                     │
      ┌──────────────┼──────────────┐
      │              │              │
   Owner          Group          Others
    rwx            r-x             r--
     7              5               4

           chmod 754 app.js
```

---

## 🛠️ Step-by-Step Practical Implementation

**Step 1** — Create a workspace:
```bash
mkdir -p ~/devops-course/day4
cd ~/devops-course/day4
```

**Step 2** — Create a script:
```bash
touch deploy.sh
```

**Step 3** — Add content:
```bash
echo '#!/bin/bash' > deploy.sh
echo 'echo "Deployment Started"' >> deploy.sh
```

**Step 4** — View permissions:
```bash
ls -l
```
Output: `-rw-r--r-- 1 ubuntu ubuntu 50 deploy.sh`

**Step 5** — Try running (will fail):
```bash
./deploy.sh
```
Expected: `Permission denied`

**Step 6** — Grant execute permission:
```bash
chmod +x deploy.sh
```

**Step 7** — Run again:
```bash
./deploy.sh
```
Expected output: `Deployment Started`

**Step 8** — Display updated permissions:
```bash
ls -l deploy.sh
```
Output: `-rwxr-xr-x 1 ubuntu ubuntu 50 deploy.sh`

**Step 9** — Practice numeric permissions:
```bash
chmod 644 deploy.sh    # rw-r--r--
chmod 755 deploy.sh    # rwxr-xr-x
chmod 600 deploy.sh    # rw-------
```

---

## ✅ Best Practices

- Use the **principle of least privilege** — grant only the permissions required.
- Store secrets (`.env`, SSH private keys) with **600** permissions.
- Use **755** for executable scripts and directories that need to be traversed.
- Avoid using **777**; it grants full access to everyone and is rarely appropriate.
- Verify permissions after deployments with `ls -l`.
- Use dedicated service accounts instead of running everything as root.
- Set ownership so the web server user can read files but not modify source code.

---

## ❌ Common Mistakes

- Setting **777** on application directories to "fix" permission problems.
- Forgetting to make deployment scripts executable.
- Running everything as **root** instead of using appropriate service accounts.
- Leaving private SSH keys **world-readable**.
- Confusing **file permissions** with **directory permissions**.
- Using `chmod 777` thinking it's a quick fix — it's a security vulnerability.

---

## 📚 Recommended Documentation

- `man chmod`
- `man chown`
- `man chgrp`
- `man ls`