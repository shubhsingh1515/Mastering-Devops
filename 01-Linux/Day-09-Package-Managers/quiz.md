# Day 09 — Quiz: Linux Package Managers

## Questions

1. **Which command refreshes package metadata?**

   <details>
   <summary>Answer</summary>

   `sudo apt update` — Downloads the latest package lists from all configured repositories without upgrading any software.
   </details>

2. **Which command upgrades installed packages?**

   <details>
   <summary>Answer</summary>

   `sudo apt upgrade` — Installs newer versions of all upgradable packages currently installed on the system.
   </details>

3. **Which command installs software on Ubuntu?**

   <details>
   <summary>Answer</summary>

   `sudo apt install <package-name>` — For example, `sudo apt install nginx` installs the Nginx web server along with its dependencies.
   </details>

4. **Which command removes unused dependencies?**

   <details>
   <summary>Answer</summary>

   `sudo apt autoremove` — Removes packages that were installed automatically as dependencies but are no longer needed by any installed software.
   </details>

5. **Which package manager is used on Ubuntu?**

   <details>
   <summary>Answer</summary>

   **`apt`** (Advanced Package Tool) — The standard package manager for Debian-based distributions like Ubuntu.
   </details>

6. **Which package manager is used on modern Rocky Linux?**

   <details>
   <summary>Answer</summary>

   **`dnf`** (Dandified YUM) — The modern package manager for RHEL-based distributions like Rocky Linux, AlmaLinux, and Fedora.
   </details>

7. **What does `apt show <package>` display?**

   <details>
   <summary>Answer</summary>

   Detailed package information including: version number, description, dependencies, maintainer, homepage URL, download size, and installation size.
   </details>

8. **What is a package repository?**

   <details>
   <summary>Answer</summary>

   A **trusted server** that hosts a collection of software packages along with metadata about versions, dependencies, and signatures. Examples include Ubuntu's `archive.ubuntu.com` and Docker's official APT repository.
   </details>

9. **Which command lists installed packages?**

   <details>
   <summary>Answer</summary>

   `apt list --installed` — Shows every package currently installed on the system. On a typical server, this can be hundreds of packages.
   </details>

10. **Why is dependency resolution important?**

    <details>
    <summary>Answer</summary>

    It **automatically identifies and installs all required libraries and components** that a package needs to function. Without it, software would fail to run with "missing library" errors, and administrators would have to manually track and install dependencies.
    </details>

## Bonus Questions

11. **What is the difference between `apt remove` and `apt purge`?**

    <details>
    <summary>Answer</summary>

    `apt remove` uninstalls the software but **leaves configuration files** behind. `apt purge` removes **everything**, including configuration files. Use `purge` for a complete cleanup.
    </details>

12. **What is the difference between `apt update` and `apt upgrade`?**

    <details>
    <summary>Answer</summary>

    `apt update` refreshes the **package index** (metadata about available packages). `apt upgrade` actually **installs newer versions** of already installed packages. `update` must be run before `upgrade`.
    </details>

13. **Which package manager is the legacy version for RHEL/CentOS?**

    <details>
    <summary>Answer</summary>

    **`yum`** — Used on older CentOS/RHEL 7 and earlier. It has been replaced by `dnf` in modern versions.
    </details>

14. **What does `apt list --upgradable` show?**

    <details>
    <summary>Answer</summary>

    Lists all installed packages that have newer versions available in the repositories. This helps review what will change before running `apt upgrade`.
    </details>

15. **How can you search for a package using apt?**

    <details>
    <summary>Answer</summary>

    `apt search <keyword>` — For example, `apt search docker` returns all packages related to Docker available in the configured repositories.
    </details>
