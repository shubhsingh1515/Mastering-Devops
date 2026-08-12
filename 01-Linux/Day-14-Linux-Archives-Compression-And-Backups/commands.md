# Day 14 — Commands: Linux Archives, Compression & Backups

## Archiving with tar

| Command | Purpose | Example |
|---------|---------|---------|
| `tar -cf archive.tar dir/` | Create a tar archive | `tar -cf app.tar backend/` |
| `tar -xf archive.tar` | Extract a tar archive | `tar -xf app.tar` |
| `tar -tf archive.tar` | List archive contents | `tar -tf app.tar` |
| `tar -czf archive.tar.gz dir/` | Create a gzipped tar archive | `tar -czf app.tar.gz backend/` |
| `tar -xzf archive.tar.gz` | Extract a gzipped tar archive | `tar -xzf app.tar.gz` |
| `tar -tzf archive.tar.gz` | View contents of `.tar.gz` without extracting | `tar -tzf app.tar.gz` |
| `tar -cJf archive.tar.xz dir/` | Create xz-compressed archive | `tar -cJf app.tar.xz backend/` |
| `tar -xJf archive.tar.xz` | Extract xz-compressed archive | `tar -xJf app.tar.xz` |

## Compression with gzip

| Command | Purpose | Example |
|---------|---------|---------|
| `gzip file` | Compress a file with gzip | `gzip app.tar` |
| `gunzip file.gz` | Decompress a gzip file | `gunzip app.tar.gz` |

## ZIP tools

| Command | Purpose | Example |
|---------|---------|---------|
| `zip -r archive.zip dir/` | Create ZIP archive | `zip -r app.zip backend/` |
| `unzip archive.zip` | Extract ZIP archive | `unzip app.zip` |
| `unzip -l archive.zip` | List ZIP archive contents | `unzip -l app.zip` |

## Backup examples

```bash
# Backup app directory
 tar -czf app-backup.tar.gz /opt/mern/

# Backup nginx config
 tar -czf nginx-backup.tar.gz /etc/nginx/

# Restore app files
 tar -xzf app-backup.tar.gz -C /tmp/
```

## Quick Reference

```bash
# 1. Create a tar archive
 tar -cf app.tar backend/

# 2. View archive contents
 tar -tf app.tar

# 3. Create a compressed archive
 tar -czf app.tar.gz backend/

# 4. Extract compressed archive
 tar -xzf app.tar.gz

# 5. Inspect compressed archive before extraction
 tar -tzf app.tar.gz

# 6. Transfer archive to a remote server
 scp app.tar.gz ubuntu@server:/tmp/

# 7. Backup app directory
 tar -czf /var/backups/mern-app-$(date +%F).tar.gz /opt/mern/
```

## Practical Pipeline

```bash
# Create project folder
mkdir -p ~/devops-course/day14/mern-app/{backend,frontend,config,scripts}

# Create sample files
echo "Node backend" > mern-app/backend/server.js
echo "React frontend" > mern-app/frontend/index.html

# Package it
tar -czf mern-app.tar.gz mern-app/

# Inspect it
tar -tzf mern-app.tar.gz

# Extract it to a test directory
mkdir extracted
tar -xzf mern-app.tar.gz -C extracted/
```

## Important Notes

- `tar` is for archiving; it does not compress by itself.
- `gzip` compresses files and is commonly used with `tar`.
- Always inspect archives before extracting.
- Production backups should include app files, config, and database strategy.
- Use timestamps in backup names to avoid overwriting old data.
