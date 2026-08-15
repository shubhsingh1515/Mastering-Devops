# Day 18 Interview Questions: Linux Security & Server Hardening

## Beginner

### Q1. What is least privilege?
Giving users and processes only the permissions necessary to perform their tasks.

### Q2. Why shouldn't applications run as root?
A compromise of the application could otherwise provide the attacker with root-level privileges and greatly increase the impact.

### Q3. What does a firewall do?
It filters or controls network traffic based on defined rules to reduce unwanted exposure.

## Intermediate

### Q4. How would you secure SSH on a production server?
Use key-based authentication, protect private keys, disable direct root login, prefer non-root admin users, restrict SSH with firewall rules, and keep the SSH daemon updated.

### Q5. A MERN server has ports 22, 80, 443, 3000, and 27017 open publicly. What concerns you?
The most concerning exposures are Node.js on port 3000 and MongoDB on port 27017. These services should usually be restricted to internal networks or localhost unless there is a clear reason to expose them.

## Advanced

### Q6. Walk me through how you would harden a Linux server hosting a production MERN app.
I would:
1. Create a dedicated application user.
2. Remove root-based execution from the app.
3. Restrict file ownership and permissions.
4. Protect secrets such as .env files.
5. Harden SSH and disable password login if possible.
6. Configure firewall rules to allow only required ports.
7. Keep Nginx public, but restrict Node.js and MongoDB exposure.
8. Monitor logs and authentication events.
9. Apply package updates and security patches.
10. Review access and service exposure regularly.

### Q7. Why is 127.0.0.1:3000 better than 0.0.0.0:3000 from a security perspective?
It means the service is bound to the local loopback interface, so it is not reachable from outside the machine without additional routing or tunnel access.

### Q8. What are the most common mistakes in Linux server hardening?
- Running services as root
- Using chmod 777
- Exposing databases directly to the internet
- Allowing password-based SSH access
- Leaving all ports open
- Ignoring logs and patching
- Hardcoding secrets in source files or world-readable files

### Q9. Why is sudo safer than direct root login?
It enforces least privilege, leaves a trace of administrative actions, and reduces the risk of using a full root login for routine tasks.

### Q10. How do you know whether a service is unnecessarily exposed?
By reviewing listening ports, checking what the service is, mapping its role to the architecture, and confirming whether external clients truly need direct access.
