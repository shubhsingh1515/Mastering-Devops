# Day 14: Linux Archives, Compression & Backups

## 1. 📚 Topic Name

Linux Archives & Compression

---

## 2. 🎯 Learning Objectives

By the end of today's lesson, you'll understand:

- What an archive is.
- The difference between archiving and compression.
- How `tar` works.
- `gzip`, `gunzip`, and `xz`.
- How to create and extract archives.
- How to inspect archives before extracting them.
- How to create application backups.
- How archives fit into MERN deployments and disaster recovery.

---

## 3. 🤔 Why This Matters in DevOps

Suppose your MERN application contains:

```text
backend/
frontend/
config/
scripts/
```

You need to transfer the release to another server.

Sending thousands of individual files is inefficient.

Instead:

```text
Application files
       ↓
    tar archive
       ↓
   compressed
       ↓
 app-release.tar.gz
       ↓
      scp
       ↓
Production server
```

Archives are also useful for:

- Backups
- Deployment artifacts
- Log archives
- Configuration snapshots
- Disaster recovery

---

## 4. 📖 Theory Explained

### Archive vs Compression

These are different concepts.

#### Archiving

Combines multiple files into one:

```text
file1
file2
file3

↓

archive.tar
```

#### Compression

Reduces the size:

```text
archive.tar

↓

archive.tar.gz
```

`tar` primarily handles archiving.

`gzip` handles compression.

Together:

```text
tar + gzip = .tar.gz
```

---

### 5. tar

`tar` is one of the most important Linux tools for packaging files.

#### Create an Archive

```bash
tar -cf app.tar backend/
```

Meaning:

- `c` → create
- `f` → file/archive name

#### Extract an Archive

```bash
tar -xf app.tar
```

- `x` → extract
- `f` → archive file

#### View Contents Without Extracting

```bash
tar -tf app.tar
```

This is a very useful production habit.

Inspect before extracting.

---

### 6. gzip

Compress a file:

```bash
gzip app.tar
```

Produces:

```text
app.tar.gz
```

Decompress:

```bash
gunzip app.tar.gz
```

---

### 7. Create a .tar.gz Directly

Most commonly you'll use:

```bash
tar -czf app.tar.gz backend/
```

Options:

- `c` → create
- `z` → gzip
- `f` → filename

Extract:

```bash
tar -xzf app.tar.gz
```

---

### 8. xz

`xz` is another compression format.

#### Create:

```bash
tar -cJf app.tar.xz backend/
```

#### Extract:

```bash
tar -xJf app.tar.xz
```

Generally, `xz` can achieve stronger compression than `gzip`, often at the cost of more CPU/time.

---

### 9. zip and unzip

ZIP is common when working across Linux and Windows.

Create:

```bash
zip -r app.zip backend/
```

Extract:

```bash
unzip app.zip
```

List contents:

```bash
unzip -l app.zip
```

---

### 10. 🌍 MERN Production Example

Suppose you're preparing a release:

```text
mern-app/
├── backend/
├── frontend/dist/
├── nginx/
└── scripts/
```

Create an artifact:

```bash
tar -czf mern-release-2026-08-08.tar.gz mern-app/
```

Inspect it:

```bash
tar -tzf mern-release-2026-08-08.tar.gz
```

Transfer it:

```bash
scp mern-release-2026-08-08.tar.gz ubuntu@server:/tmp/
```

On the server:

```bash
tar -xzf /tmp/mern-release-2026-08-08.tar.gz
```

This is a simple version of the artifact-based deployment pattern you'll later automate with CI/CD.

---

### 11. 🗄️ Backup Example

A simple configuration backup:

```bash
tar -czf config-backup.tar.gz /etc/nginx/
```

Application backup:

```bash
tar -czf app-backup.tar.gz /opt/mern/
```

But remember:

A file backup is not automatically a database backup.

For MongoDB, you would normally use a database-aware backup mechanism such as `mongodump`, rather than simply copying live database files.

---

### 12. 🏗️ Backup Architecture

```text
              Production Server

                    │
          ┌─────────┴─────────┐
          │                   │
       App Files           MongoDB
          │                   │
         tar              mongodump
          │                   │
          └─────────┬─────────┘
                    │
                 Backups
                    │
              Remote Storage
```

A production backup strategy should also consider:

- Retention
- Encryption
- Off-site storage
- Restore testing
- Access control

> A backup that has never been restored successfully is not a proven recovery strategy.

---

### 13. 🛠️ Practical Implementation

Create today's workspace:

```bash
mkdir -p ~/devops-course/day14/mern-app/{backend,frontend,config,scripts}
cd ~/devops-course/day14
```

Create sample files:

```bash
echo "Node backend" > mern-app/backend/server.js
echo "React frontend" > mern-app/frontend/index.html
echo "production config" > mern-app/config/app.conf
echo "# deployment scripts" > mern-app/scripts/deploy.sh
```

Create an archive:

```bash
tar -cf mern-app.tar mern-app/
```

Check its contents:

```bash
tar -tf mern-app.tar
```

Compress it:

```bash
gzip mern-app.tar
```

You now have:

```text
mern-app.tar.gz
```

---

### 14. Extract the Archive

Create a test extraction directory:

```bash
mkdir extracted
```

Extract:

```bash
tar -xzf mern-app.tar.gz -C extracted/
```

Verify:

```bash
find extracted/
```

---

## 15. 💻 Commands Cheat Sheet

| Command | Purpose |
|---------|---------|
| `tar -cf archive.tar dir/` | Create archive |
| `tar -xf archive.tar` | Extract archive |
| `tar -tf archive.tar` | List contents |
| `tar -czf archive.tar.gz dir/` | Create gzip-compressed archive |
| `tar -xzf archive.tar.gz` | Extract `.tar.gz` |
| `gzip file` | Compress with gzip |
| `gunzip file.gz` | Decompress gzip |
| `tar -cJf archive.tar.xz dir/` | Create xz archive |
| `tar -xJf archive.tar.xz` | Extract xz archive |
| `zip -r archive.zip dir/` | Create ZIP |
| `unzip archive.zip` | Extract ZIP |
| `unzip -l archive.zip` | List ZIP contents |

---

## 16. ✅ Best Practices

- Inspect archives before extracting them.
- Don't blindly extract untrusted archives as root.
- Keep production backups separate from the server being backed up.
- Use timestamps in backup filenames.
- Encrypt sensitive backups.
- Define retention periods.
- Regularly test restores.
- Don't treat MongoDB's live data directory as a substitute for a proper database backup.

---

## 17. ❌ Common Mistakes

### Mistake 1: Confusing tar with compression

`tar` packages files; compression tools reduce size.

### Mistake 2: Extracting directly into production

Always inspect the archive first:

```bash
tar -tzf release.tar.gz
```

### Mistake 3: Backing up only application code

Your production recovery plan may also require:

- Database
- Environment/configuration
- Nginx configuration
- Certificates
- Persistent data

### Mistake 4: Never testing restores

A backup is useful only if you can actually recover from it.

---

## 18. 💼 Interview Preparation

### Beginner
- What is the difference between archiving and compression?
- What does `tar` do?
- What does `.tar.gz` mean?
- How do you extract a `.tar.gz` file?
- How do you view archive contents without extracting them?

### Intermediate
- Why is `tar` commonly used in Linux deployments?
- Compare `gzip` and `xz`.
- How would you transfer a release archive to a remote server?
- What files should be included in a production backup?
- Why should you test backups by restoring them?

### Advanced
- Design a backup strategy for a production MERN application.
- How would you avoid downtime while creating backups?
- Why shouldn't you rely on copying MongoDB's live data directory as your primary backup strategy?
- How would you securely store production backups?
- How would you automate backup creation, retention, verification, and cleanup?

---

## 19. 📝 Quiz

### Questions
1. What does `tar -c` mean?
2. What does `tar -x` mean?
3. What does `tar -t` do?
4. What does the `z` option represent?
5. What command extracts a `.tar.gz` file?
6. How do you create a ZIP archive?
7. How do you inspect an archive before extraction?
8. What is the difference between an application backup and a database backup?
9. Why should backups be stored away from the production server?
10. Why must restores be tested?

### ✅ Answers
- Create.
- Extract.
- List contents.
- `gzip` compression.
- `tar -xzf archive.tar.gz`
- `zip -r archive.zip directory/`
- For example, `tar -tzf archive.tar.gz`.
- They use different recovery mechanisms and protect different types of data.
- A server failure or compromise could destroy both the production data and its local backup.
- To prove that the backup is actually recoverable.

---

## 20. 👨‍💻 Coding Exercise

Create `backup_app.sh`:

```bash
#!/bin/bash

APP_DIR="/opt/mern"
BACKUP_DIR="/var/backups/mern"
TIMESTAMP=$(date +%Y-%m-%d_%H-%M-%S)

mkdir -p "$BACKUP_DIR"

tar -czf "$BACKUP_DIR/mern-$TIMESTAMP.tar.gz" "$APP_DIR"

if [ $? -eq 0 ]; then
    echo "Backup successful: $BACKUP_DIR/mern-$TIMESTAMP.tar.gz"
else
    echo "Backup failed"
    exit 1
fi
```

Notice how today's lesson combines:

- Bash variables
- `date`
- `mkdir`
- `tar`
- Exit-code checking

This is exactly how earlier lessons begin connecting into automation.

---

## 21. 🧪 Hands-on Assessment

### Scenario

Your production MERN server contains:

- `/opt/mern/`
- `/etc/nginx/`
- `/var/log/nginx/`

Create a backup plan that:

### Task 1

Archives the application:

```bash
tar -czf mern-app.tar.gz /opt/mern/
```

### Task 2

Archives Nginx configuration:

```bash
tar -czf nginx-config.tar.gz /etc/nginx/
```

### Task 3

Inspect both archives before extracting them.

### Task 4

Transfer one archive to another machine using:

```bash
scp
```

### Task 5

Explain separately how you would back up MongoDB.

### Task 6

Describe how you would restore the application after a server failure.

---

## 22. 📋 Monthly Cumulative Project – Month 1

You've now completed two weeks of Linux foundations, so it's time to begin the first cumulative real-world project.

### 🏗️ Project: Production MERN Linux Server

### Scenario

You're the DevOps engineer for a startup preparing its first production deployment.

You must design and operate a Linux server hosting:

```text
Internet
   │
 Nginx
   │
 Node.js API
   │
 MongoDB
```

### Project Requirements

#### Infrastructure
- Ubuntu Linux server
- Dedicated application user
- Secure SSH access
- Proper file permissions

#### Application
- MERN backend
- Node.js service managed with `systemd`
- Nginx reverse proxy

#### Operations
Create:

- Server health script
- Deployment script
- Backup script
- Application health check
- Log analysis commands
- Automation

Use:

- `cron`
- Bash
- `systemctl`
- Backup

### Your design must include:

- Application backup
- Configuration backup
- MongoDB backup strategy
- Backup retention
- Restore procedure

### Security
Document:

- SSH key permissions
- Service users
- Least privilege
- `.env` protection
- Why production services should not run as root

### 🎤 Project Interview Challenge

Be prepared to explain:

> "How would you deploy and operate a production MERN application on a Linux server?"

Your answer should connect the concepts you've learned across Days 1–14 rather than listing commands independently.

A strong answer should flow roughly like:

```text
Provision server
      ↓
Secure SSH
      ↓
Create users
      ↓
Install packages
      ↓
Deploy application
      ↓
Configure permissions
      ↓
Create systemd service
      ↓
Configure Nginx
      ↓
Add health checks
      ↓
Centralize/rotate logs
      ↓
Automate backups
      ↓
Test recovery
```

This project will continue to evolve as the course introduces Docker, CI/CD, cloud infrastructure, and Kubernetes.

---

## 23. 📖 Today's Key Takeaways

You learned:

- ✅ Archiving vs compression
- ✅ `tar`
- ✅ `gzip`
- ✅ `xz`
- ✅ ZIP
- ✅ Archive inspection
- ✅ Application backups
- ✅ Production backup principles
- ✅ How today's tools combine with Bash and SSH
- ✅ How to design a real MERN recovery strategy

The important DevOps progression is now becoming visible:

```text
Linux
  ↓
Bash
  ↓
Cron
  ↓
Logs
  ↓
Archives & Backups
  ↓
Automation
  ↓
Production Operations
```

---

## Bonus: Practical Backup Script Example

```bash
#!/bin/bash

APP_DIR="/opt/mern"
BACKUP_DIR="/var/backups/mern"
TIMESTAMP=$(date +%Y-%m-%d_%H-%M-%S)

mkdir -p "$BACKUP_DIR"

if [ -d "$APP_DIR" ]; then
    tar -czf "$BACKUP_DIR/mern-$TIMESTAMP.tar.gz" "$APP_DIR"
    echo "Backup created: $BACKUP_DIR/mern-$TIMESTAMP.tar.gz"
else
    echo "Application directory not found: $APP_DIR"
    exit 1
fi
```

This is a small but realistic example of the type of automation DevOps engineers use in real deployments.

---

## Final Reflection

Today's lesson connects everything you have learned so far. You are no longer just running commands—you are learning how to package, move, and recover real application systems.

This is a critical step toward production operations, release management, and disaster recovery.

The ability to create a clean archive, verify it, and restore it safely is a core DevOps skill.
