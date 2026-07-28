# Day 09 — Interview Questions: Linux Package Managers

## Beginner

### Q1: What is a package manager?
A **package manager** is a tool that automates the process of installing, updating, configuring, and removing software packages on a Linux system. It handles downloading from repositories, resolving dependencies, and maintaining consistency. Examples include `apt` (Ubuntu/Debian), `dnf` (Fedora/Rocky Linux), and `yum` (older CentOS).

### Q2: What does `apt update` do?
`sudo apt update` refreshes the **local package index** — it connects to all configured repositories and downloads the latest list of available packages and their versions. It does **not** install or upgrade any software. This is the first step before installing anything to ensure I'm getting the latest versions.

### Q3: What is the difference between `apt update` and `apt upgrade`?
- **`apt update`** — Refreshes the package metadata (the list of what's available)
- **`apt upgrade`** — Actually installs newer versions of already installed packages

They are typically run in sequence. Think of `update` as checking the menu and `upgrade` as ordering the food.

### Q4: How do you install Git on Ubuntu?
```bash
sudo apt update
sudo apt install git
```
First I refresh the package list, then I install Git. After installation, I verify with `git --version`.

### Q5: How do you search for available packages?
```bash
apt search <keyword>
```
For example, `apt search docker` returns all packages related to Docker. I can also use `apt show <package>` to see detailed information about a specific package.

---

## Intermediate

### Q6: What are package repositories?
**Package repositories** are trusted servers that host collections of software packages. They provide:
- **Authentication** — Packages are signed with GPG keys to verify authenticity
- **Metadata** — Information about versions, dependencies, and conflicts
- **Updates** — Security patches and bug fixes distributed through repos

Ubuntu's main repositories are at `archive.ubuntu.com`. Additional repositories like Docker's official repo can be added for newer versions.

### Q7: What is dependency resolution?
**Dependency resolution** is the process where a package manager automatically identifies, downloads, and installs all libraries and packages that a piece of software requires to function. For example, installing Nginx also installs OpenSSL, PCRE, and zlib if they aren't already present. This prevents "missing library" errors that would otherwise make software unusable.

### Q8: When would you use `apt purge` instead of `apt remove`?
- **`apt remove`** — Removes the software binaries but **keeps configuration files**. Use this when I might reinstall later and want to preserve my custom settings.
- **`apt purge`** — Removes **everything**, including configuration files. Use this when I want to completely remove a piece of software and all traces of it — for example, when cleaning up after a failed configuration experiment.

### Q9: Why is `apt autoremove` useful?
`sudo apt autoremove` removes packages that were automatically installed as **dependencies** but are **no longer required** by any currently installed software. This is useful because:
1. It **frees up disk space** — Old libraries can accumulate over time
2. It **reduces the attack surface** — Fewer installed packages means fewer potential vulnerabilities
3. It **keeps the system clean** — Prevents unused packages from cluttering the system

### Q10: Compare apt, yum, and dnf.
| Manager | Used By | Status | Example Command |
|---------|---------|--------|----------------|
| **apt** | Ubuntu, Debian | Current standard | `apt install nginx` |
| **yum** | Older CentOS/RHEL 7- | Legacy | `yum install nginx` |
| **dnf** | Fedora, Rocky Linux, RHEL 8+ | Modern replacement | `dnf install nginx` |

All three serve the same purpose — installing, updating, and removing software. The syntax is similar but commands differ slightly (e.g., `apt update` vs `yum update` vs `dnf upgrade`).

---

## Advanced

### Q11: How would you safely patch production servers?
Safe patching requires a disciplined approach:
1. **Have a rollback plan** — Take snapshots or backups before making changes
2. **Test in a staging environment** — Never apply patches directly to production without testing
3. **Schedule a maintenance window** — Communicate expected downtime to stakeholders
4. **Review what will change** — `apt list --upgradable` shows which packages will be updated
5. **Update the package list** — `sudo apt update`
6. **Apply updates** — `sudo apt upgrade` (or selectively upgrade specific packages)
7. **Test the application** — Verify functionality after the update
8. **Monitor** — Watch logs and metrics for issues in the hours after patching

### Q12: Why might a company use vendor-maintained repositories instead of distribution defaults?
Companies often prefer vendor repositories (e.g., Docker's official repo, NodeSource for Node.js, Nginx's official repo) because:
- **Newer software versions** — Distribution repos often lag significantly behind upstream releases
- **Faster security patches** — Vendors release critical fixes more quickly
- **Enterprise support** — Some vendors offer commercial support agreements
- **Optimized packages** — Custom build flags and configurations specific to the software
- **Consistency** — Same versions across multiple distributions (e.g., Docker on Ubuntu and RHEL)

The trade-off is introducing an additional trust boundary and maintenance overhead for repo keys.

### Q13: How would you standardize package versions across multiple environments?
To ensure consistency across dev, staging, and production:
1. **Use a local repository mirror** — Host an internal apt repository (with tools like `aptly`) that pins specific package versions
2. **Configuration management** — Use Ansible to declare exact package versions: `apt: name=nginx version=1.24.0 state=present`
3. **Containerization** — Use Docker to package the application with all dependencies, eliminating host-level variation
4. **Version pinning** — `apt-mark hold <package>` prevents accidental upgrades
5. **Infrastructure as Code** — Define all package versions in version-controlled config files
6. **Golden images** — Create base AMIs/VM images with pre-installed packages at specific versions

### Q14: What risks exist when installing software from unofficial sources?
Risks include:
1. **Malware and backdoors** — Tampered packages can contain malicious code
2. **Instability** — Unofficial packages may conflict with system libraries
3. **No security updates** — No mechanism to receive critical patches
4. **Broken dependencies** — May install incompatible library versions that break other software
5. **Compliance violations** — Using unapproved software may breach organizational policies
6. **No accountability** — If something breaks, there's no vendor to support

Always prefer official distribution repositories or well-known vendor repositories. If an external repo is necessary, verify its GPG key and maintain it properly.

### Q15: How do package managers support infrastructure automation?
Package managers are fundamental to automation because:
1. **Scriptable** — All commands can be used in shell scripts and CI/CD pipelines
2. **Idempotent** — Running `apt install nginx` on an already-installed package is safe and makes no changes
3. **Ansible/Chef/Puppet modules** — Configuration management tools have dedicated modules for package management
4. **Dockerfiles** — Docker images are built using package manager commands in `RUN` instructions:
   ```dockerfile
   RUN apt update && apt install -y nginx
   ```
5. **Immutable infrastructure** — Packer uses package managers to build server images
6. **Desired state configuration** — Tools like Ansible declare what packages should be installed, and the tool handles the rest:
   ```yaml
   - name: Install required packages
     apt:
       name: "{{ packages }}"
       state: present
