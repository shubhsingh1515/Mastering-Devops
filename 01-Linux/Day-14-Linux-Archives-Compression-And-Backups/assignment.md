# Day 14 — Assignment: Linux Archives, Compression & Backups

## Objective
Apply what you've learned about archiving, compression, and backup strategies for a real-world MERN deployment.

---

## Part 1: Create Your Practice Workspace

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

---

## Part 2: Create and Inspect an Archive

Create a `.tar` archive:

```bash
tar -cf mern-app.tar mern-app/
```

List archive contents:

```bash
tar -tf mern-app.tar
```

Create a compressed archive:

```bash
tar -czf mern-app.tar.gz mern-app/
```

Inspect the compressed archive without extracting:

```bash
tar -tzf mern-app.tar.gz
```

---

## Part 3: Extract the Archive

Create a test folder:

```bash
mkdir extracted
```

Extract the archive:

```bash
tar -xzf mern-app.tar.gz -C extracted/
```

Verify the extracted files:

```bash
find extracted/
```

---

## Part 4: Create a Backup Script

Create a script named `backup_app.sh`:

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

Make it executable:

```bash
chmod +x backup_app.sh
```

---

## Part 5: ZIP Practice

Create a ZIP archive for the app:

```bash
zip -r mern-app.zip mern-app/
```

List ZIP contents:

```bash
unzip -l mern-app.zip
```

Extract it to a different folder:

```bash
mkdir zip-extract
unzip mern-app.zip -d zip-extract/
```

---

## Part 6: Hands-on Backup Scenario

Imagine your production MERN app runs under:

```bash
/opt/mern/
```

Create a backup of the app and its Nginx config:

```bash
tar -czf mern-app-backup.tar.gz /opt/mern/
tar -czf nginx-config-backup.tar.gz /etc/nginx/
```

Inspect the archives:

```bash
tar -tzf mern-app-backup.tar.gz
tar -tzf nginx-config-backup.tar.gz
```

---

## Part 7: Reflection Questions

Answer the following in your own words:

1. What is the difference between `tar` and `gzip`?
2. Why is `tar -tzf archive.tar.gz` useful before extraction?
3. Why should a production backup not rely only on app files?
4. What is the advantage of timestamping backup filenames?
5. Why should backup restores be tested regularly?

---

## Bonus Challenge

Create a shell script that:

- Creates a timestamped backup of `/opt/mern/`
- Stores it in `/var/backups/mern/`
- Keeps only the last 5 backups
- Prints a success or failure message

Example logic:

```bash
ls -t /var/backups/mern/*.tar.gz | tail -n +6 | xargs -r rm -f
```

---

## Success Criteria

You should be able to:

- Create a `.tar` archive
- Create a `.tar.gz` archive
- Extract an archive safely
- Inspect its contents before extraction
- Explain why backups matter in production
- Use shell commands to package and recover app files

---

## Detailed Worked Answers

### Part 1: Create Your Practice Workspace

```bash
mkdir -p ~/devops-course/day14/mern-app/{backend,frontend,config,scripts}
cd ~/devops-course/day14
```

This creates a realistic project layout for a MERN-like app. The directory structure is useful because production systems often contain multiple app components such as backend code, frontend build files, configuration, and deployment scripts.

```bash
echo "Node backend" > mern-app/backend/server.js
echo "React frontend" > mern-app/frontend/index.html
echo "production config" > mern-app/config/app.conf
echo "# deployment scripts" > mern-app/scripts/deploy.sh
```

These sample files simulate a real app release. In practice, your backend and frontend directories may contain much more content, but the idea is the same: package the whole directory into a reusable archive.

---

### Part 2: Create and Inspect an Archive

Create a `.tar` archive:

```bash
tar -cf mern-app.tar mern-app/
```

This command packages the `mern-app` folder into a single archive file named `mern-app.tar`.

- `c` = create
- `f` = file name

List archive contents:

```bash
tar -tf mern-app.tar
```

This prints the file names inside the archive without extracting them. This is important because it lets you confirm the archive contains the expected files before expanding it on a server.

Create a compressed archive:

```bash
tar -czf mern-app.tar.gz mern-app/
```

This is the most common production pattern:

- `tar` archives the files
- `gzip` compresses the archive
- `.tar.gz` means the archive is compressed with gzip

Inspect the compressed archive without extracting:

```bash
tar -tzf mern-app.tar.gz
```

The `t` flag lists archive contents and the `z` flag tells tar to read a gzip-compressed archive. This is a production-safe habit: always inspect the archive before extraction.

---

### Part 3: Extract the Archive

Create a test folder:

```bash
mkdir extracted
```

Extract the archive:

```bash
tar -xzf mern-app.tar.gz -C extracted/
```

This extracts the contents into the `extracted/` directory.

- `x` = extract
- `z` = gzip-compressed archive
- `f` = archive file
- `-C` = change to destination directory

Verify the extracted files:

```bash
find extracted/
```

Expected output will show the unpacked folders such as:

```text
extracted/
extracted/mern-app/
extracted/mern-app/backend/
extracted/mern-app/frontend/
extracted/mern-app/config/
extracted/mern-app/scripts/
```

This confirms the release archive was created correctly and extracted as expected.

---

### Part 4: Create a Backup Script

Create a script named `backup_app.sh`:

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

This script does the following:

1. Defines the application directory to back up.
2. Defines the backup storage directory.
3. Creates a timestamp such as `2026-08-12_14-30-00`.
4. Creates the backup directory if it does not already exist.
5. Uses `tar -czf` to create a compressed archive.
6. Checks the exit code to see if the command succeeded.
7. Prints a success or failure message.

This is a strong example of real-world automation because it combines:

- Variables
- Date formatting
- Directory creation
- Archive creation
- Error handling

Make it executable:

```bash
chmod +x backup_app.sh
```

This makes the script runnable like any normal command.

---

### Part 5: ZIP Practice

Create a ZIP archive for the app:

```bash
zip -r mern-app.zip mern-app/
```

Explanation:

- `zip` creates a ZIP archive
- `-r` means recursive, so all files inside the folder are included

List ZIP contents:

```bash
unzip -l mern-app.zip
```

This lists every file inside the ZIP archive without unpacking it. It's useful for checking if the archive is complete before extracting.

Extract it to a different folder:

```bash
mkdir zip-extract
unzip mern-app.zip -d zip-extract/
```

This extracts the ZIP archive into a new directory named `zip-extract/` so you can compare the output and validate the archive.

ZIP is common when files need to move between Linux and Windows environments, but in Linux production, `.tar.gz` is often preferred.

---

### Part 6: Hands-on Backup Scenario

Imagine your production MERN app runs under:

```bash
/opt/mern/
```

Create a backup of the app and Nginx config:

```bash
tar -czf mern-app-backup.tar.gz /opt/mern/
tar -czf nginx-config-backup.tar.gz /etc/nginx/
```

These commands create compressed archives of:

- the application directory
- the Nginx configuration directory

Inspect the archives:

```bash
tar -tzf mern-app-backup.tar.gz
tar -tzf nginx-config-backup.tar.gz
```

This allows you to confirm the archive contains the right data before restoring it. In production, you should never blindly extract an archive without checking it.

---

### Part 7: Reflection Questions

#### 1. What is the difference between `tar` and `gzip`?

`tar` packages multiple files into one archive. `gzip` compresses a file to reduce its size.

They are often used together:

```bash
tar -czf archive.tar.gz files/
```

This creates a tar archive and compresses it with gzip.

#### 2. Why is `tar -tzf archive.tar.gz` useful before extraction?

Because it lets you inspect the archive contents without unpacking it. This helps prevent mistakes such as overwriting the wrong files or extracting malicious content.

#### 3. Why should a production backup not rely only on app files?

Because a complete recovery plan needs more than just application code. It may also require:

- database backups
- Nginx config
- environment files
- certificates
- logs
- secret/configuration data

Without these, the app may not run correctly after failure.

#### 4. What is the advantage of timestamping backup filenames?

It prevents overwriting previous backups and makes it easier to restore a particular version. Example:

```bash
mern-backup-2026-08-12_14-30-00.tar.gz
```

#### 5. Why should backup restores be tested regularly?

Because a backup is only useful if it actually works. Testing restores verifies that the archive is valid, complete, and recoverable in a real disaster scenario.

---

### Bonus Challenge

Create a shell script that:

- Creates a timestamped backup of `/opt/mern/`
- Stores it in `/var/backups/mern/`
- Keeps only the last 5 backups
- Prints a success or failure message

Example solution:

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

ls -t "$BACKUP_DIR"/*.tar.gz | tail -n +6 | xargs -r rm -f
```

This script does three important things:

1. Creates a time-stamped archive for versioning.
2. Saves the backup in a dedicated backup directory.
3. Keeps only the newest 5 backups and deletes older ones.

This is a common DevOps pattern for backup retention.

---

### Final Summary

The concepts in this assignment are essential in DevOps and production operations:

- `tar` archives files
- `gzip` compresses files
- `.tar.gz` is a standard deployment artifact format
- inspecting archives before extraction reduces risk
- backups must include application files, config, and database strategy
- timestamping and retention help with disaster recovery
- tested restores are the difference between a backup and a real recovery plan

A production environment is not just about writing code—it is also about safely packaging, moving, and restoring the system when needed.
