# 👨‍💻 Coding / Scripting Exercise

## 13. 👨‍💻 Coding / Scripting Exercise

Create a Bash script named `network_check.sh`:

```bash
#!/bin/bash

echo "Hostname:"
hostname

echo

echo "IP Addresses:"
ip addr

echo

echo "Routing Table:"
ip route

echo

echo "Listening Ports:"
ss -tuln
```

Make it executable:

```bash
chmod +x network_check.sh
```

Run it:

```bash
./network_check.sh
```

**Expected output (example):**

```
Hostname:
ubuntu-server

IP Addresses:
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 ...
    inet 127.0.0.1/8 scope host lo
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    inet 192.168.1.15/24 brd 192.168.1.255 scope global eth0

Routing Table:
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.15

Listening Ports:
LISTEN 0 128 0.0.0.0:22 0.0.0.0:*
LISTEN 0 128 0.0.0.0:80 0.0.0.0:*
LISTEN 0 128 0.0.0.0:443 0.0.0.0:*
```

---

# 🔬 Hands-on Lab

## 14. 🧪 Hands-on Lab
### Lab: Investigate a Simulated MERN Server

**Goal:** Practice using networking commands to inspect a server after an alert.

### Instructions

Imagine you've SSH'd into a server after an alert. Perform the following:

**Step 1** — Display the hostname:
```bash
hostname
```

**Step 2** — Display IP addresses:
```bash
ip addr
```

**Step 3** — Display the routing table:
```bash
ip route
```

**Step 4** — Test connectivity to a public site with ping:
```bash
ping -c 4 google.com
```

**Step 5** — Retrieve HTTP headers from https://example.com:
```bash
curl -I https://example.com
```

**Step 6** — List listening ports:
```bash
ss -tuln
```

**Step 7** — Perform a DNS lookup for example.com:
```bash
nslookup example.com
```

**Step 8** — If available, run a detailed DNS query:
```bash
dig example.com
```

### Documentation Template

Record your findings in a markdown file:

```markdown
# Server Investigation Report

## Hostname
[Your server hostname]

## IP Address
[Your IP address]

## Default Gateway
[Your gateway IP]

## DNS Resolution
- example.com resolves to: [IP address]
- DNS server used: [DNS server IP]
- DNS working: [Yes/No]

## Listening Services
| Port | Service |
|------|---------|
| 22   | SSH     |
| 80   | HTTP    |
| ...  | ...     |

## Connectivity
- ping to google.com: [Success/Failure]
- HTTP headers from example.com: [Status code]

## Notes
[Any observations or issues found]
```

---

# 📋 Mini Assignment

## 15. 📋 Mini Assignment
### MERN API Connectivity Troubleshooting Checklist

A user reports:

> **"Our React frontend loads, but the API requests fail."**

Write a troubleshooting checklist using today's commands. Include:

1. **How you'll verify DNS.**
2. **How you'll test the API endpoint.**
3. **How you'll check if the backend is listening.**
4. **How you'll distinguish between a network issue and an application issue.**

---

### ✅ Sample Solution: MERN API Troubleshooting Checklist

#### Step 1 — Verify DNS Resolution

Before assuming the server is down, verify that the domain resolves correctly.

```bash
# Check the IP the domain resolves to
nslookup api.company.com

# Get detailed DNS info
dig api.company.com +short

# Compare with expected IP
# If the IP is wrong → DNS propagation issue or misconfigured A record
# If DNS fails → DNS server down or domain expired
```

**Expected:** Domain resolves to the server's public IP.
**If not:** Check DNS provider, domain registration, and TTL.

---

#### Step 2 — Test the API Endpoint

Test the API from your local machine first.

```bash
# Check HTTP response headers
curl -I https://api.company.com/health

# Expected: HTTP/2 200 or HTTP/1.1 200 OK
# If timeout → Firewall or network issue
# If connection refused → Service not running or wrong port
# If 4xx/5xx → Application error (not network)

# Check the full response body
curl https://api.company.com/health

# Expected: {"status": "ok"} or similar
```

**Decision:**
- **200 OK** → DNS and network are fine. Issue is likely application-specific.
- **Connection timeout** → Firewall, security group, or routing issue.
- **Connection refused** → Service not listening on that port.
- **4xx/5xx** → Application error — check logs.

---

#### Step 3 — Check If the Backend Is Listening (from the server)

SSH into the server and verify the application is running.

```bash
# Check if the port is open
ss -tuln | grep 3000

# Expected: LISTEN 0 128 0.0.0.0:3000
# If nothing → Application is not running
# If only 127.0.0.1:3000 → Only listening locally (needs reverse proxy)

# Test locally
curl http://localhost:3000/health

# If localhost works → Issue is with Nginx, firewall, or security group
# If localhost fails → Application is down

# Check the process
ps aux | grep node
# Expected: nodeapp 1234 node app.js

# Check the service status
systemctl status backend
# If inactive → Start the service
# If failed → Check logs
```

---

#### Step 4 — Distinguish Between Network and Application Issues

**Use this decision framework:**

| Test | Result | Diagnosis |
|------|--------|-----------|
| `nslookup api.company.com` | ✅ Resolves to correct IP | DNS is fine |
| `curl -I api.company.com` | ⏱ Timeout | **Network issue** → Firewall, security group, routing |
| `ssh server-ip` | ✅ Connects | Server is online |
| `curl localhost:3000` | ✅ 200 OK | Application is running |
| `curl -I api.company.com` | ❌ 502 Bad Gateway | **Application issue** → Nginx can't reach backend |
| `curl -I api.company.com` | ❌ 503 Service Unavailable | **Application issue** → Backend overloaded or down |

**Network issue (timeout):**
```
nslookup ✅ → curl ❌ (timeout) → ssh ✅
→ Check: Security group inbound rules, firewall (ufw/iptables), Nginx
```

**Application issue (error response):**
```
nslookup ✅ → curl ❌ (5xx) → localhost ✅
→ Check: Nginx proxy_pass config, application logs, environment variables
```

**Application issue (not running):**
```
nslookup ✅ → curl ❌ (refused) → localhost ❌
→ Check: systemctl status, application logs, recent deployments
```

---

#### Quick Reference: Common Scenarios

```bash
# Scenario 1: API returns timeout
# Fix: Check AWS security group / firewall
sudo ufw status
# Ensure port 443 is open

# Scenario 2: API returns 502 Bad Gateway
# Fix: Check Nginx proxy_pass
nginx -t
tail -50 /var/log/nginx/error.log

# Scenario 3: API returns connection refused
# Fix: Start the Node.js service
sudo systemctl restart backend
tail -50 /var/log/mern/backend.log

# Scenario 4: API returns wrong data
# Fix: Check application config and environment
cat /var/www/mern-app/backend/.env
