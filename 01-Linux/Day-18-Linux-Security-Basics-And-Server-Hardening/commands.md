# Day 18 Commands: Linux Security & Server Hardening

## Identity and privileges
```bash
whoami
groups
sudo -l
```

## SSH and permissions
```bash
ls -ld ~/.ssh
ls -l ~/.ssh/id_ed25519 2>/dev/null
chmod 600 ~/.ssh/id_ed25519
chmod 700 ~/.ssh
```

## SSH validation and reload
```bash
sudo sshd -t
sudo systemctl reload ssh
# or
sudo systemctl reload sshd
```

## Ports and services
```bash
ss -tuln
ss -tulpn | grep LISTEN
```

## Firewall basics with ufw
```bash
sudo ufw status
sudo ufw status verbose
sudo ufw allow ssh
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status numbered
```

## Secure file permissions
```bash
chmod 600 .env
chmod 700 /opt/mern
chown -R nodeapp:nodeapp /opt/mern
```

## Avoid dangerous permissions
```bash
chmod 777 file.txt
# Avoid this in production
```

## Useful security review commands
```bash
sudo journalctl -u nginx -n 50
sudo journalctl -u ssh -n 50
sudo journalctl -xe
```

## Production security mindset
```bash
# inventory services
ss -tuln

# review users
id
getent passwd

# review sudo access
sudo -l

# check firewall
sudo ufw status verbose
```
