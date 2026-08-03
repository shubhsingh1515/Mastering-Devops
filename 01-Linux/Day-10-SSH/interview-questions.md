# Day 10 — Interview Questions: SSH (Secure Shell)

## Beginner

### Q1: What is SSH?
**SSH (Secure Shell)** is a network protocol that provides secure, encrypted communication between two computers. It's used to remotely access and manage servers, transfer files, and tunnel connections. Unlike older protocols like Telnet, SSH encrypts all traffic so that credentials and commands cannot be read by anyone on the network.

### Q2: What is the default SSH port?
The default SSH port is **22 (TCP)**. This is the standard port the SSH server (`sshd`) listens on and that the SSH client connects to.

### Q3: What is the difference between a public key and a private key?
- **Private key** — Stays on my computer and is never shared. It's used to prove my identity and decrypt data.
- **Public key** — Safe to share. It's placed on servers and used to verify my identity and encrypt data.

They form a **key pair**: anything encrypted with the public key can only be decrypted with the private key, and vice versa.

### Q4: What is scp used for?
`scp` (Secure Copy) is used to **copy files securely** between computers over SSH. I can copy files to a server (`scp file user@host:path`), copy directories with `-r`, or copy files from a server back to my machine.

### Q5: Why is SSH preferred over Telnet?
- **SSH encrypts** all traffic (passwords, commands, data), so nothing can be intercepted.
- **Telnet sends everything in plaintext**, which anyone on the network can read.
- SSH supports stronger authentication (key-based auth, 2FA) and is the modern industry standard for remote administration.

---

## Intermediate

### Q6: Explain the SSH authentication process.
1. Client connects to the server on port 22.
2. Server presents its host key, which the client verifies against `known_hosts`.
3. Client presents its public key, which the server checks against `authorized_keys`.
4. Server sends a **cryptographic challenge** encrypted with the client's public key.
5. Client proves it holds the matching **private key** by decrypting the challenge.
6. If verification succeeds, an encrypted session is established.
7. **No password is ever sent over the network.**

### Q7: What is the purpose of authorized_keys?
`~/.ssh/authorized_keys` on the **server** contains the public keys of all users who are allowed to log in. When a client connects, the server checks whether the client's public key appears in this file. If it does, the client can authenticate using the matching private key.

### Q8: What information is stored in known_hosts?
`~/.ssh/known_hosts` on the **client** stores the **host fingerprints** (public keys) of servers I've connected to before. This allows SSH to detect when a server's identity changes unexpectedly — which could indicate a man-in-the-middle attack or a compromised server.

### Q9: Why should private keys have 600 permissions?
The `600` permission (`rw-------`) means **only the owner** can read or modify the key. If a private key has overly permissive permissions (like `644`), any user on the system could read it and impersonate me. In fact, SSH itself **refuses to use** private keys with insecure permissions.

### Q10: What is the benefit of an SSH config file?
The `~/.ssh/config` file lets me define **aliases** for frequently accessed servers, including their hostname, username, and identity file. Instead of typing a long command like `ssh ubuntu@203.0.113.10 -i ~/.ssh/id_ed25519`, I just type `ssh prod-api`. This simplifies managing multiple servers and environments.

---

## Advanced

### Q11: Design a secure SSH strategy for a production MERN deployment.
A secure SSH strategy includes:
1. **Key-based authentication only** — Disable password authentication after keys are configured.
2. **No root login** — Use a regular user with `sudo` privileges instead of logging in as root.
3. **Restrict access** — Limit SSH access to specific users and trusted IP ranges (using a firewall or security groups).
4. **Protect private keys** — `600` permissions and a strong passphrase.
5. **Separate keys per environment** — Different keys for dev, staging, and production.
6. **Monitor authentication logs** — Watch `/var/log/auth.log` for failed login attempts and suspicious activity.
7. **Consider a jump host (bastion)** — A single hardened entry point for accessing the production network.

### Q12: How would you disable password authentication after configuring SSH keys?
In the SSH server configuration file `/etc/ssh/sshd_config`, I would set:
```
PasswordAuthentication no
```
Then restart the SSH service:
```bash
sudo systemctl restart ssh
```
**Critical safety step:** Verify key-based login works **before** disabling passwords — otherwise I could lock myself out of the server.

### Q13: Explain the role of the SSH agent.
The **SSH agent** is a background process that keeps decrypted SSH keys in memory. When I use a key protected by a passphrase, the agent decrypts it once and stores it, so I don't have to enter the passphrase repeatedly during the session. This is especially useful for:
- Automation and scripts that need to make multiple SSH connections
- **SSH agent forwarding** for multi-hop connections (connecting from one server to another)

### Q14: How would you securely manage SSH access for multiple DevOps engineers?
Options for managing team SSH access:
1. **Individual keys** — Each engineer has their own key (never share keys).
2. **Centralized authentication** — Use an SSH certificate authority (CA) or tools like HashiCorp Vault to issue and revoke keys.
3. **Bastion/jump host** — A single hardened entry point that logs all SSH activity.
4. **Key rotation** — Regularly rotate keys and revoke access when engineers leave the team.
5. **Audit logging** — Track who connected, when, and what commands were run.
6. **Role-based access control** — Grant only the privileges each engineer needs (least privilege).

### Q15: Describe a secure deployment workflow using SSH, Git, and systemd.
A secure deployment workflow:
1. Developer pushes code to a Git repository (authenticated via SSH or HTTPS).
2. On the server, an operator (or CI/CD pipeline) connects via SSH.
3. Pull the latest code: `git pull`.
4. Install dependencies: `npm install`.
5. Restart the service: `sudo systemctl restart myapp`.
6. Verify the deployment:
   ```bash
   sudo systemctl status myapp
   ss -tuln | grep 3000
   curl -I http://localhost:3000
   ```
7. This workflow can be fully automated with CI/CD tools (Jenkins, GitHub Actions) that use SSH keys for server access.
</content>
