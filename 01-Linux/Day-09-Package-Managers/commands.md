# Day 09 — Commands: Linux Package Managers

## apt (Ubuntu/Debian)

| Command | Purpose | Example |
|---------|---------|---------|
| `sudo apt update` | Refresh package list from repositories | `sudo apt update` |
| `sudo apt upgrade` | Upgrade all installed packages | `sudo apt upgrade` |
| `sudo apt install <pkg>` | Install a package | `sudo apt install nginx` |
| `sudo apt remove <pkg>` | Remove package (keep configs) | `sudo apt remove nginx` |
| `sudo apt purge <pkg>` | Remove package and configs | `sudo apt purge nginx` |
| `apt search <keyword>` | Search for packages | `apt search docker` |
| `apt show <pkg>` | Show package details | `apt show git` |
| `apt list --installed` | List all installed packages | `apt list --installed` |
| `apt list --upgradable` | List packages that can be upgraded | `apt list --upgradable` |
| `sudo apt autoremove` | Remove unused dependencies | `sudo apt autoremove` |
| `sudo apt clean` | Clear cached package files | `sudo apt clean` |

## dnf (Fedora, Rocky Linux, AlmaLinux)

| Command | Purpose | Example |
|---------|---------|---------|
| `sudo dnf install <pkg>` | Install a package | `sudo dnf install nginx` |
| `sudo dnf upgrade` | Upgrade all packages | `sudo dnf upgrade` |
| `sudo dnf remove <pkg>` | Remove a package | `sudo dnf remove nginx` |
| `dnf search <keyword>` | Search for packages | `dnf search nodejs` |
| `dnf info <pkg>` | Show package details | `dnf info git` |
| `dnf list installed` | List installed packages | `dnf list installed` |
| `sudo dnf autoremove` | Remove unused dependencies | `sudo dnf autoremove` |

## yum (Older CentOS/RHEL)

| Command | Purpose | Example |
|---------|---------|---------|
| `sudo yum install <pkg>` | Install a package | `sudo yum install nginx` |
| `sudo yum update` | Update all packages | `sudo yum update` |
| `sudo yum remove <pkg>` | Remove a package | `sudo yum remove nginx` |
| `yum search <keyword>` | Search for packages | `yum search docker` |
| `yum info <pkg>` | Show package details | `yum info git` |

## Quick Reference

```bash
# Standard workflow for installing software
sudo apt update                    # 1. Refresh package list
sudo apt install <package-name>    # 2. Install the package
<command> --version                # 3. Verify installation

# Standard workflow for daily server maintenance
sudo apt update                    # Refresh list
sudo apt upgrade                   # Apply updates
sudo apt autoremove                # Clean unused deps
sudo apt clean                     # Clear cache
