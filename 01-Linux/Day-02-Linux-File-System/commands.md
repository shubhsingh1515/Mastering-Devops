# Linux Commands (Day 2 — Linux File System)

## Navigation Commands
| Command | Explanation |
|---------|-------------|
| `cd /` | Go to root directory |
| `cd ~` | Go to home directory |
| `cd ..` | Move up one directory |
| `pwd` | Print current working directory |

## Listing & Inspection
| Command | Explanation |
|---------|-------------|
| `ls` | List files/directories |
| `ls -l` | Long listing with permissions and file types |
| `ls -R` | Recursive listing (shows subdirectories) |
| `ls -a` | List all files including hidden ones |

## File Operations
| Command | Explanation |
|---------|-------------|
| `touch file` | Create an empty file |
| `mkdir dir` | Create a directory |
| `mkdir -p path/to/dir` | Create nested directories |

## Links
| Command | Explanation |
|---------|-------------|
| `ln -s src dest` | Create a symbolic (soft) link |
| `ln src dest` | Create a hard link |

## System Inspection
| Command | Explanation |
|---------|-------------|
| `cat /proc/cpuinfo` | Display CPU information |
| `cat /proc/meminfo` | Display memory information |

## File Type Detection
The first character of `ls -l` output indicates file type:
- `-` Regular file
- `d` Directory
- `l` Symbolic link
- `c` Character device
- `b` Block device

## Quick Reminders
- `/` is the root directory (top-most).
- `/root` is the home directory for the root user (different from `/`).
- `/var/log` is the primary location for system and application logs.
- `/etc` stores system configuration files.
- `/tmp` is for temporary files only — never store important data there.

