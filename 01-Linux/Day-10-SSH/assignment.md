# Day 10 — Assignment: SSH (Secure Shell)

## Objective
Apply what you've learned about SSH to build a secure remote access workflow, practice key management, and create a deployment runbook for a MERN application.

---

## Part 1: Build Your SSH Workspace

Create the following directory structure:

```
~/devops-labs/day10/
├── ssh-notes.md
├── deployment-checklist.md
└── config-example.txt
```

### Step 1: Create the directory structure
```bash
mkdir -p ~/devops-labs/day10
```

### Step 2: Generate an SSH key pair (if you don't have one)
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
Press Enter to accept the default location `~/.ssh/id_ed25519`.

### Step 3: Verify the private key permissions
```bash
ls -l ~/.ssh/id_ed25519
```
Expected: `-rw-------` (600)

If not, fix it:
```bash
chmod 600 ~/.ssh/id_ed25519
```

### Step 4: View your public key
```bash
cat ~/.ssh/id_ed25519.pub
```

### Step 5: Create `config-example.txt`
Write a sample SSH config:

```
Host production
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519

Host staging
    HostName 198.51.100.20
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
```

---

## Part 2: SSH Notes

In `ssh-notes.md`, explain the difference between `authorized_keys` and `known_hosts`.

**Prompt for your answer:**

| File | Location | Purpose |
|------|----------|---------|
| `authorized_keys` | On the server (`~/.ssh/authorized_keys`) | Lists public keys allowed to log in — the server trusts these clients |
| `known_hosts` | On the client (`~/.ssh/known_hosts`) | Stores server fingerprints — the client trusts these servers |

Write a paragraph explaining why both are needed for secure SSH communication.

---

## Part 3: Hands-on Lab — Simulated First-Time Server Access

Imagine you've received a new Ubuntu server with IP `203.0.113.50`.

### Your checklist:

1. **Verify SSH is installed locally**
   ```bash
   ssh -V
   ```

2. **Generate an Ed25519 key pair**
   ```bash
   ssh-keygen -t ed25519
   ```

3. **Confirm private key permissions**
   ```bash
   ls -l ~/.ssh/id_ed25519
   ```

4. **Create an SSH config entry named `staging`**
   ```
   Host staging
       HostName 203.0.113.50
       User ubuntu
       IdentityFile ~/.ssh/id_ed25519
   ```

### Write the commands you would use to:

5. **Connect to the server**
   ```bash
   ssh staging
   ```

6. **Copy your backend project**
   ```bash
   scp -r backend/ staging:/home/ubuntu/
   ```

7. **Restart the application service**
   ```bash
   sudo systemctl restart myapp
   ```

8. **Verify the service status**
   ```bash
   sudo systemctl status myapp
   ```

**Explain in your notes why each step is important.**

---

## Part 4: Remote Deployment Runbook for a MERN Application

Create a `deployment-checklist.md` file containing a runbook with these sections:

### 1. SSH Connection Steps
```bash
ssh ubuntu@<server-ip>
```
Or using an alias:
```bash
ssh production
```

### 2. Secure Key Management Practices
- Private keys must have `600` permissions
- Never share private keys
- Use key-based authentication instead of passwords
- Use a strong passphrase for important keys
- Never commit private keys to Git repositories

### 3. File Transfer Process
```bash
# Copy backend to server
scp -r backend/ ubuntu@<server-ip>:/home/ubuntu/

# Copy config file
scp .env ubuntu@<server-ip>:/opt/mern/backend/
```

### 4. Deployment Commands
```bash
cd /opt/mern/backend
git pull
npm install
```

### 5. Service Restart Procedure
```bash
sudo systemctl restart myapp
```

### 6. Post-Deployment Verification
```bash
# Check service status
sudo systemctl status myapp

# Verify listening port
ss -tuln | grep 3000

# Test application response
curl -I http://localhost:3000
```

---

## Part 5: Knowledge Check

Answer the following questions:

1. **What is the difference between `ssh-keygen -t ed25519` and `ssh-keygen -t rsa -b 4096`?**

   <details>
   <summary>Answer</summary>

   `ed25519` is the modern, faster, and more secure algorithm. `rsa -b 4096` is a 4096-bit RSA key used for broader compatibility with older systems that don't support Ed25519.
   </details>

2. **Why must private keys always stay on your computer?**

   <details>
   <summary>Answer</summary>

   The private key proves my identity. If it's compromised, anyone holding it can log in to any server that trusts the corresponding public key.
   </details>

3. **Why should you verify key-based authentication works before disabling password authentication?**

   <details>
   <summary>Answer</summary>

   If key-based login isn't configured correctly and I disable passwords first, I'll be locked out of the server with no way to get back in.
   </details>

4. **What does the SSH config file do?**

   <details>
   <summary>Answer</summary>

   It defines aliases for servers with their hostname, username, and identity file, simplifying long SSH commands into short aliases.
   </details>

5. **Why is the SSH agent useful?**

   <details>
   <summary>Answer</summary>

   It keeps decrypted keys in memory so I don't have to type my passphrase repeatedly, which is essential for automation and multi-hop connections.
   </details>

---

## Submission Checklist

- [ ] Created `~/devops-labs/day10/` directory structure
- [ ] Generated an SSH key pair
- [ ] Verified private key permissions (600)
- [ ] Viewed public key
- [ ] Created `config-example.txt`
- [ ] Explained `authorized_keys` vs `known_hosts` in `ssh-notes.md`
- [ ] Completed all lab steps
- [ ] Created the deployment runbook in `deployment-checklist.md`
- [ ] Answered the knowledge check questions
</content>
