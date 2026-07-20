# 👨‍💻 Coding / Scripting Exercise

## Exercise 1: Create Production Directory Structure

Create this structure using terminal commands:

```
mern-production/
├── frontend
├── backend
├── logs
├── configs
└── scripts
```

Then:
1. Create `backend/app.js`.
2. Create a symbolic link named `backend-latest` pointing to the `backend` directory.
3. Verify with:
   ```bash
   ls -l
   ```

---

# 🔬 Hands-on Lab

## Lab: Explore Your Linux Filesystem

### Goal
Become comfortable navigating and understanding the Linux filesystem layout.

### Instructions
1. Visit each of the following directories:
   - `/`
   - `/home`
   - `/etc`
   - `/var/log`
   - `/usr`
   - `/proc`

2. Record **one interesting file or subdirectory** from each location.

3. Create a markdown file summarizing what each directory is used for.

### Example Output Format
```markdown
# Filesystem Exploration Notes

## /
- Interesting: Contains all other directories
- Purpose: Root directory

## /etc
- Interesting: hosts file maps hostnames to IPs
- Purpose: System configuration files

## /var/log
- Interesting: syslog contains system messages
- Purpose: Log files

## /proc
- Interesting: cpuinfo shows CPU details
- Purpose: Virtual filesystem for kernel/process info
```

---

# 📋 Mini Assignment

## Assignment: Prepare an Ubuntu Server for a MERN Application

Imagine you're preparing an Ubuntu server for a MERN application.

Write a short document (at least 4 paragraphs) explaining:

1. **Where you would place the application code.**
   - Under `/home/ubuntu/` or `/opt/` and why.

2. **Where configuration files would live.**
   - Under `/etc/` or within the app directory and why.

3. **Where you'd look for logs if the application crashed.**
   - Under `/var/log/` and what specific logs to check.

4. **Why `/tmp` is not appropriate for storing uploaded user files.**
   - Reference: temporary nature, auto-cleaning, security concerns.

---

# Additional Practice: Directory Tree

Create a directory structure for a multi-service MERN deployment:

```
mern-stack/
├── frontend/
│   ├── public/
│   └── src/
│       ├── components/
│       └── pages/
├── backend/
│   ├── routes/
│   ├── models/
│   ├── controllers/
│   └── middleware/
├── database/
│   ├── migrations/
│   └── seeds/
├── config/
├── logs/
└── scripts/
```

Use `mkdir -p` to create it, then verify with `ls -R`.

