# Day 18 Assignment: Linux Security Basics & Server Hardening

## Task 1: Review your current Linux access model
Run the following commands on your machine or VM:

```bash
whoami
groups
sudo -l
```

Write down:
- Whether you are using a normal user or root
- Which groups you belong to
- Whether your user can run sudo commands

## Task 2: Check SSH safety basics
```bash
ls -ld ~/.ssh
ls -l ~/.ssh/id_ed25519 2>/dev/null
```

Answer:
- Are key files protected with proper permissions?
- Are the SSH directories accessible only to the correct user?

## Task 3: Inspect exposed services
```bash
ss -tuln
```

Identify:
- Which ports are listening
- Which services are bound to all interfaces
- Which are only local loopback interfaces

## Task 4: Review firewall configuration
If ufw is installed:

```bash
sudo ufw status verbose
```

Check whether:
- SSH is allowed
- HTTP and HTTPS are allowed
- Unneeded ports are blocked

## Task 5: Secure a MERN server design
Imagine a server running:
- NGINX
- Node.js API
- MongoDB
- SSH

Create a target architecture showing:
- Only required public ports
- Local-only application ports
- Restricted MongoDB exposure
- Dedicated application user
- No unnecessary root execution

## Task 6: Write a hardening checklist
Create a checklist like this:

- Dedicated application user created
- SSH key-based authentication enabled
- Private keys protected
- Password authentication disabled where appropriate
- Root login restricted
- .env secured with 600 permissions
- Firewall rules configured
- Nginx only public entry point
- MongoDB not exposed publicly
- Logs reviewed regularly
- System updated regularly

## Submission idea
Submit a short security architecture note in your own words: 
"How would you harden a Linux server hosting a production MERN application?"
