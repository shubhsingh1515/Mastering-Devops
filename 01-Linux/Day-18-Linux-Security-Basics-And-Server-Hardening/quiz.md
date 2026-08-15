# Day 18 Quiz: Linux Security Basics & Server Hardening

## Questions

1. What principle says users should receive only necessary permissions?
   - A. Defense by obscurity
   - B. Least privilege
   - C. Maximum privilege
   - D. Root-first administration

2. What command checks listening ports?
   - A. ss -tuln
   - B. ports -a
   - C. netports
   - D. systemctl ports

3. What is the purpose of sudo?
   - A. Compress files
   - B. Temporarily execute commands with elevated privileges
   - C. Create SSH keys
   - D. Start the firewall

4. Why is chmod 777 generally inappropriate for production application files?
   - A. It makes them read-only
   - B. It prevents execution
   - C. It grants overly broad permissions
   - D. It encrypts the files

5. What file commonly contains the SSH server configuration?
   - A. /etc/ssh/sshd_config
   - B. /etc/ssh/public
   - C. /var/ssh/config
   - D. /etc/ssh/firewall

6. Which command validates SSH daemon configuration?
   - A. ssh --check
   - B. sshd -t
   - C. ssh-test
   - D. systemctl validate ssh

7. Which ports would typically be needed for an HTTPS Nginx reverse proxy?
   - A. 3000 only
   - B. 27017 only
   - C. 80 and/or 443
   - D. 3306

8. Why should MongoDB generally not be publicly exposed without a compelling reason and strong controls?
   - A. MongoDB cannot use networking
   - B. It unnecessarily increases attack surface and potential unauthorized database access
   - C. It makes Nginx slower
   - D. It disables SSH

9. What does 127.0.0.1:3000 imply?
   - A. Port 3000 is listening on every interface
   - B. Port 3000 is locally bound to loopback
   - C. Port 3000 is blocked
   - D. Port 3000 belongs to SSH

10. What should you do before enabling a firewall on a remote server?
   - A. Remove SSH rules
   - B. Ensure required management access is explicitly allowed
   - C. Reboot
   - D. Delete existing network interfaces

## Answer Key
1. B
2. A
3. B
4. C
5. A
6. B
7. C
8. B
9. B
10. B
