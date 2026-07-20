# Day-02: Linux File System (Filesystem Hierarchy Standard - FHS)

## 📅 Phase 1: Linux
- **Topic:** Linux File System
- **Estimated Time:** 20–30 minutes
- **Goal:** Learn how Linux organizes files and directories so you can confidently navigate production servers, Docker containers, and cloud instances.

## 📊 Progress Tracker
- ✅ **Phase:** Linux Fundamentals
- ✅ **Completed Lessons:** Day 1 – Linux Fundamentals
- 📖 **Today's Lesson:** Day 2 – Linux File System
- 🎯 **Next Lesson:** Day 3 – Shell Commands (Deep Dive)

---

## 🔁 Quick Review (Day 1)

Try answering these before checking your notes.

1. **What command shows your current working directory?**
   ✅ `pwd`

2. **What is the difference between an absolute path and a relative path?**
   ✅ Absolute path starts from `/` (root).
   ✅ Relative path starts from your current directory.

3. **Why is Linux the preferred operating system for DevOps?**
   ✅ Stable, secure, lightweight, scriptable, and used by most cloud providers.

4. **Which command creates a directory?**
   ✅ `mkdir`

5. **What is the Linux kernel responsible for?**
   ✅ Managing hardware resources such as CPU, memory, storage, and devices.

---

## 🎯 Learning Objectives

By the end of this lesson, you'll be able to:

- Understand the Linux Filesystem Hierarchy Standard (FHS)
- Identify the purpose of important directories
- Differentiate between file types
- Understand symbolic and hard links
- Navigate a Linux server confidently
- Relate filesystem layout to Docker containers and MERN deployments

---

## 🤔 Why This Topic Is Important

Imagine you're on call.

Your Node.js application suddenly crashes.

Where do you look?

- Logs?
- Configuration?
- Installed binaries?
- Temporary files?

Without understanding the Linux filesystem, troubleshooting production systems becomes difficult.

Every DevOps engineer should instantly recognize directories like:

- `/etc`
- `/var`
- `/home`
- `/usr`
- `/tmp`

---

## 📖 Theory Explained in Simple Language

### Everything Is a File

Linux follows a simple philosophy:

> **Everything is treated as a file.**

Examples:
- Text files
- Directories
- Hard disks
- USB drives
- Network devices
- Processes (represented under `/proc`)

This consistent model makes automation easier.

### The Root Directory

Everything begins from:

```
/
```

Think of it as the root of a tree.

```
/
├── home
├── etc
├── usr
├── var
├── tmp
├── boot
└── dev
```

Unlike Windows, Linux does **not** have drive letters like `C:` or `D:`.

### Important Directories

| Directory | Purpose | Example Path |
|-----------|---------|--------------|
| `/home` | Stores personal files for normal users | `/home/ubuntu/projects/mern-app` |
| `/root` | Home directory of the root (administrator) user | `/root` |
| `/etc` | System configuration files | `/etc/hosts`, `/etc/ssh/sshd_config` |
| `/usr` | Installed software and shared libraries | `/usr/bin`, `/usr/lib` |
| `/var` | Frequently changing data (logs, caches, queues) | `/var/log` |
| `/tmp` | Temporary files (auto-cleared) | — |
| `/boot` | Files required for booting Linux | Kernel images, bootloader config |
| `/dev` | Device files | `/dev/sda`, `/dev/null` |
| `/proc` | Virtual filesystem for processes and kernel info | `/proc/cpuinfo` |
| `/opt` | Optional / third-party software | `/opt/sonarqube` |

> ⚠️ **Do not confuse `/` (root directory) with `/root` (root user's home directory). They are different.**

### File Types in Linux

Use `ls -l` — the first character indicates the file type:

| Symbol | Type |
|--------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |

**Example:** `drwxr-xr-x` — The `d` indicates a directory.

### Hard Links vs Symbolic Links

**Hard Link:**
```
fileA
  ↓
inode 1234
  ↑
fileB
```
Both names point to the same data. Deleting one name does not remove the data as long as another hard link exists.

**Symbolic Link (Soft Link):**
```
shortcut
  ↓
original file
```
Think of it like a desktop shortcut. If the original file is deleted, the symbolic link becomes broken.

Create one with:
```bash
ln -s original.txt shortcut.txt
```

---

## 🌍 Real-World Example

Suppose your MERN backend isn't starting. Where do you investigate?

```
Application Logs
     ↓
   /var/log
     ↓
System Configuration
     ↓
   /etc
     ↓
Application Files
     ↓
/home/ubuntu/backend
```

A DevOps engineer quickly moves through these locations to diagnose problems.

---

## 🏗️ Architecture Diagram (ASCII)

```
                  /
   ├──────────── home
   ├──────────── etc
   ├──────────── usr
   ├──────────── var
   ├──────────── tmp
   ├──────────── boot
   ├──────────── dev
   ├──────────── proc
   └──────────── root
```

---

## 🛠️ Step-by-Step Practical Implementation

**Step 1** — Go to the root directory:
```bash
cd /
```

**Step 2** — List all top-level directories:
```bash
ls
```

**Step 3** — Inspect the `/etc` directory:
```bash
cd /etc
ls
```

**Step 4** — View your home directory:
```bash
cd ~
pwd
```

**Step 5** — List log files:
```bash
cd /var/log
ls
```

**Step 6** — View CPU information:
```bash
cat /proc/cpuinfo
```

**Step 7** — Create a practice area:
```bash
mkdir -p ~/devops-course/day2
cd ~/devops-course/day2
```

**Step 8** — Create a file:
```bash
touch original.txt
```

**Step 9** — Create a symbolic link:
```bash
ln -s original.txt shortcut.txt
```

**Step 10** — Verify:
```bash
ls -l
```
You should see `shortcut.txt` pointing to `original.txt`.

---

## ✅ Best Practices

- Store project files under your home directory unless instructed otherwise.
- Keep configuration changes documented, especially under `/etc`.
- Check logs in `/var/log` before making assumptions about failures.
- Avoid storing important data in `/tmp`.
- Use symbolic links when you need a convenient alias to another file or directory.

---

## ❌ Common Mistakes

- Confusing `/` with `/root`.
- Deleting files in system directories without understanding their purpose.
- Editing configuration files without creating backups.
- Assuming `/tmp` is permanent storage.
- Following a broken symbolic link without verifying the original target exists.

---

## 📚 Recommended Documentation

- `man hier` (Filesystem Hierarchy)
- `man ln`
- Ubuntu Server documentation: https://ubuntu.com/server/docs
- The Linux Documentation Project: https://tldp.org/

---

## 📖 Summary

Today you learned:

✅ The Linux Filesystem Hierarchy Standard (FHS)
✅ The purpose of essential directories like `/etc`, `/var`, `/home`, and `/usr`
✅ The difference between `/` and `/root`
✅ Common Linux file types
✅ Hard links vs. symbolic links
✅ Practical filesystem navigation for production environments

These concepts are fundamental when managing servers, Docker containers, Kubernetes nodes, and cloud instances.

