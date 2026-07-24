# Linux Commands (Day 6 — Networking Commands)

## Network Interface & Routing
| Command | Purpose |
|---------|---------|
| `ip addr` | Show network interfaces and IP addresses |
| `ip route` | Display routing table |
| `hostname` | Show the system hostname |
| `sudo hostname new-name` | Change hostname (temporary) |

## Connectivity Testing
| Command | Purpose |
|---------|---------|
| `ping google.com` | Test connectivity (press Ctrl+C to stop) |
| `ping -c 4 google.com` | Send 4 ping packets then stop |
| `traceroute google.com` | Display network path hops |

## HTTP & File Transfer
| Command | Purpose |
|---------|---------|
| `curl https://example.com` | Send HTTP request, show response body |
| `curl -I https://example.com` | Fetch HTTP headers only |
| `curl http://localhost:3000` | Test a local service (e.g., Node.js) |
| `wget https://example.com/file.zip` | Download a file |

## Port & Service Inspection
| Command | Purpose |
|---------|---------|
| `ss -tuln` | Display listening TCP/UDP ports (numeric) |
| `netstat -tuln` | Legacy command for listening ports |
| `ss -tulnp` | Show listening ports with process names (requires root) |

## DNS Resolution
| Command | Purpose |
|---------|---------|
| `nslookup example.com` | Basic DNS lookup |
| `dig example.com` | Detailed DNS query with records |

## Quick Examples

**Find your IP address:**
```bash
ip addr | grep inet
```

**Check if a web server is running:**
```bash
curl -I http://localhost:80
```

**Check if Node.js is listening on port 3000:**
```bash
ss -tuln | grep 3000
```

**Resolve a domain:**
```bash
nslookup google.com
```

**Download a script and run it:**
```bash
wget https://example.com/deploy.sh
chmod +x deploy.sh
./deploy.sh
```

**Troubleshooting one-liner — check everything at once:**
```bash
echo "=== Hostname ===" && hostname && echo "=== IP ===" && ip addr && echo "=== Route ===" && ip route && echo "=== DNS ===" && nslookup google.com && echo "=== Ports ===" && ss -tuln
