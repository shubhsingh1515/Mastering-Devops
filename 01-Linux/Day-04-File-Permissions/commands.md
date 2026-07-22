# Linux Commands (Day 4 — File Permissions & Ownership)

## Viewing Permissions
| Command | Purpose |
|---------|---------|
| `ls -l` | Show permissions and ownership for files/directories |
| `ls -ld dir` | Show permissions of a directory itself (not its contents) |

## Changing Permissions (chmod)
| Command | Purpose |
|---------|---------|
| `chmod +x file` | Add execute permission for all |
| `chmod -w file` | Remove write permission for all |
| `chmod u+x file` | Add execute for owner only |
| `chmod g+w file` | Add write for group only |
| `chmod o-r file` | Remove read for others |
| `chmod 755 file` | Set rwxr-xr-x (numeric) |
| `chmod 644 file` | Set rw-r--r-- (numeric) |
| `chmod 600 file` | Set rw------- (numeric) |
| `chmod 700 file` | Set rwx------ (numeric) |
| `chmod 775 dir` | Set rwxrwxr-x for shared directories |

## Changing Ownership
| Command | Purpose |
|---------|---------|
| `chown user file` | Change file owner |
| `chown user:group file` | Change owner and group |
| `chown :group file` | Change group only |
| `chgrp group file` | Change group (alternative) |
| `sudo chown -R user dir` | Recursively change ownership |

## Permission Numeric Reference
| Octal | Owner | Group | Others | Description |
|-------|-------|-------|--------|-------------|
| 7 | rwx | — | — | Read, write, execute |
| 6 | rw- | — | — | Read, write |
| 5 | r-x | — | — | Read, execute |
| 4 | r-- | — | — | Read only |
| 0 | --- | — | — | No permissions |

## Common Permission Combinations
| Octal | Use Case |
|-------|----------|
| `755` | Executable scripts, directories |
| `644` | Source code, config files |
| `600` | Private keys, `.env`, secrets |
| `700` | Private scripts (owner only) |
| `775` | Shared directories with group write |
| `444` | Read-only files |
| `555` | Read + execute only |

## Quick Examples

**Make a script executable:**
```bash
chmod +x deploy.sh
```

**Secure a .env file:**
```bash
chmod 600 .env
```

**Set proper directory permissions:**
```bash
chmod 755 /var/www/mern-app
```

**Change ownership for deployment:**
```bash
sudo chown -R www-data:www-data /var/www/mern-app
```

**Verify permissions:**
```bash
ls -l
