# Day-07: Linux Processes & Process Management

## 🎯 Learning Objectives

By the end of this lesson you'll be able to:

- Understand what a process is.
- Distinguish between a program and a process.
- View running processes.
- Find processes by name or PID.
- Stop, restart, and terminate processes.
- Understand foreground vs background execution.
- Monitor CPU and memory usage.
- Troubleshoot a production Node.js application.

---

## 🤔 Why This Topic Is Important

Imagine your MERN backend becomes slow.

Possible reasons:

- High CPU usage
- Memory leak
- Stuck process
- Zombie process
- Multiple Node.js instances
- Process crashed unexpectedly

Every DevOps engineer must know how to identify and manage processes.

---

## 📖 Theory Explained in Simple Language

### What is a Program?

A **program** is a file stored on disk.

**Example:** `node` or `nginx`

It isn't doing anything until it's executed.

### What is a Process?

A **process** is a running instance of a program.

**Example:** `node server.js`

Now Linux creates a process.

```
Program → Execution → Running Process
```

### Process ID (PID)

Every process has a unique Process ID.

**Example:**
```
PID 2451 → node
PID 781  → nginx
PID 991  → mongod
```

Linux uses the PID to manage a process.

### Parent & Child Processes

When one process starts another:

```
Terminal
   │
   └── bash
         │
         └── node server.js
```

- `bash` is the **parent**.
- `node` is the **child**.

### Foreground Process

Runs in your terminal.

**Example:**
```bash
node server.js
```

Your terminal is occupied until the process exits.

Stop with: `Ctrl + C`

### Background Process

Runs without blocking your terminal.

**Example:**
```bash
node server.js &
```

The `&` sends it to the background.

List background jobs:
```bash
jobs
```

Bring one back:
```bash
fg
```

### Viewing Processes

#### `ps`

```bash
ps
```

Shows processes associated with your current terminal.

#### Full Process List

```bash
ps -ef
```

or

```bash
ps aux
```

**Common columns:**

| Column | Description |
|--------|-------------|
| USER | Owner of the process |
| PID | Process ID |
| %CPU | CPU usage percentage |
| %MEM | Memory usage percentage |
| COMMAND | The command that started the process |

### Live Process Monitoring

#### `top`

```bash
top
```

Displays:

- CPU usage
- Memory usage
- Running processes
- Load averages

Exit with: `q`

#### `htop`

A more user-friendly alternative.

```bash
htop
```

May require installation:
```bash
sudo apt install htop
```

### Searching for a Process

Find Node.js:

```bash
ps -ef | grep node
```

**Example:**
```
ubuntu    2511    node server.js
```

> You'll learn `grep` in depth later, but for now think of it as a filter.

### Terminating Processes

#### Graceful Stop

```bash
kill 2511
```

This sends the **SIGTERM** signal. The application has a chance to clean up resources.

#### Force Kill

```bash
kill -9 2511
```

This sends **SIGKILL**. The process stops immediately.

> ⚠️ Use only when necessary.

#### Kill by Name

```bash
pkill node
```

Kills processes matching the name.

### Process States

| State | Meaning |
|-------|---------|
| Running | Currently executing |
| Sleeping | Waiting for work |
| Stopped | Paused |
| Zombie | Finished but not cleaned up |

> Zombie processes are uncommon but indicate that a parent process has not collected the child's exit status.

### Load Average

Run:

```bash
uptime
```

**Example:**
```
load average: 0.45, 0.60, 0.72
```

These values show average system load over:

- **1 minute**
- **5 minutes**
- **15 minutes**

> A load significantly higher than the number of CPU cores may indicate the system is overloaded.

---

## 🌍 Real-World Example: MERN Backend Stops Responding

Users report:

```
https://api.company.com returns errors.
```

**Troubleshooting:**

**Step 1** — Check if Node.js is running:
```bash
ps -ef | grep node
```

**Step 2** — Monitor CPU:
```bash
top
```

**Step 3** — If the process is consuming 100% CPU unexpectedly, investigate application logs before restarting.

**Step 4** — If needed:
```bash
kill <PID>
```

**Step 5** — Then restart the service (we'll learn proper service management using `systemctl` tomorrow).

---

## 🏗️ Architecture Diagram (ASCII)

```
              systemd (PID 1)
                     │
              ┌──────┴──────┐
              │             │
           nginx         mongod
              │
          node server.js
              │
          Worker Threads
```

---

## 🛠️ Step-by-Step Practical Implementation

**Step 1** — View running processes:
```bash
ps -ef
```

**Step 2** — Monitor system:
```bash
top
```
Press `q` to quit.

**Step 3** — Display uptime:
```bash
uptime
```

**Step 4** — Open another terminal and run:
```bash
sleep 300
```

**Step 5** — Find it:
```bash
ps -ef | grep sleep
```

**Step 6** — Terminate it gracefully:
```bash
kill <PID>
```
Replace `<PID>` with the actual process ID.

**Step 7** — Start a background process:
```bash
sleep 120 &
```

**Step 8** — List jobs:
```bash
jobs
```

**Step 9** — Bring it back:
```bash
fg
```

**Step 10** — Stop it:
```
Ctrl + C
```

---

## ✅ Best Practices

- Prefer `kill` (SIGTERM) before `kill -9`.
- Investigate why a process failed before restarting it.
- Use `top` or `htop` during performance investigations.
- Monitor CPU and memory trends instead of reacting to one snapshot.
- Avoid killing system processes unless you understand their purpose.

---

## ❌ Common Mistakes

- Using `kill -9` as the first option.
- Killing the wrong PID.
- Assuming high CPU always means an application bug (it could be expected under load).
- Ignoring zombie processes during debugging.
- Restarting services without checking logs.

---

