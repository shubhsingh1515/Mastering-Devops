# Day-03: Shell Commands (Essential Linux Commands for DevOps)

## 📅 Phase 1: Linux
- **Topic:** Shell Commands (Essential Linux Commands for DevOps)
- **Estimated Time:** 25–30 minutes
- **Goal:** Learn the Linux commands you will use every day as a DevOps engineer to manage files, inspect systems, troubleshoot issues, and automate tasks.

---

## 🎯 Learning Objectives

By the end of today's lesson, you'll be able to:

- Create, copy, move, rename, and delete files.
- Search for files.
- Read large log files efficiently.
- Check disk usage.
- Redirect command output.
- Combine commands using pipes.
- Apply these skills while managing MERN applications on Linux servers.

---

## 🤔 Why This Topic Is Important

Imagine you're on-call at 2 AM.

Your production Node.js API is failing.

You'll need to:

- Find logs.
- Search configuration files.
- Inspect disk usage.
- Copy backups.
- Remove temporary files.
- Read only the last few log lines.

Every one of these tasks relies on shell commands.

These commands form the daily toolkit of every DevOps engineer.

---

## 📖 Theory Explained in Simple Language

Think of the Linux shell as a conversation.

You give commands:

```
You
 ↓
Shell (Bash)
 ↓
Linux Kernel
 ↓
Hardware
```

The shell interprets your commands and asks the operating system to perform them.

### File Management Commands

**Create an Empty File**
```bash
touch app.log
```
Creates: `app.log`

**Copy Files**
```bash
cp app.log backup.log
```
Copies a file. Copy a directory recursively:
```bash
cp -r project backup-project
```

**Move or Rename**

Rename:
```bash
mv app.log server.log
```

Move:
```bash
mv server.log logs/
```

**Delete Files**
```bash
rm server.log
```
Delete a directory:
```bash
rm -r old-project
```

⚠️ **Be careful:** `rm` permanently deletes files.

### Viewing File Contents

**Display Entire File**
```bash
cat app.log
```
Good for small files.

**View One Screen at a Time**
```bash
less app.log
```
Useful for large log files.

Controls:
- `Space` → Next page
- `b` → Previous page
- `q` → Quit

**First Few Lines**
```bash
head app.log
```
Default: first 10 lines. Specify the number of lines:
```bash
head -20 app.log
```

**Last Few Lines**
```bash
tail app.log
```
Perfect for logs.

**Follow a Growing Log File**
```bash
tail -f app.log
```
As new lines are added, they appear in real time. This is one of the most frequently used commands in production. Press `Ctrl + C` to stop.

### Searching for Files

**Find**
```bash
find . -name "*.js"
```
Finds all JavaScript files from the current directory.

Search for Dockerfiles:
```bash
find . -name "Dockerfile"
```

### Disk Usage

**Check Free Disk Space**
```bash
df -h
```
Example:
```
Filesystem     Size  Used Avail
/dev/sda1       50G  18G   30G
```
The `-h` flag means "human-readable."

**Directory Sizes**
```bash
du -sh logs
```
Example: `250M logs`

### Command History

View recent commands:
```bash
history
```

Repeat the last command:
```bash
!!
```

Repeat command number 120:
```bash
!120
```

### Output Redirection

Overwrite a file:
```bash
echo "Hello DevOps" > notes.txt
```

Append to a file:
```bash
echo "Day 3 completed" >> notes.txt
```

**Difference:**
- `>` overwrites
- `>>` appends

### Pipes

One of Linux's greatest strengths.

```bash
history | tail
```

```
history
   ↓
  tail
```

Another example:
```bash
ls -l | less
```

Large directory listings become easier to read.

---

## 🌍 Real-World Example

Your MERN backend logs are stored in:
```
/var/log/mern/backend.log
```

You receive an alert.

**Steps:**

1. View the latest errors:
   ```bash
   tail -50 /var/log/mern/backend.log
   ```

2. Watch the log live:
   ```bash
   tail -f /var/log/mern/backend.log
   ```

3. Check disk usage:
   ```bash
   df -h
   ```

4. See if logs are consuming space:
   ```bash
   du -sh /var/log/mern
   ```

These are common first steps in production troubleshooting.

---

## 🏗️ Architecture Diagram (ASCII)

```
                Terminal
                    │
              Shell (Bash)
                    │
     ┌──────────────┼──────────────┐
     │              │              │
 File Commands   Search       Disk Usage
     │              │              │
 touch cp mv    find ls      du df tail
```

---

## 🛠️ Step-by-Step Practical Implementation

**Step 1** — Create a practice workspace:
```bash
mkdir -p ~/devops-course/day3
cd ~/devops-course/day3
```

**Step 2** — Create files:
```bash
touch app.log
touch server.log
```

**Step 3** — Copy a file:
```bash
cp app.log app-backup.log
```

**Step 4** — Rename:
```bash
mv server.log backend.log
```

**Step 5** — View:
```bash
ls -l
```

**Step 6** — Write text:
```bash
echo "Server started" > backend.log
echo "Database connected" >> backend.log
```

**Step 7** — Read the file:
```bash
cat backend.log
```

**Step 8** — Read the last line:
```bash
tail -1 backend.log
```

**Step 9** — Check disk space:
```bash
df -h
```

**Step 10** — Check folder size:
```bash
du -sh .
```

**Step 11** — Search for log files:
```bash
find . -name "*.log"
```

---

## ✅ Best Practices

- Prefer `less` over `cat` for large files.
- Use `tail -f` when monitoring live logs.
- Double-check your path before running `rm`.
- Use `cp -r` only when copying directories.
- Monitor disk usage regularly with `df -h` and `du -sh`.
- Redirect logs with `>>` instead of `>` when you want to preserve existing content.

---