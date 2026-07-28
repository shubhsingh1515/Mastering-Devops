# Day 09: Linux Package Managers (apt, dnf, yum)


## 🎯 Learning Objectives

By the end of this lesson, I'll be able to:

- Understand what **package managers** are and why they exist.
- **Install, update, and remove** software using package managers.
- **Search** for available packages in repositories.
- Understand **package repositories** and why they matter.
- Differentiate between **apt, yum, and dnf**.
- Install common **DevOps tools** (Git, Docker, Nginx, Node.js).
- Follow **safe package management practices** in production.

---

## 🤔 Why This Topic Is Important

Imagine I just provisioned a brand new Ubuntu server. It's completely empty — no Git, no Docker, no Nginx, no Node.js.

**Without a package manager, I would have to:**
1. Search the internet for each tool's download page
2. Verify the source is legitimate and not tampered with
3. Manually download the right version for my OS
4. Manually install all dependencies one by one
5. Repeat this for every single tool

**With a package manager, I just run:**
```bash
sudo apt install git nginx docker.io nodejs npm
```

The package manager handles everything — downloading, dependency resolution, and installation — all from **trusted repositories**.

This is why package managers are the foundation of Linux system administration and DevOps.

---

## 📖 Theory Explained in Simple Language

### What is a Package?

A **package** is a compressed archive that contains everything needed to install a piece of software:

| Component | What it is |
|-----------|-----------|
| **Binaries** | The actual executable programs |
| **Configuration templates** | Default config files that I can customize |
| **Libraries** | Shared code that the application depends on |
| **Documentation** | Man pages, README files |
| **Metadata** | Version number, description, dependencies, maintainer info |

> Think of a package as a **professionally wrapped software bundle** — like receiving a product in its original box with all accessories included.

### What is a Package Manager?

A **package manager** is a tool that automates the process of installing, updating, configuring, and removing software packages.

**Key responsibilities:**
- **Download** packages from repositories
- **Resolve dependencies** — automatically install required libraries
- **Install** software to the correct locations
- **Update** to newer versions
- **Remove** software cleanly
- **Verify** installed versions and integrity

> Without a package manager, managing software on hundreds of servers would be extremely difficult and error-prone.

### Package Repositories

A **repository** (or "repo") is a trusted server that hosts a collection of packages.

```
Ubuntu Server
     ↓
    APT
     ↓
Ubuntu Repository (packages.ubuntu.com)
     ↓
 ┌─────┼─────┐
 Git  Nginx  Node.js
```

**Why repositories matter:**
- **Trust** — Packages are signed and verified
- **Consistency** — All packages are tested to work together
- **Updates** — Security patches are distributed through repos
- **Dependencies** — Repository knows which packages depend on each other

> Using official repositories reduces the risk of installing tampered or malicious software.

### apt (Ubuntu/Debian)

**apt** (Advanced Package Tool) is the package manager used on Ubuntu and Debian systems. This is what I'll use most often.

#### 1. Update Package Index
```bash
sudo apt update
```
This refreshes the list of available packages from all configured repositories.

**Important:** This does NOT upgrade installed software — it just updates the package list.

#### 2. Upgrade Installed Packages
```bash
sudo apt upgrade
```
Installs newer versions of already installed packages.

#### 3. Install Software
```bash
sudo apt install git
sudo apt install nginx
sudo apt install docker.io
```

I can install multiple packages at once:
```bash
sudo apt install git nginx docker.io nodejs npm
```

> **Note:** On Ubuntu, Docker is available as `docker.io` from the default repos. For the latest version, Docker's official repository is preferred.

#### 4. Remove Software

**Keep configuration files:**
```bash
sudo apt remove nginx
```

**Remove everything including configuration:**
```bash
sudo apt purge nginx
```

#### 5. Search Packages
```bash
apt search docker
```

#### 6. Show Package Information
```bash
apt show nginx
```
Useful details include: Version, Description, Dependencies, Homepage, Download size.

#### 7. List Installed Packages
```bash
apt list --installed
```

#### 8. Clean Unused Packages

**Remove unused dependencies:**
```bash
sudo apt autoremove
```

**Clean cached package files:**
```bash
sudo apt clean
```

> These commands help reclaim disk space on production servers.

### yum (Older RHEL/CentOS)

**yum** (Yellowdog Updater Modified) was the package manager for older RHEL and CentOS distributions.

**Examples:**
```bash
sudo yum install nginx
sudo yum update
yum search docker
```

### dnf (Modern RHEL/Fedora)

**dnf** (Dandified YUM) is the modern replacement for yum, used on Fedora, Rocky Linux, and AlmaLinux.

**Examples:**
```bash
sudo dnf install nginx
sudo dnf upgrade
dnf search nodejs
```

### Which Package Manager Should I Use?

| Distribution | Package Manager |
|-------------|----------------|
| Ubuntu | `apt` |
| Debian | `apt` |
| Fedora | `dnf` |
| Rocky Linux | `dnf` |
| AlmaLinux | `dnf` |
| Older CentOS | `yum` |

### Dependency Management

Here's something that amazed me: when I install Nginx with:
```bash
sudo apt install nginx
```

Linux automatically identifies and installs all the libraries Nginx needs (like OpenSSL, PCRE, zlib). This is called **dependency resolution**.

**Without this**, software could fail to run because required components are missing. Imagine downloading a game only to find you need to hunt down 20 different libraries manually — that's what Linux was like before package managers.

---

## 🌍 Real-World Example

### Provisioning a New MERN Server

When I provision a new Ubuntu server for a MERN application, my first steps look like this:

```bash
# 1. Update the package list
sudo apt update

# 2. Install version control
sudo apt install git

# 3. Install web server
sudo apt install nginx

# 4. Install container runtime
sudo apt install docker.io

# 5. Install Node.js and npm
sudo apt install nodejs npm
```

**Then verify everything:**
```bash
git --version
nginx -v
docker --version
node --version
npm --version
```

> Package managers make this workflow **repeatable** across development, staging, and production environments. This consistency is crucial for DevOps.

---

## 🏗️ Architecture Diagram (ASCII)

```
          DevOps Engineer
                │
           apt install
                │
         Package Repository
                │
    ┌───────────┼───────────┐
    │           │           │
   Git       Docker      Nginx
    │           │           │
    └───────────┼───────────┘
                │
        Installed on Server
```

---

## 🛠️ Step-by-Step Practical Implementation

*These examples assume Ubuntu.*

### Step 1 — Refresh Package Metadata
```bash
sudo apt update
```
This connects to all configured repositories and downloads the latest package lists.

### Step 2 — Search for Git
```bash
apt search git
```
Lists all packages related to Git.

### Step 3 — View Package Details
```bash
apt show git
```
Shows version, description, dependencies, homepage, and more.

### Step 4 — Install Git
```bash
sudo apt install git
```
If Git is already installed, this will upgrade it to the latest version.

### Step 5 — Verify Installation
```bash
git --version
```
Expected output: `git version 2.x.x`

### Step 6 — List Installed Packages
```bash
apt list --installed
```
This shows every package installed on the system — there will be hundreds.

### Step 7 — Remove Unused Packages
```bash
sudo apt autoremove
```
Safe to run anytime. Removes packages that were automatically installed as dependencies but are no longer needed.

### Step 8 — Clean Package Cache
```bash
sudo apt clean
```
Removes downloaded `.deb` files from `/var/cache/apt/archives/`, freeing up disk space.

---

## 💻 Commands with Explanations

| Command | Purpose | Explanation |
|---------|---------|-------------|
| `sudo apt update` | Refresh package list | Downloads latest package metadata from repos |
| `sudo apt upgrade` | Upgrade installed packages | Updates all upgradable packages to newest versions |
| `sudo apt install <pkg>` | Install software | Downloads and installs a package + dependencies |
| `sudo apt remove <pkg>` | Remove software (keep config) | Uninstalls but leaves config files |
| `sudo apt purge <pkg>` | Remove software and configs | Complete removal — no traces left |
| `apt search <keyword>` | Search repositories | Finds packages matching the keyword |
| `apt show <pkg>` | Display package info | Shows version, dependencies, description, etc. |
| `apt list --installed` | List installed packages | Shows everything installed on the system |
| `sudo apt autoremove` | Remove unused dependencies | Cleans up orphaned packages safely |
| `sudo apt clean` | Clear package cache | Deletes cached .deb files to free disk space |

---

## ✅ Best Practices

1. **Always run `apt update` before installing new packages** — This ensures I'm getting the latest available versions.

2. **Apply updates during maintenance windows on production servers** — Package upgrades can introduce breaking changes or require service restarts.

3. **Prefer official repositories or vendor-provided repositories** — Official repos are vetted and maintained. Only add third-party repos (like Docker's official repo) when necessary.

4. **Verify software versions after installation** — Always confirm with `--version` to ensure the expected version was installed.

5. **Remove unused packages** — Reduces attack surface and saves disk space on production servers.

6. **Keep production servers minimal** — Install only what's required. Every extra package is a potential vulnerability.

---

## ❌ Common Mistakes

1. **Running `apt upgrade` without understanding what will change** — Always review the list of packages to be upgraded first.

2. **Installing from untrusted sources** — When a trusted repository exists, use it. Avoid downloading `.deb` files from random websites.

3. **Forgetting to verify installations** — Always confirm with `--version` afterward.

4. **Leaving unused packages installed indefinitely** — Unused packages consume disk space and increase the attack surface.

5. **Confusing `apt update` with `apt upgrade`** — This is one of the most common mistakes. `update` refreshes the package list, `upgrade` actually installs newer versions.

---

## 💼 Interview Questions

### Beginner

**Q1: What is a package manager?**
> A **package manager** is a tool that automates installing, updating, configuring, and removing software packages. It handles downloading from repositories, resolving dependencies, and maintaining software consistency across the system. Examples include `apt`, `yum`, and `dnf`.

**Q2: What does `apt update` do?**
> `sudo apt update` refreshes the **local package index** — it downloads the latest list of available packages and versions from all configured repositories. It does **not** install or upgrade any software.

**Q3: What is the difference between `apt update` and `apt upgrade`?**
> - **`apt update`** — Refreshes the package list from repositories (metadata only)
> - **`apt upgrade`** — Actually installs newer versions of already installed packages
> They are typically run together: first `update`, then `upgrade`.

**Q4: How do you install Git on Ubuntu?**
> ```bash
> sudo apt update
> sudo apt install git
> ```

**Q5: How do you search for available packages?**
> ```bash
> apt search <keyword>
> ```
> For example, `apt search docker` lists all Docker-related packages available in the repositories.

### Intermediate

**Q6: What are package repositories?**
> **Package repositories** are trusted servers that host collections of software packages. They provide:
> - **Authenticity** — Packages are signed and verified
> - **Dependency information** — Repository knows which packages need each other
> - **Updates** — Security patches and bug fixes are distributed through repos
> 
> Ubuntu's official repository is at `archive.ubuntu.com`, and I can add third-party repositories like Docker's official repo for newer versions.

**Q7: What is dependency resolution?**
> **Dependency resolution** is the process where the package manager automatically identifies and installs all libraries and packages that a piece of software needs to function. For example, installing Nginx also installs OpenSSL and other required libraries. Without this, software would fail with "missing library" errors.

**Q8: When would you use `apt purge` instead of `apt remove`?**
> - **`apt remove`** — Uninstalls the software but **keeps configuration files**. Use this when I might reinstall later and want to preserve settings.
> - **`apt purge`** — Removes **everything**, including configuration files. Use this when I want a **complete cleanup** — for example, when removing software I no longer need and don't want leftover configs.

**Q9: Why is `apt autoremove` useful?**
> `sudo apt autoremove` removes packages that were automatically installed as **dependencies** but are **no longer needed** by any installed software. This is useful because:
> - It frees up **disk space**
> - It reduces the **attack surface** by removing unnecessary software
> - It keeps the system **clean and minimal**

**Q10: Compare apt, yum, and dnf.**
> | Package Manager | Used By | Status |
> |----------------|---------|--------|
> | **apt** | Ubuntu, Debian | Current standard |
> | **yum** | Older CentOS/RHEL (v7 and earlier) | Legacy — being replaced by dnf |
> | **dnf** | Fedora, Rocky Linux, AlmaLinux, RHEL 8+ | Modern replacement for yum |
>
> All three do the same core tasks (install, update, remove). The syntax is similar but not identical — for example, `apt install` vs `dnf install` vs `yum install`.

### Advanced

**Q11: How would you safely patch production servers?**
> Safe patching workflow:
> 1. **Test in staging first** — Never apply updates directly to production without testing
> 2. **Schedule a maintenance window** — Inform stakeholders about expected downtime
> 3. **Take a backup or snapshot** — So I can roll back if something goes wrong
> 4. **Run `apt update`** — Get the latest package lists
> 5. **Review changes** — Use `apt list --upgradable` to see what will change
> 6. **Apply updates** — `sudo apt upgrade` (or use `--only-upgrade` for specific packages)
> 7. **Test the application** — Verify everything still works
> 8. **Monitor logs** — Watch for issues in the hours following the update

**Q12: Why might a company use vendor-maintained repositories instead of distribution defaults?**
> Companies often use **vendor repositories** (e.g., Docker's official repo, NodeSource for Node.js) because:
> - **Newer versions** — Distribution repos often lag behind (e.g., Ubuntu's Node.js may be several versions old)
> - **Security patches** — Vendors release patches faster
> - **Enterprise support** — Some vendors offer paid support for their repos
> - **Specific configurations** — Vendor repos provide packages optimized for their software
>
> The trade-off is that adding external repos introduces a new trust boundary.

**Q13: How would you standardize package versions across multiple environments?**
> Approaches to standardize versions:
> 1. **Use a local repository mirror** — Host an internal apt repository (e.g., with `aptly` or `reprepro`) that pins specific versions
> 2. **Configuration management** — Use Ansible, Puppet, or Chef to enforce package versions declaratively
> 3. **Docker containers** — Package the application with all dependencies in a container image (most reliable approach)
> 4. **Version pinning** — Use `apt-mark hold <package>` to prevent accidental upgrades
> 5. **Infrastructure as Code** — Define the exact package versions in version-controlled config files

**Q14: What risks exist when installing software from unofficial sources?**
> Risks include:
> - **Malware** — Tampered packages can contain backdoors, keyloggers, or ransomware
> - **Instability** — Unofficial packages may conflict with system libraries
> - **No updates** — No mechanism to receive security patches
> - **Broken dependencies** — May install incompatible library versions
> - **Compliance violations** — Using unapproved software may violate organizational policies
> 
> Always prefer official repositories or well-known vendor repositories (e.g., Docker, NodeSource, Nginx Official).

**Q15: How do package managers support infrastructure automation?**
> Package managers are essential for automation because:
> - **Scriptable** — All commands can be used in shell scripts
> - **Idempotent** — Running `apt install` on an already-installed package is safe
> - **Configuration management** — Tools like Ansible use package manager modules (`apt`, `yum`, `dnf`) to ensure desired state
> - **Dockerfiles** — Docker images are built using package manager commands in `RUN` instructions
> - **CI/CD pipelines** — Package managers install build tools and dependencies in automated pipelines
>
> Example automation with Ansible:
> ```yaml
> - name: Install Nginx
>   apt:
>     name: nginx
>     state: present
> ```

---

## 📝 Quiz (10 Questions)

Let me test my understanding:

**1. Which command refreshes package metadata?**
<details>
<summary>Answer</summary>

`sudo apt update` — Downloads the latest package lists from all configured repositories.

</details>

**2. Which command upgrades installed packages?**
<details>
<summary>Answer</summary>

`sudo apt upgrade` — Installs newer versions of all upgradable packages.

</details>

**3. Which command installs software?**
<details>
<summary>Answer</summary>

`sudo apt install <package-name>` — For example, `sudo apt install nginx`.

</details>

**4. Which command removes unused dependencies?**
<details>
<summary>Answer</summary>

`sudo apt autoremove` — Removes packages that were installed as dependencies but are no longer needed.

</details>

**5. Which package manager is used on Ubuntu?**
<details>
<summary>Answer</summary>

**`apt`** — Advanced Package Tool, the standard for Debian-based distributions.

</details>

**6. Which package manager is used on modern Rocky Linux?**
<details>
<summary>Answer</summary>

**`dnf`** — Dandified YUM, the modern package manager for RHEL-based distributions.

</details>

**7. What does `apt show` display?**
<details>
<summary>Answer</summary>

Package details: version, description, dependencies, homepage, maintainer, download size, and more.

</details>

**8. What is a package repository?**
<details>
<summary>Answer</summary>

A **trusted server** that hosts a collection of software packages, along with metadata about versions and dependencies.

</details>

**9. Which command lists installed packages?**
<details>
<summary>Answer</summary>

`apt list --installed` — Shows every package currently installed on the system.

</details>

**10. Why is dependency resolution important?**
<details>
<summary>Answer</summary>

It **automatically installs required libraries and components** that a package needs to function. Without it, software would fail with missing dependency errors.

</details>

---

## 👨‍💻 Coding / Scripting Exercise

### package_audit.sh

I created a script that checks which common DevOps tools are installed:

```bash
#!/bin/bash

echo "=== Git ==="
git --version 2>/dev/null || echo "Git not installed"

echo
echo "=== Node.js ==="
node --version 2>/dev/null || echo "Node.js not installed"

echo
echo "=== Docker ==="
docker --version 2>/dev/null || echo "Docker not installed"

echo
echo "=== Nginx ==="
nginx -v 2>&1 || echo "Nginx not installed"
```

**To use it:**
```bash
chmod +x package_audit.sh
./package_audit.sh
```

**What I learned:**
- `2>/dev/null` — Redirects error output to null (hides "command not found" errors)
- `||` — Logical OR: if the first command fails, run the second
- This script is useful for quickly assessing what's installed on any server

**Sample output:**
```
=== Git ===
git version 2.34.1

=== Node.js ===
v18.16.0

=== Docker ===
Docker version 24.0.5

=== Nginx ===
nginx version: nginx/1.24.0
```

---

## 🧪 Hands-on Lab

### Lab: Prepare a Fresh Ubuntu Server

**Objective:** Become comfortable discovering and installing software using the package manager.

**Steps I completed:**

1. **Refreshed package metadata:**
   ```bash
   sudo apt update
   ```

2. **Searched for packages:**
   ```bash
   apt search git | head -10
   apt search nginx | head -10
   apt search nodejs | head -10
   ```

3. **Viewed package details for Git:**
   ```bash
   apt show git
   ```
   I noted: version, description, dependencies, homepage.

4. **Installed Git if missing:**
   ```bash
   sudo apt install git
   ```

5. **Verified the installation:**
   ```bash
   git --version
   ```

6. **Listed installed packages and identified five:**
   ```bash
   apt list --installed | head -20
   ```
   I found packages like: `bash`, `coreutils`, `systemd`, `openssh-server`, `curl`.

**My recorded findings:**

| Task | Command | Result |
|------|---------|--------|
| Refresh metadata | `sudo apt update` | Packages updated |
| Search Git | `apt search git` | Multiple git-related packages found |
| Git details | `apt show git` | Version, deps, homepage shown |
| Install Git | `sudo apt install git` | Installed successfully |
| Verify Git | `git --version` | `git version 2.x.x` |
| List installed | `apt list --installed` | Hundreds of packages listed |

---

## 📋 Mini Assignment

### Server Provisioning Checklist for a New Ubuntu Machine (MERN Application)

When provisioning a new Ubuntu server for a MERN stack application, I follow this checklist:

#### Step 1: Update Package Metadata
```bash
sudo apt update
```
**Why:** Fresh servers have outdated package lists. This ensures I install the latest available versions.

#### Step 2: Install Git
```bash
sudo apt install git
```
**Why:** Version control is essential. I need Git to clone the application repository from GitHub/GitLab and manage code deployments.

#### Step 3: Install Node.js and npm
```bash
sudo apt install nodejs npm
```
**Why:** Node.js is the runtime for the backend API. npm is the package manager for installing Express.js, Mongoose, and other dependencies.

> For production, I might use NodeSource's official repository for a newer Node.js version.

#### Step 4: Install Nginx
```bash
sudo apt install nginx
```
**Why:** Nginx acts as a **reverse proxy** — it forwards public traffic to the Node.js backend, handles SSL termination, and serves static frontend files.

#### Step 5: Install Docker
```bash
sudo apt install docker.io
```
**Why:** Docker allows me to containerize the application for consistent deployments across environments. In production, I'd add Docker's official repository for the latest version.

#### Step 6: Verify All Versions
```bash
git --version
node --version
npm --version
nginx -v
docker --version
```
**Why:** Confirms each tool was installed correctly and I know which versions are running in production.

#### Step 7: Clean Up
```bash
sudo apt autoremove
sudo apt clean
```
**Why:** Removes unnecessary packages and cached files, keeping the server minimal and reducing the attack surface.

#### Step 8: Enable Services at Boot
```bash
sudo systemctl enable nginx
sudo systemctl enable docker
```
**Why:** Ensures critical services start automatically if the server reboots.

---

## 📚 Recommended Documentation

I should read the manual pages for deeper understanding:

```bash
man apt           # APT package manager documentation
man apt-get       # Older APT interface (still useful to know)
man dnf           # DNF package manager (RHEL-based systems)
man yum           # Legacy YUM package manager
```

**Additional resources:**
- Ubuntu Server Guide — Package management section
- Debian APT User's Guide — Comprehensive reference
- Docker's official installation documentation

```bash
apt --help        # Quick reference for apt commands
dnf --help        # Quick reference for dnf commands
```

---

## 📖 Summary

Today I learned:

✅ **What Linux packages are** — Bundled software containing binaries, configs, libraries, and metadata.

✅ **How package managers work** — They download, resolve dependencies, install, update, and remove software automatically.

✅ **The difference between apt, dnf, and yum** — `apt` for Ubuntu/Debian, `dnf` for modern RHEL systems, `yum` for legacy systems.

✅ **How to install, update, search, and remove software** — The essential commands I'll use daily.

✅ **How dependency resolution simplifies software installation** — The package manager automatically handles required libraries.

✅ **Best practices for maintaining production servers** — Update lists first, test changes, keep systems minimal.

### Key Takeaways

```
New Server → apt update → apt install → Verify → Clean Up → Ready
```

Package managers are the **foundation for provisioning Linux systems**. They will be heavily used when I:
- Automate server setup with **Ansible**
- Create **Docker images** with `apt` commands in Dockerfiles
- Build **CI/CD pipelines** that install dependencies
- Manage servers at **scale** with infrastructure as code

---

## 🔗 Connecting to the Bigger Picture

Package managers connect with everything I've learned so far:

- **Services** → Installed packages often register as services (`systemctl enable nginx`)
- **Permissions** → Installed files have specific permissions (binaries are 755, configs are 644)
- **Processes** → Running services are processes managed by systemd
- **Networking** → Services listen on ports after installation
- **Users** → Some packages create service accounts during installation

---

*"Package managers transform an empty Linux server from a blank slate into a production-ready machine in minutes. They are the foundation of infrastructure automation."*
