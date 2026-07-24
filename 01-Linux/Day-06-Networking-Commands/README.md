# Day-06: Linux Networking Commands

## 🎯 Learning Objectives

By the end of today's lesson, you'll be able to:

- Understand basic network troubleshooting workflow.
- View network interfaces and IP addresses.
- Test connectivity.
- Inspect listening ports.
- Verify DNS resolution.
- Download files from the command line.
- Test HTTP endpoints.
- Apply these commands to troubleshoot MERN applications in production.

---

## 🤔 Why This Topic Is Important

Imagine your production MERN application is down.

Users report:

```
https://api.myapp.com is unreachable.
```

Possible causes:

- Server offline
- DNS issue
- Firewall blocking traffic
- Application not listening
- Reverse proxy misconfiguration
- Network routing issue

A DevOps engineer uses Linux networking commands to isolate the problem systematically.

---

## 📖 Theory Explained in Simple Language

A network request travels through several layers:

```
Browser
   ↓
   DNS
   ↓
Internet
   ↓
  Server
   ↓
Operating System
   ↓
Application
```

If any layer fails, the application may become unreachable.

Your job is to determine where the failure occurs.

### Network Interface

Think of a network interface as a network card.

View interfaces:

```bash
ip addr
```

**Example output:**
```
2: eth0    inet 192.168.1.15/24
```

`eth0` (or names like `ens5`) is a common network interface.

> **Legacy Command:** `ifconfig` — older systems may use this. Prefer `ip addr`.

### View Routing Table

```bash
ip route
```

**Example:**
```
default via 192.168.1.1 dev eth0
```

This shows the default gateway used to reach external networks.

### Test Connectivity with `ping`

```bash
ping google.com
```

**Output:**
```
64 bytes from ... time=22 ms
```

Stop with `Ctrl + C`. Use a fixed number of packets:

```bash
ping -c 4 google.com
```

### Test HTTP Services with `curl`

```bash
curl https://example.com
```

Returns the response body.

Check only headers:

```bash
curl -I https://example.com
```

**Example:**
```
HTTP/2 200
```

Test a local Node.js API:

```bash
curl http://localhost:3000
```

### Download Files with `wget`

```bash
wget https://example.com/file.zip
```

Useful for:
- Downloading software
- Fetching scripts
- Retrieving release artifacts

### Inspect Listening Ports with `ss`

Modern command:

```bash
ss -tuln
```

Options:
- `t` → TCP
- `u` → UDP
- `l` → Listening sockets
- `n` → Numeric output

**Example:**
```
LISTEN 0.0.0.0:3000
```

If your Node.js app is running, you should see its port here.

> **Legacy:** `netstat -tuln` — many systems now prefer `ss`.

### View Hostname

```bash
hostname
```

**Example:** `prod-web-01`

Change temporarily (requires privileges):
```bash
sudo hostname new-server-name
```

### DNS Lookup with `nslookup`

```bash
nslookup google.com
```

Shows:
- IP address
- DNS server used

### Advanced DNS Query with `dig`

```bash
dig google.com
```

Useful information:
- Answer section
- TTL
- DNS records
- Query time

### Trace the Network Path

```bash
traceroute google.com
```

Shows each hop between your machine and the destination. Useful when identifying routing problems.

> Some distributions require installing `traceroute` first.

---

## 🌍 Real-World Example: MERN Backend Is Down

Users report:

```
https://api.company.com returns an error.
```

**Step 1** — Check network:
```bash
ping api.company.com
```

**Step 2** — Check DNS:
```bash
nslookup api.company.com
```

**Step 3** — Check HTTP response:
```bash
curl -I https://api.company.com
```

**Step 4** — SSH into the server and verify the application is listening:
```bash
ss -tuln
```
Look for `:3000`.

**Step 5** — Test locally:
```bash
curl http://localhost:3000
```

If localhost works but the public URL doesn't, the issue may be with a firewall, reverse proxy, or load balancer.

---

## 🏗️ Architecture Diagram (ASCII)

```
          Browser
             │
          DNS Lookup
             │
         Internet
             │
        Linux Server
             │
        Port 443 (HTTPS)
             │
          Nginx
             │
        Port 3000
             │
       Node.js Backend
```

---

## 🛠️ Step-by-Step Practical Implementation

**Step 1** — View your IP addresses:
```bash
ip addr
```

**Step 2** — View routing information:
```bash
ip route
```

**Step 3** — Display the hostname:
```bash
hostname
```

**Step 4** — Test internet connectivity:
```bash
ping -c 4 google.com
```

**Step 5** — Test HTTP headers:
```bash
curl -I https://example.com
```

**Step 6** — Download a sample file:
```bash
wget https://example.com/
```

This saves the response to a local file.

**Step 7** — Inspect listening ports:
```bash
ss -tuln
```

**Step 8** — Resolve a domain:
```bash
nslookup example.com
```

**Step 9** — Run a detailed DNS query:
```bash
dig example.com
```

> Install `dnsutils` (Debian/Ubuntu) if `dig` is unavailable.

---

## ✅ Best Practices

- Use `ip` instead of legacy networking commands when possible.
- Test DNS before assuming the application is broken.
- Use `curl` to verify API responses instead of relying solely on a browser.
- Confirm your application is listening with `ss`.
- Troubleshoot from the network layer upward: **connectivity → DNS → ports → application**.
