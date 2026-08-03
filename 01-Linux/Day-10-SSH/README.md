# Day 10: SSH (Secure Shell) – Remote Server Access & Authentication

## 📅 Phase 1: Linux
- **Topic:** SSH (Secure Shell) – Remote Server Access & Authentication
- **Goal:** Learn how DevOps engineers securely connect to remote Linux servers, use SSH keys, transfer files, and prepare for AWS EC2, GitHub, and production deployments.

---

## 🎯 Learning Objectives

By the end of this lesson, I'll be able to:

- Understand **how SSH works** and why it's secure.
- **Connect securely** to remote Linux servers.
- **Generate and use SSH key pairs**.
- Configure **passwordless authentication**.
- **Copy files securely** with `scp`.
- Understand **authorized_keys** and **known_hosts**.
- Apply SSH in **AWS EC2** and **Git workflows**.
- Follow **SSH security best practices**.

---

## 🤔 Why This Topic Is Important

Imagine I've deployed a MERN application to an Ubuntu server in the cloud.

How do I:
- Restart the backend?
- View logs?
- Update code?
- Configure Nginx?

I don't use a graphical desktop. Instead, I securely connect using SSH:

```bash
ssh ubuntu@203.0.113.10
```

**SSH is one of the most frequently used tools in a DevOps engineer's daily workflow.**

---

## 📖 Theory Explained in Simple Language

### What is SSH?

**SSH (Secure Shell)** is a protocol that lets me securely access and manage a remote computer over a network.

> Unlike older protocols such as **Telnet**, SSH **encrypts** all traffic so that credentials and commands cannot be read by anyone on the network.

### SSH Architecture

```
Your Laptop
     │
  SSH Client
     │
Encrypted Connection (TCP Port 22)
     │
SSH Server (sshd)
     │
Remote Linux Machine
```

### Password Authentication

```
Laptop
 ↓
Username
 ↓
Password
 ↓
Server
```

*Works, but is less secure and harder to automate.*

### SSH Key Authentication

```
Private Key              Public Key
(Your Computer) ────────▶ (Server)
     │
Cryptographic Challenge
     │
No password sent across the network
```

This is the **preferred method** for production servers.

### Generating an SSH Key Pair

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

If I need broad compatibility with older systems:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**Default location:** `~/.ssh/`

**Files created:**
- `id_ed25519` ← **Private key** (never share this!)
- `id_ed25519.pub` ← **Public key** (safe to share)

### File Permissions

| File | Permission | Reason |
|------|-----------|--------|
| Private key | `600` | Only owner can read/write |
| Public key | `644` | Safe to share, read-only for others |

```bash
chmod 600 ~/.ssh/id_ed25519
```

### Connecting to a Server

```bash
ssh ubuntu@203.0.113.10
```

**Format:** `ssh username@hostname`

**Examples:**
```bash
ssh ec2-user@203.0.113.10
ssh ubuntu@example.com
```

### authorized_keys

On the **server**, the file `~/.ssh/authorized_keys` contains the public keys allowed to log in.

```
ssh-ed25519 AAAAC3Nza... user@example.com
```

When my client proves it has the matching private key, access is granted.

### known_hosts

On my **computer**, the file `~/.ssh/known_hosts` stores fingerprints of servers I've connected to before.

This helps me detect **unexpected server identity changes** (which could indicate a man-in-the-middle attack).

### Copy Files Securely with scp

**Copy a file to a server:**
```bash
scp app.js ubuntu@203.0.113.10:/home/ubuntu/
```

**Copy a directory:**
```bash
scp -r backend/ ubuntu@203.0.113.10:/home/ubuntu/
```

**Copy from the server:**
```bash
scp ubuntu@203.0.113.10:/var/log/app.log .
```

### SSH Agent

Start the agent:
```bash
eval "$(ssh-agent -s)"
```

Add my key:
```bash
ssh-add ~/.ssh/id_ed25519
```

The agent keeps my decrypted key in memory so I don't repeatedly enter the passphrase.

### SSH Configuration File

Create or edit `~/.ssh/config`:

```
Host prod-api
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
```

Now connect using:
```bash
ssh prod-api
```

This simplifies managing multiple servers.

---

## 🌍 Real-World MERN Production Example

### Deploying a MERN Backend to AWS

```
Developer Laptop
     │
    SSH
     │
Ubuntu EC2 Server
     │
  Git Pull
     │
 npm install
     │
systemctl restart myapp
     │
Nginx serves requests
```

**Typical deployment session:**
```bash
ssh ubuntu@203.0.113.10
cd /opt/mern/backend
git pull
npm install
sudo systemctl restart myapp
sudo systemctl status myapp
```

---

## 🏗️ Architecture Diagram (ASCII)

```
            Developer Laptop
                  │
          SSH Private Key
                  │
       Encrypted SSH Session
                  │
        Ubuntu Production Server
          │                 │
     authorized_keys     systemd
          │                 │
      Node.js API       Nginx
```

---

## 🛠️ Step-by-Step Practical Implementation

### Step 1 — Check if SSH is installed
```bash
ssh -V
```

### Step 2 — Generate an SSH key
```bash
ssh-keygen -t ed25519
```
Press **Enter** to accept the default location.

### Step 3 — View generated files
```bash
ls -l ~/.ssh
```

### Step 4 — Secure the private key
```bash
chmod 600 ~/.ssh/id_ed25519
```

### Step 5 — Display the public key
```bash
cat ~/.ssh/id_ed25519.pub
```

### Step 6 — Connect to a server
```bash
ssh username@server-ip
```

### Step 7 — Transfer a file
```bash
scp notes.txt username@server-ip:/home/username/
```

---

## 💻 Commands with Explanations

| Command | Purpose |
|---------|---------|
| `ssh -V` | Show SSH version |
| `ssh user@host` | Connect to a remote server |
| `ssh-keygen -t ed25519` | Generate an SSH key pair |
| `scp file user@host:path` | Copy a file to a server |
| `scp -r dir user@host:path` | Copy a directory |
| `ssh-add key` | Add a key to the SSH agent |
| `chmod 600 key` | Secure a private key |
| `cat ~/.ssh/id_ed25519.pub` | Display the public key |

---

## ✅ Best Practices

1. **Use key-based authentication instead of passwords.**
2. **Protect private keys with `600` permissions.**
3. **Use a strong passphrase** for important SSH keys.
4. **Never commit private keys to Git repositories.**
5. **Use different keys** for personal and work environments when appropriate.
6. **Verify server fingerprints** before accepting a first connection.

---

## ❌ Common Mistakes

1. **Sharing private keys.**
2. **Using overly permissive permissions (644) on private keys.**
3. **Logging in as root** when a regular user with sudo is sufficient.
4. **Forgetting to back up important SSH keys securely.**
5. **Ignoring warnings about changed server fingerprints** without investigation.

---

## 💼 Interview Questions

### Beginner

**Q1: What is SSH?**
> **SSH (Secure Shell)** is a network protocol that provides secure, encrypted communication between two computers. It's used to remotely access and manage servers, transfer files, and tunnel connections. It encrypts all traffic so credentials and commands can't be intercepted.

**Q2: What is the default SSH port?**
> **Port 22** (TCP). This is the standard port SSH listens on by default.

**Q3: What is the difference between a public key and a private key?**
> - **Private key** — Stays on my computer, never shared. It's used to prove my identity and decrypt data.
> - **Public key** — Safe to share. It's placed on servers and used to verify my identity and encrypt data.
> They form a **key pair** that works together for authentication.

**Q4: What is scp used for?**
> `scp` (Secure Copy) is used to **copy files securely** between computers over SSH. I can copy files to or from a remote server, and with `-r` I can copy entire directories.

**Q5: Why is SSH preferred over Telnet?**
> **SSH** encrypts all traffic (including passwords and commands), while **Telnet** sends everything in **plaintext**, which anyone on the network can read. SSH also has stronger authentication options (key-based auth) and is the modern standard.

### Intermediate

**Q6: Explain the SSH authentication process.**
> 1. Client connects to the server on port 22
> 2. Server presents its host key (verified against `known_hosts`)
> 3. Client presents its public key (checked against `authorized_keys`)
> 4. Server sends a **challenge** encrypted with the public key
> 5. Client proves it holds the matching **private key** by decrypting the challenge
> 6. If verified, an encrypted session is established
> No password is ever sent over the network.

**Q7: What is the purpose of authorized_keys?**
> `~/.ssh/authorized_keys` on the **server** contains the public keys of all users allowed to log in. When a client connects, the server checks if the client's public key is in this file. If it is, the client can authenticate using its private key.

**Q8: What information is stored in known_hosts?**
> `~/.ssh/known_hosts` on the **client** stores the **host fingerprints** (public keys) of servers I've connected to before. This allows me to detect when a server's identity changes unexpectedly — which could signal a man-in-the-middle attack.

**Q9: Why should private keys have 600 permissions?**
> The `600` permission (rw-------) means **only the owner** can read or modify the key. If the private key is world-readable (like 644), **any user** on the system could read it and impersonate me. SSH itself refuses to use keys with overly permissive permissions.

**Q10: What is the benefit of an SSH config file?**
> The `~/.ssh/config` file lets me define **aliases** for servers with their hostname, username, and key. Instead of typing `ssh ubuntu@203.0.113.10 -i ~/.ssh/id_ed25519`, I can just type `ssh prod-api`. This simplifies managing multiple servers.

### Advanced

**Q11: Design a secure SSH strategy for a production MERN deployment.**
> A secure strategy includes:
> 1. **Key-based authentication only** — Disable password login
> 2. **Dedicated service users** — No root login, use a regular user with sudo
> 3. **Restrict SSH access** — Limit to specific users and IP ranges
> 4. **Private keys with 600 permissions** and strong passphrases
> 5. **Protect the server** with a firewall (only port 22 open from trusted IPs)
> 6. **Use different keys** for different environments (dev, staging, prod)
> 7. **Monitor SSH logs** (`/var/log/auth.log`) for suspicious activity

**Q12: How would you disable password authentication after configuring SSH keys?**
> In `/etc/ssh/sshd_config`, set:
> ```
> PasswordAuthentication no
> ```
> Then restart SSH:
> ```bash
> sudo systemctl restart ssh
> ```
> **Important:** Always verify key-based login works **before** disabling passwords, or I'll lock myself out!

**Q13: Explain the role of the SSH agent.**
> The **SSH agent** is a background process that keeps decrypted SSH keys in memory. When I use a key with a passphrase, the agent decrypts it once and stores it, so I don't have to enter the passphrase repeatedly. This is especially useful for automation and for forwarding keys during multi-hop SSH connections.

**Q14: How would you securely manage SSH access for multiple DevOps engineers?**
> Options include:
> 1. **Individual keys** — Each engineer gets their own key (no shared keys)
> 2. **Centralized management** — Tools like Vault, or a bastion/SSH jump host
> 3. **Key rotation** — Regularly rotate keys and revoke access when engineers leave
> 4. **Audit logs** — Monitor who connects and when
> 5. **Role-based access** — Restrict what each user can do (least privilege)
> 6. **SSH CA** — Use a certificate authority to sign and revoke keys centrally

**Q15: Describe a secure deployment workflow using SSH, Git, and systemd.**
> 1. Developer pushes code to Git (using SSH for Git auth)
> 2. On the server, pull the latest code: `git pull`
> 3. Install dependencies: `npm install`
> 4. Restart the service: `sudo systemctl restart myapp`
> 5. Verify: `sudo systemctl status myapp` and `ss -tuln`
> 6. This workflow can be automated with CI/CD (Jenkins, GitHub Actions) using SSH keys for deployment

---

## 📝 Quiz (10 Questions)

Let me test my understanding:

**1. What does SSH stand for?**
<details>
<summary>Answer</summary>

**Secure Shell** — A protocol for secure remote access to computers.

</details>

**2. What is the default SSH port?**
<details>
<summary>Answer</summary>

**22** (TCP)

</details>

**3. Which command generates an Ed25519 SSH key?**
<details>
<summary>Answer</summary>

`ssh-keygen -t ed25519`

</details>

**4. Which file stores authorized public keys on the server?**
<details>
<summary>Answer</summary>

`~/.ssh/authorized_keys`

</details>

**5. Which file stores known server fingerprints on the client?**
<details>
<summary>Answer</summary>

`~/.ssh/known_hosts`

</details>

**6. Which command copies files over SSH?**
<details>
<summary>Answer</summary>

`scp` — For example, `scp file user@host:/path/`

</details>

**7. What permission should a private key have?**
<details>
<summary>Answer</summary>

**600** (rw-------) — Only the owner can read/write.

</details>

**8. Which command adds a key to the SSH agent?**
<details>
<summary>Answer</summary>

`ssh-add ~/.ssh/id_ed25519`

</details>

**9. Why should you never share a private key?**
<details>
<summary>Answer</summary>

It grants access to any system that trusts the corresponding public key. Sharing it means anyone holding it can impersonate me.

</details>

**10. Why is key-based authentication preferred?**
<details>
<summary>Answer</summary>

It's more secure than password-based authentication (no password travels over the network) and better suited for automation.

</details>

---

## 👨‍💻 Practical Exercise

### Build Your SSH Workspace

Create the following structure:
```
~/devops-labs/day10/
├── ssh-notes.md
├── deployment-checklist.md
└── config-example.txt
```

**In `config-example.txt`, write a sample SSH config:**
```
Host production
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
```

**Then:**
1. Generate an SSH key pair (if I don't already have one)
2. Verify the private key permissions
3. View my public key
4. Explain (in `ssh-notes.md`) the difference between `authorized_keys` and `known_hosts`

---

## 🧪 Hands-on Lab

### Lab: Simulated First-Time Server Access

Imagine I've received a new Ubuntu server.

**My checklist:**

1. Verify SSH is installed locally
2. Generate an Ed25519 key pair
3. Confirm private key permissions
4. Create an SSH config entry named `staging`

**Write the commands I would use to:**
- Connect to the server
- Copy my backend project
- Restart the application service
- Verify the service status

```bash
# 1. Verify SSH
ssh -V

# 2. Generate key
ssh-keygen -t ed25519

# 3. Confirm permissions
ls -l ~/.ssh/id_ed25519

# 4. Create SSH config entry
# Add to ~/.ssh/config:
# Host staging
#     HostName <server-ip>
#     User ubuntu
#     IdentityFile ~/.ssh/id_ed25519

# 5. Connect
ssh staging

# 6. Copy backend project
scp -r backend/ staging:/home/ubuntu/

# 7. Restart service
sudo systemctl restart myapp

# 8. Verify status
sudo systemctl status myapp
```

---

## 📋 Mini Assignment

### Remote Deployment Runbook for a MERN Application

I'll create a runbook covering:

**1. SSH connection steps**
```bash
ssh ubuntu@<server-ip>
```

**2. Secure key management practices**
- Private keys: `600` permissions
- Never share private keys
- Use key-based auth, not passwords
- Use a strong passphrase

**3. File transfer process**
```bash
scp -r backend/ ubuntu@<server-ip>:/home/ubuntu/
```

**4. Deployment commands**
```bash
cd /opt/mern/backend
git pull
npm install
```

**5. Service restart procedure**
```bash
sudo systemctl restart myapp
```

**6. Post-deployment verification**
```bash
sudo systemctl status myapp
ss -tuln | grep 3000
curl -I http://localhost:3000
```

> This runbook will become the basis for future CI/CD automation.

---

## 📚 Recommended Documentation

I should read the manual pages for deeper understanding:

```bash
man ssh          # SSH command reference
man ssh-keygen   # Key generation options
man scp          # Secure copy
man ssh_config   # Client configuration
man sshd_config  # Server configuration
```

---

## 📖 Summary

Today I learned:

✅ **How SSH provides secure remote access** — Encrypted connections over port 22.

✅ **How public/private key authentication works** — Cryptographic challenge without sending passwords.

✅ **How to generate and protect SSH keys** — `ssh-keygen` and `600` permissions.

✅ **How `authorized_keys` and `known_hosts` are used** — Server-side and client-side trust.

✅ **How to transfer files with `scp`** — Secure file copying.

✅ **How SSH fits into real-world MERN deployments** — AWS EC2, Git, and production servers.

### Key Takeaways

```
Generate Key → Secure Key (600) → Add Public Key to Server → Connect via ssh → Deploy
```

SSH is one of the **foundational skills** for cloud engineering, Linux administration, and DevOps. I'll use it repeatedly when working with Git, Docker, AWS, CI/CD, and Kubernetes.

---

## 🔗 Connecting to the Bigger Picture

SSH connects with everything I've learned:

- **Permissions** → Private keys need `600` permissions
- **Users & Groups** → Access remote servers as specific users
- **Networking** → SSH uses port 22
- **Services** → `sshd` is an SSH service managed by systemd
- **Package Managers** → Install SSH tools with `apt install openssh-server`
- **Processes** → SSH sessions create processes on the server

---

*"SSH is the gateway to every server you'll ever manage. Mastering it is the first step to mastering production infrastructure."*
</content>
