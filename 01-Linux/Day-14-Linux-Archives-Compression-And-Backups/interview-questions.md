# Day 14 — Interview Questions: Linux Archives, Compression & Backups

## Beginner Level

### 1. What is the difference between archiving and compression?

**Answer:** Archiving combines multiple files into one file, while compression reduces the size of that file.

### 2. What does `tar` do?

**Answer:** `tar` creates and extracts archive files, usually without compressing them by default.

### 3. What does `.tar.gz` mean?

**Answer:** It means a tar archive that was compressed with `gzip`.

### 4. How do you extract a `.tar.gz` file?

**Answer:**

```bash
tar -xzf archive.tar.gz
```

### 5. How can you view the contents of an archive without extracting it?

**Answer:**

```bash
tar -tzf archive.tar.gz
```

---

## Intermediate Level

### 6. Why is `tar` commonly used in Linux deployments?

**Answer:** It is reliable for packaging app files, config files, and deployment artifacts into a single manageable unit.

### 7. Compare `gzip` and `xz`.

**Answer:** Both compress files, but `xz` often provides better compression ratios while using more CPU and time.

### 8. How would you transfer a release archive to a remote server?

**Answer:** Use `scp` or another secure transfer method, for example:

```bash
scp app.tar.gz ubuntu@server:/tmp/
```

### 9. What files should be included in a production backup?

**Answer:** Application code, config files, environment variables, database backup, certificates, and any persistent data needed for restore.

### 10. Why should you test backups by restoring them?

**Answer:** A backup is only useful if it can be restored successfully during a real incident.

---

## Advanced Level

### 11. Design a backup strategy for a production MERN application.

**Answer:** Include application code backups, Nginx config backups, environment file protection, database backup using `mongodump`, remote storage, retention periods, encryption, and a restore drill.

### 12. How would you avoid downtime while creating backups?

**Answer:** Use consistent file snapshots, backup the application while it is quiescent if needed, take database backups during a stable window, and validate that the app remains healthy after backup execution.

### 13. Why shouldn't you rely on copying MongoDB's live data directory as your primary backup?

**Answer:** Live database files may be inconsistent while the database is running; proper database backups ensure consistency and recovery integrity.

### 14. How would you securely store production backups?

**Answer:** Use encrypted backups, store them off-site or on a separate system, restrict access with least privilege, and rotate old backups based on retention policy.

### 15. How would you automate backup creation, retention, verification, and cleanup?

**Answer:** Use a Bash script with timestamps, run it with cron, verify the archive with `tar -tzf`, and delete old backups beyond the retention window.

---

## Scenario-Based Questions

### 16. Your deployment archive is missing necessary files. What should you do?

**Answer:** Inspect the archive contents before extraction using `tar -tzf` and verify the expected directories and files are included.

### 17. A user asks you to restore an application from a backup. What is your process?

**Answer:** Confirm the backup integrity, copy or mount the archive, extract it into the target location, validate configuration, restart the service, and verify the application is healthy.

### 18. Why is it dangerous to extract a random archive as root?

**Answer:** Untrusted archives may contain malicious paths, overwrite critical files, or execute harmful scripts if used incorrectly.

---

## Short Answer Practice

- What is `tar -cf`?
- What is `tar -xzf`?
- What is `gzip` used for?
- Why do DevOps engineers keep backup names timestamped?
- Why do we inspect backups before restoring them?

### Expected Answers
- Create a tar archive
- Extract a gzip-compressed tar archive
- Compress files to reduce size
- To avoid overwriting previous backup files
- To confirm the archive is valid and contains the right files
