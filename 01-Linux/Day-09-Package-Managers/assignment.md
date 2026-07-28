# Day 09 — Assignment: Linux Package Managers

## Objective
Apply what you've learned about package managers to audit installed tools, provision a server, and build a reusable provisioning checklist.

---

## Part 1: Coding Exercise — package_audit.sh

Create a script named `package_audit.sh` that checks which common DevOps tools are installed on a system.

### Requirements:
1. Check if Git is installed and show its version (or "Git not installed")
2. Check if Node.js is installed and show its version (or "Node.js not installed")
3. Check if Docker is installed and show its version (or "Docker not installed")
4. Check if Nginx is installed and show its version (or "Nginx not installed")
5. Use `2>/dev/null` to suppress error messages for missing tools

### Solution:

```bash
#!/bin/bash

echo "=== Git ==="
git --version 2>/dev/null || echo "Git not installed"

echo
echo "=== Node.js ==="
node --version 2>/dev/null || echo "Node.js not installed"

echo
echo "=== Docker ==="
docker --version 2>/dev/null || echo "Docker not installed"

echo
echo "=== Nginx ==="
nginx -v 2>&1 || echo "Nginx not installed"
```

### To run:
```bash
chmod +x package_audit.sh
./package_audit.sh
```

---

## Part 2: Hands-on Lab — Prepare a Fresh Ubuntu Server

**Scenario:** Imagine you're provisioning a new Ubuntu server. Complete the following steps and record your results.

### Steps:

1. **Refresh package metadata:**
   
```
bash
   sudo apt update
   
```
   *Record the output:*

2. **Search for packages:**
   
```
bash
   apt search git
   apt search nginx
   apt search nodejs
   
```
   *How many results did each search return?*

3. **View package details for Git:**
   
```bash
   apt show git
   
```
   *Record: Version, Description, Dependencies, Homepage*

4. **Install Git if it's missing:**
   
```
bash
   sudo apt install git
   
```

5. **Verify the installation:**
   
```
bash
   git --version
   
```
   *Record the version:*

6. **List installed packages and identify five packages already present:**
   
```
bash
   apt list --installed | head -20
   
```
   *List five packages and their purposes:*

| Package | Purpose |
|---------|---------|
| 1. | |
| 2. | |
| 3. | |
| 4. | |
| 5. | |

---

## Part 3: Mini Assignment — Server Provisioning Checklist

Write a **Server Provisioning Checklist** for a new Ubuntu machine intended to host a MERN (MongoDB, Express, React, Node.js) application.

### For each step, include:
- The exact command(s) to run
- Why the step is important

| Step | Command(s) | Why It's Performed |
|------|-----------|-------------------|
| 1. Update package metadata | `sudo apt update` | Ensures I'm installing the latest versions from repositories |
| 2. Install Git | | |
| 3. Install Node.js & npm | | |
| 4. Install Nginx | | |
| 5. Install Docker | | |
| 6. Verify all versions | | |
| 7. Clean up unused packages | | |

---

## Part 4: Knowledge Check

Answer the following questions in your own words:

### 1. What is the difference between `apt update` and `apt upgrade`?
```
Your answer:
```

### 2. What is a package repository and why is it important?
```
Your answer:
```

### 3. When would you use `apt purge` instead of `apt remove`?
```
Your answer:
```

### 4. Why is dependency resolution important in package management?
```
Your answer:
```

### 5. Which package manager would you use on:
   - Ubuntu → __________
   - Fedora → __________
   - Older CentOS → __________
   - Rocky Linux → __________

---

## Part 5: Research Challenge

Research and answer the following:

1. **What is the difference between `apt` and `apt-get`?**

   <details>
   <summary>Answer</summary>

   `apt` is a newer, more user-friendly interface that combines commonly used commands from `apt-get` and `apt-cache`. `apt-get` is the older, more low-level tool. Both can install packages, but `apt` provides progress bars and more readable output. For most use cases, `apt` is recommended.
   </details>

