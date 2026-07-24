# 💼 Interview Questions (Networking Commands) — with Detailed Answers & Examples

## Beginner

### 1. What does `ping` test?
**Answer:** `ping` tests **network connectivity** between your machine and a remote host by sending ICMP echo request packets and measuring the round-trip time.

**Example:**
```bash
ping -c 4 google.com
# Output:
# 64 bytes from 142.250.80.46: icmp_seq=1 ttl=118 time=22.3 ms
# 64 bytes from 142.250.80.46: icmp_seq=2 ttl=118 time=21.8 ms
# --- google.com ping statistics ---
# 4 packets transmitted, 4 received, 0% packet loss, time 3004ms
```

**Troubleshooting value:**
- **Success with low time** → Good connectivity.
- **Packet loss** → Network congestion or hardware issues.
- **100% loss** → Host unreachable, firewall blocking ICMP, or server down.

---

### 2. What is the purpose of `curl`?
**Answer:** `curl` is used to **send HTTP requests** and display the server's response. It's essential for testing APIs, checking HTTP status codes, and verifying that web services are responding correctly.

**Examples:**
```bash
# Get the full response body
curl https://api.example.com/users

# Check HTTP headers only (faster, no body)
curl -I https://api.example.com

# Test a local Node.js API
curl http://localhost:3000/health

# Send a POST request with data
curl -X POST -H "Content-Type: application/json" -d '{"name":"test"}' http://localhost:3000/api
```

---

### 3. What does `hostname` display?
**Answer:** `hostname` displays the **system's hostname** — the name assigned to the machine on the network. This is useful for identifying which server you're connected to.

**Example:**
```bash
hostname
# Output: prod-web-01
```

---

### 4. Which command shows IP addresses?
**Answer:** `ip addr` (or `ip a` for short) shows all network interfaces and their assigned IP addresses.

**Example:**
```bash
ip addr
# Output:
# 1: lo: <LOOPBACK,UP,LOWER_UP> ...
#     inet 127.0.0.1/8 scope host lo
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
#     inet 192.168.1.15/24 brd 192.168.1.255 scope global eth0
```

---

### 5. What does `wget` do?
**Answer:** `wget` **downloads files** from the internet via HTTP, HTTPS, or FTP. It's commonly used in DevOps for downloading software, scripts, and release artifacts.

**Examples:**
```bash
# Download a file
wget https://example.com/file.zip

# Download with a different output filename
wget -O deploy.sh https://example.com/scripts/deploy.sh

# Resume an interrupted download
wget -c https://example.com/large-file.iso
```

**Contrast with `curl`:** `wget` is better for downloading files (recursive, resumable); `curl` is better for testing APIs (more protocol support, more control over requests).

---

## Intermediate

### 6. Explain the difference between `curl` and `wget`.
**Answer:**

| Feature | `curl` | `wget` |
|---------|--------|--------|
| Primary use | API testing, HTTP debugging | File downloading |
| Default output | Shows response body to terminal | Saves to a file |
| Resume downloads | Limited support | Built-in (`-c` flag) |
| Recursive downloads | No | Yes (`-r` flag) |
| Protocol support | More (HTTP, HTTPS, FTP, SFTP, SMB, etc.) | Fewer (HTTP, HTTPS, FTP) |
| Request methods | GET, POST, PUT, DELETE, etc. | GET only |
| Header inspection | `curl -I` | Requires extra options |

**When to use each:**
```bash
# API testing → curl
curl -I https://api.company.com/health

# Downloading artifacts → wget
wget https://github.com/user/project/releases/download/v1.0/app.tar.gz
```

---

### 7. Why is `ss` preferred over `netstat` on modern Linux systems?
**Answer:** `ss` (socket statistics) is preferred because:
- **Faster** — reads kernel information directly
- **More detailed** — shows more connection state information
- **Modern replacement** — `netstat` is deprecated in many distributions
- **Consistent output** — better for scripting

**Example comparison:**
```bash
# Both commands do the same thing
ss -tuln     # Modern, fast
netstat -tuln # Legacy, slower

# ss can show process names
sudo ss -tulnp
```

---

### 8. How would you verify that a Node.js application is listening on port 3000?
**Answer:**

**Method 1 — Using `ss`:**
```bash
ss -tuln | grep 3000
# Expected: LISTEN 0 128 0.0.0.0:3000 0.0.0.0:*
```

**Method 2 — Using `curl` to test the service:**
```bash
curl http://localhost:3000
# Expected: HTML/JSON response (not "Connection refused")
```

**Method 3 — Using `lsof` (if installed):**
```bash
sudo lsof -i :3000
# Expected: node 1234 nodeapp 11u IPv4 56789 TCP *:3000 (LISTEN)
```

**If nothing shows up:**
```bash
# The application may not be running
systemctl status backend
# Check logs
tail -50 /var/log/mern/backend.log
```

---

### 9. What information does `ip route` provide?
**Answer:** `ip route` displays the **routing table**, which tells the system how to reach different networks. The most important entry is the **default gateway**.

**Example:**
```bash
ip route
# Output:
# default via 192.168.1.1 dev eth0 proto static
# 192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.15
```

**Key information:**
- `default via 192.168.1.1` — All traffic to external networks goes through this gateway
- `192.168.1.0/24 dev eth0` — Local network is directly connected
- Useful for verifying the server can reach the internet

---

### 10. How do `nslookup` and `dig` differ?
**Answer:**

| Feature | `nslookup` | `dig` |
|---------|------------|-------|
| Detail level | Basic | Advanced |
| Output format | Simple, human-readable | Detailed, structured |
| DNS record types | Common types | All types (A, AAAA, MX, NS, TXT, etc.) |
| Query time | Not shown | Shown |
| TTL information | Limited | Full TTL for each record |
| Use case | Quick checks | Detailed DNS debugging |

**Examples:**
```bash
# nslookup — quick check
nslookup google.com
# Output: Address: 142.250.80.46

# dig — detailed analysis
dig google.com
# Output includes: status, query time, answer section, authority, additional
```

---

## Advanced

### 11. A MERN API is reachable via localhost but not from the internet. Describe your troubleshooting process.
**Answer:**

**Step 1 — Check if the application is listening on the correct interface:**
```bash
ss -tuln | grep 3000
```
If it shows `127.0.0.1:3000` instead of `0.0.0.0:3000`, the app is only listening locally.

**Fix:** Bind to `0.0.0.0` or configure Nginx as a reverse proxy.

**Step 2 — Check the firewall:**
```bash
# Check if UFW is blocking the port
sudo ufw status

# Check iptables rules
sudo iptables -L -n | grep 3000
```

**Step 3 — Check if Nginx (reverse proxy) is running:**
```bash
systemctl status nginx
ss -tuln | grep 443
curl -I http://localhost:80
```

**Step 4 — Check external connectivity:**
```bash
# From another machine:
curl -I https://api.company.com
# If connection times out → firewall/security group issue
# If connection refused → service not running or wrong port
```

**Step 5 — Check cloud security groups (if applicable):**
- AWS: Inbound rules for the security group
- GCP: Firewall rules
- Azure: Network security group

**Decision tree:**
```
Can reach via localhost?
├── Yes → Can reach via server IP internally?
│   ├── Yes → Can reach via domain externally?
│   │   ├── Yes → DNS/SSL issue
│   │   └── No → Firewall/security group blocking
│   └── No → App not binding to 0.0.0.0
└── No → Application not running
```

---

### 12. Explain how DNS failures affect application availability.
**Answer:** DNS failures can make an application completely unreachable even though the server is perfectly healthy.

**How DNS affects availability:**

```
User's Browser
     ↓
  "What is the IP of api.myapp.com?"
     ↓
   DNS Server
     ↓
  ❌ No response / Wrong IP
     ↓
  Browser cannot connect → Error
```

**Common DNS failures:**

| Failure | Symptom | Impact |
|---------|---------|--------|
| Domain expired | `NXDOMAIN` | Complete outage |
| TTL propagation delay | Stale IP | Partial outage during migration |
| DNS server down | Timeout | Users in region cannot reach app |
| Wrong A record | Wrong server | Users sent to incorrect IP |
| CNAME misconfigured | Circular reference | DNS resolution fails |

**Diagnosing DNS issues:**
```bash
# Check if DNS resolves correctly
nslookup api.company.com

# Compare with expected IP
dig api.company.com +short

# Check against a different DNS server
dig @8.8.8.8 api.company.com

# Check if the server itself can resolve
curl -I https://api.company.com
```

---

### 13. How would you diagnose a timeout versus a connection refused error?
**Answer:** These two errors indicate different problems:

**Connection refused:**
```bash
curl http://localhost:3000
# curl: (7) Failed to connect to localhost port 3000: Connection refused
```

**Meaning:** The server is reachable, but **no application is listening** on that port.

**Causes:**
- Application crashed
- Application not started
- Wrong port number
- Application listening on a different interface

**Timeout:**
```bash
curl https://api.company.com
# curl: (28) Connection timed out after 30000 milliseconds
```

**Meaning:** The server is **not reachable** at all — packets are not getting through.

**Causes:**
- Server is down
- Firewall blocking traffic
- Network routing issue
- Security group blocking inbound traffic
- DDoS protection blocking the request

**Diagnosis comparison:**

| Check | Connection Refused | Timeout |
|-------|-------------------|---------|
| `ss -tuln \| grep :3000` | Nothing | — |
| `ping server-ip` | ✅ Works | ❌ Fails or blocked |
| `curl localhost:3000` | ❌ Refused | — |
| `systemctl status app` | ❌ Inactive | — |

---

### 14. Why might `ping` fail even though a web service is healthy?
**Answer:** `ping` can fail for reasons unrelated to the web service health:

**1. ICMP blocked by firewall:**
```bash
# ping fails, but the web service works
ping google.com
# Request timed out

curl -I https://google.com
# HTTP/2 200  ✅
```
Many cloud providers and servers block ICMP (ping) traffic for security.

**2. Server may not respond to ICMP but application responds to HTTP:**
- AWS EC2 instances block ICMP by default in security groups.
- Many production servers disable ICMP responses to prevent ping floods.

**Better health check approach:**
```bash
# Instead of ping, use:
curl -I https://api.company.com        # Check HTTP response
nc -zv api.company.com 443             # Check port connectivity
ss -tuln | grep 3000                   # Check local service
```

---

### 15. How do reverse proxies like Nginx fit into network troubleshooting?
**Answer:** Nginx (or similar reverse proxies) sit between the client and the application server, so they add both complexity and diagnostic value.

**Architecture:**
```
Client → Port 443 (HTTPS) → Nginx → Port 3000 (HTTP) → Node.js
```

**What Nginx does:**
- Terminates SSL/TLS
- Routes traffic to backend services
- Adds caching, rate limiting, load balancing

**Troubleshooting with Nginx:**

**Step 1 — Check if Nginx itself is running:**
```bash
systemctl status nginx
ss -tuln | grep -E "443|80"
```

**Step 2 — Check Nginx configuration:**
```bash
nginx -t
# Test configuration syntax
cat /etc/nginx/sites-enabled/default | grep proxy_pass
# Verify upstream target
```

**Step 3 — Check Nginx access and error logs:**
```bash
tail -100 /var/log/nginx/access.log
tail -100 /var/log/nginx/error.log
```

**Step 4 — Test each layer separately:**
```bash
# Test backend directly
curl http://localhost:3000/health

# Test through Nginx locally
curl http://localhost/health

# Test through Nginx externally
curl https://api.company.com/health
```

**If localhost:3000 works but localhost/health fails:**
- Nginx proxy_pass is misconfigured
- Nginx is pointing to wrong port or path

**If localhost/health works but https://api.company.com fails:**
- SSL certificate issue
- DNS issue
- Firewall blocking port 443
- Security group misconfiguration

**Key Nginx log locations:**
```bash
/var/log/nginx/access.log   # All requests
/var/log/nginx/error.log    # Errors and warnings
