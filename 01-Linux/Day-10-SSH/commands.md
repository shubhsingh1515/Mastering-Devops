# Day 10 — Commands: SSH (Secure Shell)

| Command | Purpose | Example |
|---------|---------|---------|
| `ssh -V` | Show SSH version | `ssh -V` |
| `ssh user@host` | Connect to a remote server | `ssh ubuntu@203.0.113.10` |
| `ssh-keygen -t ed25519` | Generate Ed25519 SSH key pair | `ssh-keygen -t ed25519 -C "user@example.com"` |
| `ssh-keygen -t rsa -b 4096` | Generate RSA key pair (compat) | `ssh-keygen -t rsa -b 4096` |
| `scp file user@host:path` | Copy a file to a server | `scp app.js ubuntu@203.0.113.10:/home/ubuntu/` |
| `scp -r dir user@host:path` | Copy a directory to a server | `scp -r backend/ ubuntu@203.0.113.10:/home/ubuntu/` |
| `scp user@host:file .` | Copy a file from a server | `scp ubuntu@203.0.113.10:/var/log/app.log .` |
| `ssh-add <key>` | Add a key to the SSH agent | `ssh-add ~/.ssh/id_ed25519` |
| `eval "$(ssh-agent -s)"` | Start the SSH agent | `eval "$(ssh-agent -s)"` |
| `chmod 600 <key>` | Secure a private key | `chmod 600 ~/.ssh/id_ed25519` |
| `cat ~/.ssh/id_ed25519.pub` | Display the public key | `cat ~/.ssh/id_ed25519.pub` |
| `ls -l ~/.ssh` | List SSH keys | `ls -l ~/.ssh` |

## Key Files

| File | Location | Purpose |
|------|----------|---------|
| Private key | `~/.ssh/id_ed25519` | My secret key (never share, keep 600) |
| Public key | `~/.ssh/id_ed25519.pub` | My public key (safe to share) |
| authorized_keys | `~/.ssh/authorized_keys` (server) | Public keys allowed to log in |
| known_hosts | `~/.ssh/known_hosts` (client) | Server fingerprints I've connected to |
| SSH config | `~/.ssh/config` | Connection aliases and settings |

## Sample SSH Config

```
Host production
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
```

## Quick Reference

```bash
# Generate a key
ssh-keygen -t ed25519

# Secure the private key
chmod 600 ~/.ssh/id_ed25519

# Connect to a server
ssh ubuntu@203.0.113.10

# Copy a file
scp app.js ubuntu@203.0.113.10:/home/ubuntu/

# Copy a directory
scp -r backend/ ubuntu@203.0.113.10:/home/ubuntu/

# Add key to agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
</content>
