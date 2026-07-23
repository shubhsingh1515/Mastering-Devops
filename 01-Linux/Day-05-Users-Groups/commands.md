# Linux Commands (Day 5 — Users & Groups)

## User Information
| Command | Purpose |
|---------|---------|
| `whoami` | Show current username |
| `id` | Show UID, GID, and group memberships |
| `who` | Show currently logged-in users |
| `groups` | List group memberships for current user |
| `groups user` | List group memberships for a specific user |

## User Management
| Command | Purpose |
|---------|---------|
| `sudo useradd user` | Create a user (no home directory) |
| `sudo useradd -m user` | Create a user with a home directory |
| `sudo passwd user` | Set or change a user's password |
| `sudo userdel user` | Delete a user account |
| `sudo userdel -r user` | Delete a user and their home directory |

## Group Management
| Command | Purpose |
|---------|---------|
| `sudo groupadd group` | Create a new group |
| `sudo groupdel group` | Delete a group |
| `sudo usermod -aG group user` | Add user to a secondary group (append) |
| `sudo gpasswd -d user group` | Remove user from a group |

## Privilege Escalation
| Command | Purpose |
|---------|---------|
| `sudo command` | Run a command with root privileges |
| `sudo -i` | Start a root login shell |
| `sudo -u user command` | Run a command as a specific user |

## Viewing System Users & Groups
| Command | Purpose |
|---------|---------|
| `cat /etc/passwd` | List all user accounts on the system |
| `cat /etc/group` | List all groups on the system |
| `cat /etc/shadow` | View encrypted passwords (requires root) |

## Quick Examples

**Check your current user:**
```bash
whoami
```

**View your UID and groups:**
```bash
id
```

**Create a new developer user:**
```bash
sudo useradd -m devuser
sudo passwd devuser
```

**Create a group and add the user:**
```bash
sudo groupadd devteam
sudo usermod -aG devteam devuser
```

**Verify group membership:**
```bash
groups devuser
```

**Run a command as another user:**
```bash
sudo -u devuser whoami
