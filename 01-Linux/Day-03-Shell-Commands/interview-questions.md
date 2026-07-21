# 💼 Interview Questions (Shell Commands) — with Detailed Answers & Examples

## Beginner

### 1. What is the difference between `cp` and `mv`?
**Answer:**
- `cp` **copies** a file/directory — the original remains in place.
- `mv` **moves** or **renames** a file/directory — the original is removed.

**Example:**
```bash
cp app.log backup.log   # app.log still exists
mv app.log old.log      # app.log is now called old.log
```

---

### 2. What does `tail -f` do?
**Answer:** `tail -f` displays the last lines of a file and then **continuously follows** the file, showing new lines as they are added in real time.

**Example:** Monitoring a live log:
```bash
tail -f /var/log/nginx/access.log
```
Press `Ctrl + C` to stop.

---

### 3. How do you check available disk space?
**Answer:** Use `df -h` (disk free, human-readable).

**Example:**
```bash
df -h
```
Output:
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   18G   30G  38% /
```

---

### 4. What is the purpose of `find`?
**Answer:** `find` searches for files and directories based on criteria like name, type, size, or modification time.

**Example:** Find all `.log` files:
```bash
find /var/log -name "*.log"
```

---

### 5. What is the difference between `>` and `>>`?
**Answer:**
- `>` **overwrites** the destination file with new content.
- `>>` **appends** content to the end of the destination file.

**Example:**
```bash
echo "First line" > file.txt    # file.txt contains: First line
echo "Second line" > file.txt   # file.txt contains: Second line (overwritten)
echo "Third line" >> file.txt   # file.txt contains: Second line\nThird line (appended)
```

---

## Intermediate

### 6. Why is `less` preferred over `cat` for large log files?
**Answer:** 
- `cat` dumps the **entire** file into the terminal, which can freeze or flood the terminal for large files.
- `less` loads the file **page by page**, allowing navigation, searching, and scrolling without memory issues.

**Example:** View a 500MB log file:
```bash
less /var/log/syslog   # Smooth, navigable
cat /var/log/syslog    # Floods terminal, hard to read
```

---

### 7. How would you locate all `.env` files in a project?
**Answer:** Use `find` with a name pattern:
```bash
find /home/ubuntu/projects -name ".env"
```

---

### 8. How would you determine which directory is consuming the most disk space?
**Answer:** Use `du` with sorting:
```bash
du -sh /home/ubuntu/* | sort -rh | head -5
```
- `du -sh` shows sizes
- `sort -rh` sorts by size (human-readable, reverse)
- `head -5` shows top 5

---

### 9. Explain how pipes improve command-line productivity.
**Answer:** Pipes (`|`) let you **chain commands** together, sending the output of one command as input to another. This avoids creating intermediate files and enables complex data processing in one line.

**Example without pipe:**
```bash
history > hist.txt
grep "docker" hist.txt
```

**Example with pipe:**
```bash
history | grep "docker"
```

---

### 10. When would you use `tail -100` during troubleshooting?
**Answer:** When you need to see the **last 100 lines** of a log file to understand recent events before a failure. This is useful when the default 10 lines aren't enough context.

**Example:**
```bash
tail -100 /var/log/application.log
```

---

## Advanced

### 11. A production server reports "No space left on device." Describe your troubleshooting process using Linux commands.
**Answer:**
1. **Check overall disk usage:**
   ```bash
   df -h
   ```
2. **Find large directories:**
   ```bash
   du -sh /var/* | sort -rh | head -10
   ```
3. **Check inode usage** (if many small files):
   ```bash
   df -i
   ```
4. **Find large files:**
   ```bash
   find / -type f -size +100M -exec ls -lh {} \; | sort -k5 -rh
   ```
5. **Check deleted files held open by processes:**
   ```bash
   lsof | grep deleted
   ```
6. **Take action:** Clear logs, remove temp files, or add more storage.

---

### 12. Explain the risks of recursive deletion with `rm -r`.
**Answer:** `rm -r` **permanently** deletes directories and all their contents with **no trash/recovery**. A single mistake (wrong path, typo, wildcard) can destroy critical data or system files.

**Safest practices:**
- Always verify with `ls` before deleting.
- Use `rm -ri` (interactive mode) to confirm each deletion.
- Use full paths instead of relative paths.
- Avoid `rm -rf` with wildcards in production.

---

### 13. How would you safely rotate and archive large application logs?
**Answer:** A safe manual log rotation process:
1. **Copy and compress the log:**
   ```bash
   cp app.log app.log.$(date +%Y%m%d)
   gzip app.log.$(date +%Y%m%d)
   ```
2. **Truncate the original log** (instead of deleting):
   ```bash
   > app.log
   ```
   This preserves the file handle so the application can keep writing.
3. **Verify the application is still logging:**
   ```bash
   tail -f app.log
   ```

**Better approach:** Use `logrotate` for automated rotation.

---

### 14. Describe a workflow for diagnosing a crashing Node.js application using shell commands.
**Answer:**
1. **Navigate to the application directory:**
   ```bash
   cd /home/ubuntu/backend
   ```
2. **Check recent logs:**
   ```bash
   tail -100 /var/log/mern/backend.log
   ```
3. **Follow the log in real time while reproducing the crash:**
   ```bash
   tail -f /var/log/mern/backend.log
   ```
4. **Check disk space and memory:**
   ```bash
   df -h
   free -h
   ```
5. **Grep for error patterns:**
   ```bash
   grep -i "error\|exception\|fatal" /var/log/mern/backend.log | tail -50
   ```
6. **Check if the process is running:**
   ```bash
   ps aux | grep node
   ```

---

### 15. How do output redirection and pipes enable automation in shell scripts?
**Answer:** Output redirection and pipes are the foundation of shell scripting automation:
- **Redirection** (`>`, `>>`) lets scripts save results, log progress, and handle errors.
- **Pipes** (`|`) let scripts chain operations without intermediate files.

**Example automation script:**
```bash
#!/bin/bash
# Backup script with logging

LOGFILE="/var/log/backup.log"

echo "Backup started: $(date)" >> $LOGFILE
df -h | grep "/dev/sda1" >> $LOGFILE
tar -czf /backups/project.tar.gz /home/ubuntu/project 2>> error.log
echo "Backup completed: $(date)" >> $LOGFILE
