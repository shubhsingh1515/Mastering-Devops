# 👨‍💻 Coding / Scripting Exercise

## Exercise 1: Production Directory & File Operations

Create the following structure:

```
deployment/
├── backend/
│   ├── app.log
│   └── server.log
├── frontend/
└── backups/
```

Then:

1. Add two lines of text to `app.log`.
2. Copy `app.log` into `backups/`.
3. Rename `server.log` to `api.log`.
4. Use `find` to locate all `.log` files.
5. Display the last line of `api.log`.

---

# 🔬 Hands-on Lab

## Lab: Simulate Basic Production Log Analysis

### Goal
Build the habit of using shell tools to inspect and troubleshoot systems.

### Instructions
1. Create a file named `application.log`.
2. Add at least 20 log entries using:
   ```bash
   echo "Log entry X" >> application.log
   ```
   (Replace X with numbers 1 through 20.)
3. Use the following commands to read the file:
   - `head`
   - `tail`
   - `less`
4. Search for all `.log` files in your practice directory.
5. Check the directory's size using `du -sh`.
6. Check overall disk usage with `df -h`.

### Expected Commands Summary
```bash
touch application.log
echo "Log entry 1" >> application.log
echo "Log entry 2" >> application.log
# ... repeat to 20
head application.log
tail -5 application.log
less application.log
find . -name "*.log"
du -sh .
df -h
```

---

# 📋 Mini Assignment

## Assignment: MERN Backend Troubleshooting Checklist

Imagine your MERN backend has stopped responding.

Write a troubleshooting checklist using **only the commands learned in Day 3**. Include:

1. **How you'll navigate to the application directory.**
   - (e.g., `cd /home/ubuntu/mern-backend`)

2. **How you'll inspect recent logs.**
   - (e.g., `tail -100 /var/log/mern/backend.log`)

3. **How you'll watch the log in real time while reproducing the issue.**
   - (e.g., `tail -f /var/log/mern/backend.log`)

4. **How you'll verify disk space.**
   - (e.g., `df -h`, `du -sh /var/log/mern`)

5. **How you'll locate configuration or log files.**
   - (e.g., `find /home/ubuntu -name "*.log"`, `find /home/ubuntu -name "*.env"`)

6. **Which commands you'll use and why.**
   - Include at least 5 different commands with explanations.

Write your answer in a clear, step-by-step format suitable for a DevOps runbook.

---

# 🧪 Additional Practice: Pipe & Redirection

Create a one-liner command that:

1. Uses `history` to list recent commands.
2. Pipes to `grep` to find commands containing "docker".
3. Redirects the output to a file named `docker-commands.txt`.

**Expected one-liner:**
```bash
history | grep "docker" > docker-commands.txt
```

Then verify:
```bash
cat docker-commands.txt
